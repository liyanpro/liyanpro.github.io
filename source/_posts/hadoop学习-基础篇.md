---
title: Hadoop学习-基础篇
date: 2020-01-19 18:08:05
header-img: https://bucketyy.oss-cn-beijing.aliyuncs.com/image/hadoop/Install-Hadoop.png
tags: 
      - 大数据
---
> Hadoop 大数据平台与架构

### 功能与优势
Hadoop是大数据存储与分析的架构，是分布式存储和分布式计算平台

### 两个核心组成
* HDFS：分布式文件系统，用于存储海量的数据
* MapReduce：并行处理框架，实现任务的分解和调度

### 应用
可用于搭建数据仓库，分析统计数据

### 生态
* HIVE：SQL语句形式，转换为Hadoop任务去执行
* HBASE：存储结构化数据的分布式数据库
* zookeeper：服务注册、治理

### HDFS概念
* 块（Block）：文件被切分成同大小的数据块，默认为64MB，是文件存储的逻辑单元
* NameNode：数据管理节点，包括文件与数据块映射表，数据块与数据节点映射表
* DataNode：工作节点，存放真正的数据块

![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/hadoop/hadoop_dataNode%E5%B7%A5%E4%BD%9C.jpg)

#### HDFS数据管理与容错
每个数据块有3个副本，分布在两个机架上，如上图数据块

#### HDFS特点
* 数据冗余，硬件容错
* 流式数据访问
* 易于存储大文件

#### HDFS使用
配置好hadoop服务并启动后，执行 jps 命令，如看到这些服务即启动成功

    [root@liyan ~]# jps
    14258 Jps
    5108 TaskTracker
    4887 SecondaryNameNode
    4634 NameNode
    4970 JobTracker
    4765 DataNode
执行 hadoop dfsadmin -report 可以看到整个hadoop的使用情况
    
    [root@liyan ~]# hadoop dfsadmin -report
    Configured Capacity: 42139451392 (39.25 GB)
    Present Capacity: 34523619328 (32.15 GB)
    DFS Remaining: 34523467776 (32.15 GB)
    DFS Used: 151552 (148 KB)
    DFS Used%: 0%
    Under replicated blocks: 8
    Blocks with corrupt replicas: 0
    Missing blocks: 0
    
    -------------------------------------------------
    Datanodes available: 1 (1 total, 0 dead)
    
    Name: 127.0.0.1:50010
    Decommission Status : Normal
    Configured Capacity: 42139451392 (39.25 GB)
    DFS Used: 151552 (148 KB)
    Non DFS Used: 7615832064 (7.09 GB)
    DFS Remaining: 34523467776(32.15 GB)
    DFS Used%: 0%
    DFS Remaining%: 81.93%
    Last contact: Sun Jan 19 18:55:45 CST 2020
    
##### HDFS的三个基本操作命令
* 从本地linux上传文件到HDFS
       
      hadoop fs -put 文件  文件夹/
        
* 从HDFS查看文件：

      hadoop fs -cat 文件夹/文件
* 从HDFS下载文件到本地：

      hadoop fs -get 文件夹/文件  新文件
    
### MapReduce原理
分而治之，将大任务拆分成多个小任务（map），并行执行，合并结果（reduce）

#### 概念
* Job&Task：一个Job被拆分为多个Task
* JobTracker：作业调度、分配任务、监控任务执行进度、监控TaskTracker状态

#### 体系结构
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/hadoop/hadoop_reduce.jpg)

#### MapReduce容错机制
* 重复执行
* 推测执行：如果之前的TaskTracker计算很慢，则推测可能出现问题，新建一台TaskTracker同时与之计算，谁先执行完成取谁结果

### Hadoop应用实践
按照官网推荐的WordCount来实践一下，输入文本文件，统计里面单词的出现次数
1. 先将wordcount.jar文件上传到Linux服务器本地

        scp wordcount.jar root@ip:/opt/examples
2.创建input文件夹，里面存放输入的文本文件
3.在HDFS里创建input_wordcount文件夹作为输入文件夹
    
        hadoop fs -mkdir input_wordcount/
4.将输入文件上传到HDFS的input_wordcount文件夹，使用 hadoop fs -ls 命令查看wordcount文件夹下是否已有文件

        [root@liyan examples]# hadoop fs -put input/* input_wordcount
        [root@liyan examples]# hadoop fs -ls input_wordcount/
        Found 2 items
        -rw-r--r--   3 root supergroup        147 2020-01-19 19:55 /user/root/input_wordcount/file1.text
        -rw-r--r--   3 root supergroup        153 2020-01-19 19:55 /user/root/input_wordcount/file2.text

5.执行jar文件，hadoop jar jar文件 main函数名称 输入文件夹  输出文件夹

        [root@liyan examples]# hadoop jar wordcount.jar WordCount input_wordcount output_wordcount
        20/01/19 20:35:13 WARN mapred.JobClient: Use GenericOptionsParser for parsing the arguments. Applications should implement Tool for the same.
        20/01/19 20:35:13 INFO input.FileInputFormat: Total input paths to process : 2
        20/01/19 20:35:13 INFO util.NativeCodeLoader: Loaded the native-hadoop library
        20/01/19 20:35:13 WARN snappy.LoadSnappy: Snappy native library not loaded
        20/01/19 20:35:13 INFO mapred.JobClient: Running job: job_202001192032_0002
        20/01/19 20:35:14 INFO mapred.JobClient:  map 0% reduce 0%
        20/01/19 20:35:22 INFO mapred.JobClient:  map 50% reduce 0%
        20/01/19 20:35:32 INFO mapred.JobClient:  map 100% reduce 0%
        20/01/19 20:35:33 INFO mapred.JobClient:  map 100% reduce 33%
        20/01/19 20:35:35 INFO mapred.JobClient:  map 100% reduce 100%
        20/01/19 20:35:36 INFO mapred.JobClient: Job complete: job_202001192032_0002
        20/01/19 20:35:36 INFO mapred.JobClient: Counters: 29
        20/01/19 20:35:36 INFO mapred.JobClient:   Map-Reduce Framework
        20/01/19 20:35:36 INFO mapred.JobClient:     Spilled Records=54
        20/01/19 20:35:36 INFO mapred.JobClient:     Map output materialized bytes=331
        20/01/19 20:35:36 INFO mapred.JobClient:     Reduce input records=27
        20/01/19 20:35:36 INFO mapred.JobClient:     Virtual memory (bytes) snapshot=5852057600
        20/01/19 20:35:36 INFO mapred.JobClient:     Map input records=8
        20/01/19 20:35:36 INFO mapred.JobClient:     SPLIT_RAW_BYTES=246
        20/01/19 20:35:36 INFO mapred.JobClient:     Map output bytes=503
        20/01/19 20:35:36 INFO mapred.JobClient:     Reduce shuffle bytes=331
        20/01/19 20:35:36 INFO mapred.JobClient:     Physical memory (bytes) snapshot=411611136
        20/01/19 20:35:36 INFO mapred.JobClient:     Reduce input groups=19
        20/01/19 20:35:36 INFO mapred.JobClient:     Combine output records=27
        20/01/19 20:35:36 INFO mapred.JobClient:     Reduce output records=19
        20/01/19 20:35:36 INFO mapred.JobClient:     Map output records=51
        20/01/19 20:35:36 INFO mapred.JobClient:     Combine input records=51
        20/01/19 20:35:36 INFO mapred.JobClient:     CPU time spent (ms)=980
        20/01/19 20:35:36 INFO mapred.JobClient:     Total committed heap usage (bytes)=292954112
        20/01/19 20:35:36 INFO mapred.JobClient:   File Input Format Counters
        20/01/19 20:35:36 INFO mapred.JobClient:     Bytes Read=300
        20/01/19 20:35:36 INFO mapred.JobClient:   FileSystemCounters
        20/01/19 20:35:36 INFO mapred.JobClient:     HDFS_BYTES_READ=546
        20/01/19 20:35:36 INFO mapred.JobClient:     FILE_BYTES_WRITTEN=156082
        20/01/19 20:35:36 INFO mapred.JobClient:     FILE_BYTES_READ=325
        20/01/19 20:35:36 INFO mapred.JobClient:     HDFS_BYTES_WRITTEN=147
        20/01/19 20:35:36 INFO mapred.JobClient:   Job Counters
        20/01/19 20:35:36 INFO mapred.JobClient:     Launched map tasks=3
        20/01/19 20:35:36 INFO mapred.JobClient:     Launched reduce tasks=1
        20/01/19 20:35:36 INFO mapred.JobClient:     SLOTS_MILLIS_REDUCES=11869
        20/01/19 20:35:36 INFO mapred.JobClient:     Total time spent by all reduces waiting after reserving slots (ms)=0
        20/01/19 20:35:36 INFO mapred.JobClient:     SLOTS_MILLIS_MAPS=13320
        20/01/19 20:35:36 INFO mapred.JobClient:     Total time spent by all maps waiting after reserving slots (ms)=0
        20/01/19 20:35:36 INFO mapred.JobClient:     Data-local map tasks=3
        20/01/19 20:35:36 INFO mapred.JobClient:   File Output Format Counters
        20/01/19 20:35:36 INFO mapred.JobClient:     Bytes Written=147
6.查看执行结果
        
        [root@liyan examples]# hadoop fs -ls output_wordcount/
        Found 3 items
        -rw-r--r--   3 root supergroup          0 2020-01-19 20:35 /user/root/output_wordcount/_SUCCESS
        drwxr-xr-x   - root supergroup          0 2020-01-19 20:35 /user/root/output_wordcount/_logs
        -rw-r--r--   3 root supergroup        147 2020-01-19 20:35 /user/root/output_wordcount/part-r-00000
  part-r-00000文件中记录了统计结果
  
        [root@liyan examples]# hadoop fs -cat output_wordcount/part-r-00000
        five	2
        four	2
        hadoop	7
        help	3
        liyan	6
        love	7
        script	1
        six	1
        tacker	2
        tracker	4
        wend	5
        window	2
        word	3
  
        




