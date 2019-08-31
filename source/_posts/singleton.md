---
title: 设计模式之—单例模式
date: 2017-11-16 22:43:21
header-img: https://bucketyy.oss-cn-beijing.aliyuncs.com/image/home-bg.jpg
tags:
---

单例模式（Singleton Pattern）涉及到一个单一的类，该类负责创建属于自己的对象，并且确保只有一个对象被创建，这种类提供了一种访问其唯一对象的方式，可以直接访问，无需实例化。
        
#### 介绍
   现在的操作系统都为多进程多线程的，如windows系统，在多个线程操作文件时就必须确保所有的文件操作都对应唯一的实例来进行；一些设备管理器经常被设计为单例，如多台打印机同时工作，需满足不能打印同一个文件。
Singleton Pattern主要解决一个全局变量频繁的被创建和销毁。

#### 使用场景
 * web计数器，每次加数不用保存到数据库，可以先用单例缓存起来
 * 创建的一个对象需要消耗的资源过多，比如I/O，数据库连接等
 * spring中的依赖注入 
步骤一：创建一个Singleton类  SingletonObject.java

       public class SingletonObject{
         private static SingletonObject instance=new SingletonObject();
          //私有化构造函数，防止该类被实例化
         private SingletonObject(){}
         public SingletonObject getInstance(){
                return instance;   
        }
         public void show() {
        System.out.println("this is single object");
    }

步骤二：从SingletonObject类获取唯一的实例

       public  class SingletonPatternDemo{
        public static void main(String[] args) {
        //不合法的构造函数
        // SingleObject instance1 = new SingleObject();
        //获取唯一可用的实例对象
        SingleObject instance = SingleObject.getInstance();
        instance.show();
    }
    
上面属于懒汉式线程不安全，安全的懒汉式如下：
        
        public class Singleton{
           private static Singleton instance;
           private Singleton(){}
           public static synchronized Singleton getInstance(){
                if(instance==null){
                   instance=new Singleton();
                }
                return instance;
           }
        
        }
        
饿汉式属于线程安全模式，但容易产生垃圾对象

         public class Singleton{
                private static Singleton instance=new Singleton;
                private Singleton(){}
                public static Singleton getInstance(){
                      return instance;
                }
                
         }     
                
双重校验锁模式，结合了volatile关键字，使作用的变量处于线程共享变量，安全且在多线程情况下能保持高性能
        
        public class Singleton{
               private volatile static Singleton instance;
               private Singleton(){}
               public static Singleton getInstance(){
                   if(instance==null){
                      synchronized(Singleton.class){
                           if(instance==null){
                              instance=new Singleton();
                           }
                      }
                   }
                  return instance;
               }
                        
        } 
静态内部类模式，这种方式同样利用了 classloader 机制来保证初始化 instance 时只有一个线程，与饿汉式不同的是，这种采用的是延迟初始化，并不是在 Singleton被装载时就初始化 Singleton 实例，而是当 getInstance() 方法被调用时才装载 SingletonHolder类

        public class Singleton{
               private static class SingletonHolder{
                 private static final Singleton INSTANCE=new Singleton();
               }
               private Singleton(){}
               public static Singleton getInstance(){
                      return SingletonHolder.INSTANCE;
               }
                       
        }         