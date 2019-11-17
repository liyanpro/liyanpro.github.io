---
title: 设计模式之代理模式
date: 2019-11-17 10:39:55
tags:
      - 设计模式
      - 动态代理
---

代理模式指的是为其他对象创建一种代理，以便控制对此对象的访问。

创建接口

    public interface Image {
    
        void display(String str);
    }
    
实现类

    public class RealImage implements Image {
    
        public String name;
    
        public RealImage(String name) {
            this.name = name;
        }
    
        @Override
        public void display(String str) {
    
            System.out.println("我是代理类输出,name=" + name + ",str=" + str);
        }
    }
    
使用jdk的动态代理实现类的代理，实现 InvocationHandler 接口

    public class DynamicProxyImageHandle<T> implements InvocationHandler{
       public T target;
       
       public DynamicProxyImageHandle (T target){
          this.target=target;
       }
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
         // spring aop思想
         System.out.println("---动态代理前置处理---" + "args=" + JSON.toJSONString(args));
         Object obj=method.invoke(target,args);
         System.out.println("---动态代理后置处理---");
         return obj;
       }
    }
demoTest

    public static void main(String[] args){
       Image realImage=new RealImage("my name");
       InvocationHandler invocationHandler = new DynamicProxyImageHandle<Image>(realImage);
       //创建一个代理实例
       Image proxyImage = (Image)Proxy.newProxyInstance(Image.class.getClassLoader(),new Class<?>[]{Image.class},invocationHandler);
       proxyImage.display("wo");
       
    } 

        