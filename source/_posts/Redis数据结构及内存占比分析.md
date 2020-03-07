---
title: Redis数据结构及内存占比分析
date: 2020-03-07 10:37:08
header-img: https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-cache-1.png
tags:
     - redis
     - 数据结构
---

### Redis数据结构存储实验
Redis常用的有五大数据结构，包括String、List、Hash、Set、SortSet，那么存入相同大小的数据，这些数据结构所占的内存大小都是多少呢？下面做个实验，看看这些不同数据结构存储同样的100W数据，所占用的内存及查询时间。这里的100W数据指的是key和value总和，数据均采用32位UUID，Hash的field和SortSet的score使用一到两位字符存储，占有内存忽略不计。
思路： 依次往各数据结构添加100W数据，选第10000个key作为查询key，添加完成后使用redis的 info‘memory’命令查看内存占用情况，计算查询key的检索时间，以纳秒为单位，完成统计后使用flushAll命令清空库，再进行一下个数据结构统计，实验结果如下：

1. String结构
占用内存：73.63M，查询key时长：130961 ns
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-string.png)
2. List结构
占用内存：37.48M，查询key时长：80195 ns（通过pop命令出队）
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-list.png)
3. Hash结构
占用内存：41.11M，查询key时长：77621 ns
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-hash.png)
4. Set结构
占用内存：93.10M，查询key时长：5495122 ns（通过 smembers命令获取key集合全部成员）
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-set.png)
5. SortSet结构
占用内存：41.11M，查询key时长：228546 ns（通过zrange命令获取key有序集合成员）
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-sortset.png)

#### 结论
Key-Value结构查询速度更快，时间复杂度为O(1)，但会消耗更多内存，相比之下，Hash更优于Set、String结构，如果单纯的存储和查询，不做集合、排序处理，优先选择Hash结构。List不适合做检索，SortSet为有序集合，采用skiplist结构，检索速度比哈希略慢。

### Redis数据结构分析
Redis各数据结构实现原理，如下图：
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)
-  动态字符串SDS
Redis 是用 C 语言写的，但是对于Redis的字符串，却不是 C 语言中的字符串，它是自己构建了一种名为简单动态字符串（simple dynamic string,SDS）的抽象类型，并将 SDS 作为 Redis的默认字符串表示，它的特点就是预先分配内存，记录字符串长度，在原字符串数组buf[]中新增一串字符串。优点就是当清空时，并没有真正释放内存，而是将长度字段len值为0，当再次使用时，避免重新分配内存，从而提高效率。
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-sources.png)
-  压缩列表ziplist
ziplist是列表键和哈希键的底层实现之一。是由一系列特殊编码的连续内存块组成，当一个列表键只包含少量列表项并且每个都是小整数值或者长度比较短的字符串时，Redis就采用压缩列表做底层实现，这个值可以在redis.conf文件中进行修改，下图是Hash结构的配置，hash-max-ziplist-entries表示键值对个数， hash-max-ziplist-value字节数，（默认是64，我这边做了修改）满足这两点hash就使用ziplist存储数据，超过后重构为hashtable。同理，还可对SortSet结构设置zset-max-ziplist-entries和zset-max-ziplist-value来使用ziplist，当不满足后重构为skiplist（字典和跳跃表结构）
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-config.png)
-  字典
Redis的字典dict使用哈希表实现，关于哈希表，可以参考java中hashmap实现原理，采用数组+链表方式，其中dictht就是哈希表结构。
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-sources1.png)
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-sources2.png)
-  跳跃表
跳跃表原理较复杂点，它的存在是牺牲一点存储空间使查询元素更接近O(logn)。下图是参考网上对跳跃表的解释，header 和 tail 指针分别指向跳跃表的表头和表尾节点，通过这两个指针，程序定位表头节点和表尾节点的复杂度为 O(1) 。通过使用 length 属性来记录节点的数量， 程序可以在 O(1) 复杂度内返回跳跃表的长度。level 属性则用于在 O(1) 复杂度内获取跳跃表中层高最大的那个节点的层数。[参考redis设计与分析](http://redisbook.com/preview/skiplist/datastruct.html)
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-simple.png)
![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-simple2.png)

如上图中，左侧L5层高为3，所在表中位置为5，所以level为5

![](https://bucketyy.oss-cn-beijing.aliyuncs.com/image/article/redis-sources3.png)

[源码来源](https://github.com/antirez/redis)


> 附 测试代码

    public void analyzedRedisMemory() {
    
        Jedis jedis = RedisClient.getInstance().getJedis();
        try {
            this.logger.info("========添加100W数据到 String 结构中=======");
            String searchKey = "";
            for (int i = 0; i < 500000; i++) {
                if (i % 100000 == 0) {
                    System.out.println("已添加 " + i * 2 + " 条数据 ");
                }
                String key = UUID.randomUUID().toString();
                if (i == 10000) {
                    searchKey = key;
                }
                jedis.set(key, UUID.randomUUID().toString());
            }
            long begin = System.nanoTime();
            jedis.get(searchKey);
            this.logger.info("查询key所用时长：{}(ns)", (System.nanoTime() - begin));
            this.logger.info(jedis.info("memory"));
            jedis.flushAll();
            Thread.sleep(500);
            this.logger.info("========添加100W数据到 List 结构中=======");
            String listKey = UUID.randomUUID().toString();
            for (int i = 0; i < 1000000; i++) {
                if (i % 100000 == 0) {
                    System.out.println("已添加 " + i + " 条数据 ");
                }
                jedis.lpush(listKey, UUID.randomUUID().toString());
            }
            begin = System.nanoTime();
            jedis.rpop(listKey);
            this.logger.info("查询key所用时长：{}(ns)", (System.nanoTime() - begin));
            this.logger.info(jedis.info("memory"));
            jedis.flushAll();
            Thread.sleep(500);
            this.logger.info("========添加100W数据到 Set 结构中=======");
            String key1 = "";
            for (int i = 0; i < 10000; i++) {
                // 每个集合数量为99个值
                key1 = UUID.randomUUID().toString();
                if (i == 5000) {
                    searchKey = key1;
                }
                for (int j = 0; j < 99; j++) {
                    jedis.sadd(key1, UUID.randomUUID().toString());
                }
                if (i % 1000 == 0) {
                    System.out.println("已添加 " + i * 100 + " 条数据 ");
                }
            }
            begin = System.nanoTime();
            jedis.smembers(searchKey);
            this.logger.info("查询key所用时长：{}(ns)", (System.nanoTime() - begin));
            this.logger.info(jedis.info("memory"));
            jedis.flushAll();
            Thread.sleep(500);
            this.logger.info("========添加100W数据到 Hash 结构中=======");
            String key2 = "";
            for (int i = 0; i < 10000; i++) {
                // 一个key有99个field
                key2 = UUID.randomUUID().toString();
                if (i == 5000) {
                    searchKey = key2;
                }
                for (int j = 0; j < 99; j++) {
                    jedis.hset(key2, j + "", UUID.randomUUID().toString());
                }
                if (i % 1000 == 0) {
                    System.out.println("已添加 " + i * 100 + " 条数据 ");
                }
            }
            begin = System.nanoTime();
            jedis.hget(searchKey, 6 + "");
            this.logger.info("查询key所用时长：{}(ns)", (System.nanoTime() - begin));
            this.logger.info(jedis.info("memory"));
            jedis.flushAll();
            Thread.sleep(500);
            this.logger.info("========添加100W数据到 SortSet 结构中=======");
            String key3 = "";
            for (int i = 0; i < 10000; i++) {
                // 每个集合数量为99个值
                key3 = UUID.randomUUID().toString();
                if (i == 5000) {
                    searchKey = key3;
                }
                for (int j = 0; j < 99; j++) {
                    jedis.zadd(key3, j, UUID.randomUUID().toString());
                }
                if (i % 1000 == 0) {
                    System.out.println("已添加 " + i * 100 + " 条数据 ");
                }
            }
            begin = System.nanoTime();
            jedis.zrange(searchKey, 0, -1);
            this.logger.info("查询key所用时长：{}(ns)", (System.nanoTime() - begin));
            this.logger.info(jedis.info("memory"));
            jedis.flushAll();
        } catch (Exception e) {
            this.logger.error("执行错误", e);
        } finally {
            jedis.close();
        }
    }
    




