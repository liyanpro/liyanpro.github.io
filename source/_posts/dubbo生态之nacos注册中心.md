---
title: dubbo生态之nacos注册中心
date: 2019-08-15 11:44:20
header-img: https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/nacos.jpeg
tags: 
    - nacos
    - dubbo
    - 注册中心
    - rpc
---

> Nacos 致力于微服务的发现、配置和管理，是现代应用结构的服务基础设施，Nacos 作为 Dubbo 生态系统中重要的注册中心实现，其中 dubbo-registry-nacos 则是 Dubbo 融合 Nacos 注册中心的实现

### 安装 Nacos 服务
可以通过源码和发行包的方式安装
 - 发行包地址：https://github.com/alibaba/nacos/releases
 - 源码方式：从github 下载源码
 
        git clone https://github.com/alibaba/nacos.git
        cd nacos/
        mvn -Prelease-nacos clean install -U  
        ls -al distribution/target/
        cd distribution/target/nacos-server-$version/nacos/bin
    
根据不通环境启动参数，linux/Mac os：sh startup.sh -m standalone   (standalone代表单机运行模式)
 
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/nacos0.png)
 
启动成功后，在浏览器输入：http://127.0.0.1:8848/nacos 进入服务管理页面，如果使用Nacos 0.8.0以上版本，会出现登录页面，默认用户名密码都为：nacos
 
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/nacos1.png)
 
### 创建应用实现服务生产-消费模型
首先创建一个springboot项目，在二级目录下创建三个子工程，目录结构如下：
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/nacos2.png)
#### 引入依赖配置
 
      <!-- Dubbo Nacos registry dependency -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>dubbo-registry-nacos</artifactId>
                <version>0.0.1</version>
            </dependency>
    
            <!-- Dubbo dependency -->
            <dependency>
                <groupId>com.alibaba</groupId>
                <artifactId>dubbo</artifactId>
                <version>2.6.5</version>
            </dependency>
            
注意dubbo版本，2.7.0以上groupId属于apache了，引入 dubbo-registry-nacos 这个版本包会报错
#### 定义服务接口
dubbo-demo-api工程中定义服务接口：
    
     package org.apache.dubbo.demo;
         
         /**
          * @author liyan
          * @description
          * @date 2019-08-09 16:44
          */
         public interface DemoService {
         
             String sayHello(String name);
         }
    
#### 服务生产者实现接口
dubbo-demo-provider工程中实现：
DemoServiceImpl.java:
 
         package org.apache.dubbo.demo.provider;
         import org.apache.dubbo.demo.DemoService;
         /**
          * @author liyan
          * @description
          * @date 2019-08-09 16:47
          */
         public class DemoServiceImpl implements DemoService{
         
             @Override
             public String sayHello(String name) {
                 return "Hello " + name;
             }
         }
    
dubbo通过spring的配置声明暴露服务（[Spring Schema](https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/xsd-configuration.html)）
在resource/META-INF/spring目录下，创建 dubbo-demo-provider.xml 文件
    
     <?xml version="1.0" encoding="UTF-8"?>
         <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
                xmlns="http://www.springframework.org/schema/beans"
                xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
                http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
         
             <!-- 提供方应用信息，用于计算依赖关系 -->
             <dubbo:application name="demo-provider"/>
             <!-- 使用 nacos 注册中心暴露服务地址 -->
             <dubbo:registry address="nacos://127.0.0.1:8848"/>
             <!-- 用dubbo协议在20880端口暴露服务 -->
             <dubbo:protocol name="dubbo" port="20880"/>
             <!-- 声明需要暴露的服务接口 -->
             <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl"/>
              <!-- 和本地bean一样实现服务 -->
             <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService"/>
         </beans>
   
#### 加载spring配置，启动生产服务
 provider.java:
    
      public class provider {
         
           public static void main(String[] args) throws Exception {
                     System.setProperty("java.net.preferIPv4Stack", "true");
                     ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"META-INF/spring/dubbo-demo-provider.xml"});
                     context.start();
                     System.out.println("Provider started.");
                     System.in.read(); // press any key to exit
           }
         }
  
#### 服务消费者
 dubbo-demo-consumer工程中，通过spring配置引用远程服务，在resource/META-INF/spring路径下，创建dubbo-demo-consumer.xml文件
  
       <?xml version="1.0" encoding="UTF-8"?>
             <beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                    xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
                    xmlns="http://www.springframework.org/schema/beans"
                    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
                    http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
             
                 <!-- consumer's application name, used for tracing dependency relationship (not a matching criterion),
                 don't set it same as provider -->
                 <dubbo:application name="demo-consumer"/>
                  <!-- 使用 nacos 注册中心暴露发现服务地址 -->
                 <dubbo:registry address="nacos://127.0.0.1:8848"/>
                 <!-- generate proxy for the remote service, then demoService can be used in the same way as the
                  <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
                 <dubbo:reference id="demoService" check="false" interface="org.apache.dubbo.demo.DemoService"/>
             </beans>
    
#### 加载spring配置，调用远程服务
 启动服务生产服务，创建 consumer.java 并执行:
     
      public class consumer {
         
             public static void main(String[] args) throws Exception {
                 ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"META-INF/spring/dubbo-demo-consumer.xml"});
                 context.start();
                 // Obtaining a remote service proxy
                 DemoService demoService = (DemoService)context.getBean("demoService");
                 // Executing remote methods
                 String hello = demoService.sayHello("world");
                 // Display the call result
                 System.out.println(hello);
             }
         }
   
#### nacos 管理页面监控
 调用完成后，打开nacos 管理页面-服务列表,可以看到具体的服务及健康详情：
 ![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/nacos3.png)
  
  
   
       
    
 
 
 
        
              
   
   


