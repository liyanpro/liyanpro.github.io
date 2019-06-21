---
title: 设计模式学习笔记之——单例模式
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