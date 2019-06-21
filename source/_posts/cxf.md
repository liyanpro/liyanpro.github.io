---
title: 基于CXF的WebService服务
date: 2017-04-15 19:49:04
header-img: https://bucketyy.oss-cn-beijing.aliyuncs.com/image/tag-bg.jpg
tags: 
---


CXF是Apache的一个开源框架，使用它可以发布一个类服务，其他应用程序可以调用接口，以下是我搭建webservice服务的步骤，主要是与spring整合搭建，用的是cxf 3.x和spring 4.x。
#### 1 服务器端webservice服务搭建
* 首先引入cxf的maven依赖
        
        <!-- CXF dependency -->
		 <dependency>
			<groupId>org.apache.cxf</groupId>
			<artifactId>cxf-core</artifactId>
			<version>3.1.10</version>
		</dependency>
		<dependency>
			<groupId>org.apache.cxf</groupId>
			<artifactId>cxf-rt-bindings-soap</artifactId>
			<version>3.1.10</version>
		</dependency>
		<dependency>
			<groupId>org.apache.cxf</groupId>
			<artifactId>cxf-rt-transports-http</artifactId>
			<version>3.1.10</version>
		</dependency>
		<dependency>
			<groupId>org.apache.cxf</groupId>
			<artifactId>cxf-rt-frontend-jaxws</artifactId>
			<version>3.1.10</version>
		</dependency>
		<dependency>
			<groupId>org.apache.cxf</groupId>
			<artifactId>cxf-rt-frontend-jaxrs</artifactId>
			<version>3.1.10</version>
		</dependency>
		<!--  CXF dependency end --> 
这里需要说明的一点是cxf-core包在2.x版本中命名为cxf-rt-core,低版本的spring用的都是cxf-rt-core，刚开始我引的包也是这个，maven库中维护到2.7版之后这个包就以cxf-core的命名开始3.x的版本维护，由于与其他包的版本冲突造成服务一直搭不起来，浪费了很多时间。
* 编写需要发布的类
    接口 IKqglZbgsService，只需在接口上添加@webservice注解，表示这是一个webservice接口，传入参数为json字符串。 
        package com.thunisoft.kqgl.service.zb;
        import javax.jws.WebMethod;
        import javax.jws.WebService;
        @WebService
        public interface IKqglZbgsService {
        /**
        * 获取反馈结果信息
        * <p>Description: TODO</p>
        * @param paramJson 参数
        * @return 正常工时 正常工时花费 加班工时 加班工时花费
        */
        @WebMethod
        public String getResponseResult(String paramJson);
        }
实现类 KqglZbgsServiceImpl
         /**
          * 获取周报工时反馈结果信息
          * <p>Description: TODO</p>
          * @param paramJson 参数
          * @return 正常工时 正常工时花费 加班工时 加班工时花费
          */
           @Override
           public String getResponseResult(String paramJson) {
            Map<String, String> params = new HashMap<String, String>();
           //将参数转换成JSON对象
           JSONObject jsonStr = JSONObject.fromObject(paramJson);
           params.put("yxtbh", jsonStr.getString("yxtbh"));
           params.put("xmzlx", jsonStr.getString("xmzlx"));
           params.put("rzlx", jsonStr.getString("rzlx"));
           JSONObject jsons = zbgsDao.getResponseResultsDao(params);
           return jsons.toString();
           }
* 在spring配置文件中发布接口，cxf 3.x只需引入cxf.xml文件即可
          <?xml version="1.0" encoding="UTF-8"?>
         <beans
	          xmlns="http://www.springframework.org/schema/beans"
	          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xmlns:p="http://www.springframework.org/schema/p"
	          xmlns:jaxws="http://cxf.apache.org/jaxws"
	          xsi:schemaLocation=" http://www.springframework.org/schema/beans 
                http://www.springframework.org/schema/beans            
                http://www.springframework.org/schema/beans/spring-beans-         
                4.2.xsd
               http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
              <import resource="classpath:META-INF/cxf/cxf.xml" />
	      <!-- 通过jaxws:server方式来配置webservice -->
	          <bean id="zbgsServiceImpl"  class="com.thunisoft.kqgl.service.impl.zb.KqglZbgsServiceImpl" />                           
              <jaxws:server   address="/zbgs" serviceClass="com.thunisoft.kqgl.service.zb.IKqglZbgsService">
              <jaxws:serviceBean>
                  <ref bean="zbgsServiceImpl" />
              </jaxws:serviceBean>
              </jaxws:server>   
         </beans>
* 在web.xml中配置拦截器，url中以ws开始的视为webservice接口
           <!-- here is CXFServlet config -->
	   <servlet>
		<servlet-name>cxf</servlet-name>
		<servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	   </servlet>
	   <servlet-mapping>
		<servlet-name>cxf</servlet-name>
		<url-pattern>/ws/*</url-pattern>
	   </servlet-mapping>
* 最后在地址栏输入访问接口的服务地址，如生成了WSDL文档说明接口发布成功
![](http://upload-images.jianshu.io/upload_images/4367237-9d1d5970c0b7588f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2 客户端调用
引入cxf.jaxws jar包
    
     /**
     * 
     * @param methondNew
     * @param jsonStr
     * @return
     */
    public String invoke(String methondNew, String jsonStr) {
        JaxWsDynamicClientFactory factory = JaxWsDynamicClientFactory
                .newInstance();
        //读取配置文件中接口地址
        String url = ArteryConfigUtil.getProperty("kqgl.webservice.address");
        HTTPClientPolicy httpClientPolicy = new HTTPClientPolicy();
        httpClientPolicy.setConnectionTimeout(CommonConstant.NUMLJ_30000); //连接超时 
        httpClientPolicy.setAllowChunking(false);
        httpClientPolicy.setReceiveTimeout(CommonConstant.NUMJS_30000); //接收超时            
        Object[] json = null;
        try {
            Client client = factory.createClient(url);
            HTTPConduit httpConduit = (HTTPConduit) client.getConduit();
            httpConduit.setClient(httpClientPolicy);
            //invoke方法传入接口方法名和参数
            json = client.invoke(methondNew, jsonStr);
        } catch (Exception e) {
            Log.error(e);
        }
        return json[0].toString();
    }