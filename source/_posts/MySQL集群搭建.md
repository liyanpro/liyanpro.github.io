---
title: MySQL集群搭建
date: 2022-02-22 10:43:26
tags:
     - 集群
     - MySQL
---

 M1：主服务器（ip1）  M2: 从服务器（ip2）

> /etc/mycnf 配置mysql的binlog
> 重启：systemctl restart mysqld
> 获取临时密码； sudo grep 'temporary' /var/log/mysqld.log  

安全配置向导
> M1&M2   newpassword:  root / rootpassword    repluser / userpassword  #主从同步专用用户    security_con  /  conpassword  #业务专用用户

  ### 1. mysqldump 导出备份，同步到从服务器，再恢复
     备份：
           
           mysqldump -uroot -p'password' --all-databases --single-transaction > /home/atrust-app/data/mysql_full-1.sql;
     恢复：
     
           mysql -uroot -p'password' < /home/atrust-app/data/mysql_full-1.sql
  ### 2. 查看主服务器状态
  
           mysql> show master status;
           +------------------+----------+--------------+------------------+-------------------+
           | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
           +------------------+----------+--------------+------------------+-------------------+
           | M1-binlog.000001 |     1433 |              |                  |                   |
           +------------------+----------+--------------+------------------+-------------------+
           1 row in set (0.00 sec)
           
   ##### 1).登录从服务器
   
         change master to
        -> master_host='10.238.0.92',
        -> master_user='repluser',
        -> master_password='cTl2021:at#89!',
        -> master_log_file='M1-binlog.000001',
        -> master_log_pos=1433;
       Query OK, 0 rows affected, 2 warnings (0.00 sec)
       
   ##### 2).启动slave同步进程
   
       start slave;
       show slave status\G; #查看slave状态
       *************************** 1. row ***************************
                   Slave_IO_State: Waiting for master to send event
                      Master_Host: 10.238.0.92
                      Master_User: repluser
                      Master_Port: 3306
                    Connect_Retry: 60
                  Master_Log_File: M1-binlog.000001
              Read_Master_Log_Pos: 1433
                   Relay_Log_File: tgypt-xx13b003-yh20w-relay-bin.000002
                    Relay_Log_Pos: 320
            Relay_Master_Log_File: M1-binlog.000001
                 Slave_IO_Running: Yes
                Slave_SQL_Running: Yes   ##这两个为yes表示成功
                
   ##### 3).验证以M1为主，M2为从的主从复制
   ##### 4).设置M2为主，M1为从模式
    M2服务器状态：
   
        mysql> show master status;
       +------------------+----------+--------------+------------------+-------------------+
       | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
       +------------------+----------+--------------+------------------+-------------------+
       | M2-binlog.000001 |   832935 |              |                  |                   |
       +------------------+----------+--------------+------------------+-------------------+
       1 row in set (0.00 sec)

    yum -y install keepalived

### 3. M2 备机上配置回切脚本 /home/atrust-app/opt/mysql_cutback.sh

### 4. VIP为：VIP1

### 5. keepalived搭建

      K1主（ip1）  K2从（ip2）
      
     1）修改 K1机器上 /etc/keepalived/keepalived.conf
       ! Configuration File for keepalived


    vrrp_script chk_mysql {
        script  "/etc/keepalived/check_mysql.sh"
        interval 30  #设置检查间隔时长
    }

    vrrp_instance VI_1 {
        state BACKUP
        interface bond0.2001 ##IP绑定在哪个网卡上写哪个，ifconfig查看
        virtual_router_id 60
        priority 100
        advert_int 1
        nopreempt
        authentication {
            auth_type PASS
            auth_pass 1111
        }
        track_script {
          chk_mysql
        }
        virtual_ipaddress {
            10.238.0.144    #需要修改
        }
    }
