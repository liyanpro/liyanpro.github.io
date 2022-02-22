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
        -> master_host='ip1',
        -> master_user='repluser',
        -> master_password='password',
        -> master_log_file='M1-binlog.000001',
        -> master_log_pos=1433;
       Query OK, 0 rows affected, 2 warnings (0.00 sec)
       
   ##### 2).启动slave同步进程
   
       start slave;
       show slave status\G; #查看slave状态
       *************************** 1. row ***************************
                   Slave_IO_State: Waiting for master to send event
                      Master_Host: ip1
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
            vip1    #需要修改
        }
    }
    
### shell脚本
   
   ##### 1) check_mysql.sh
   
        #!/bin/bash
         
        ###判断如果上次检查的脚本还没执行完，则退出此次执行
        if [ `ps -ef|grep -w "$0"|grep -v "grep"|wc -l` -gt 2 ];then
            exit 0
        fi 
        mysql_con='mysql -uroot -p123456'
        error_log="/etc/keepalived/logs/check_mysql.err"
         
        ###定义一个简单判断mysql是否可用的函数
        function excute_query {
            ${mysql_con} -e "select 1;" 2>> ${error_log}
        }
         
        ###定义无法执行查询，且mysql服务异常时的处理函数
        function service_error {
            echo -e "`date "+%F  %H:%M:%S"`    -----mysql service error，now stop keepalived-----" >> ${error_log}
            service keepalived stop &>> ${error_log}
            echo "DB1 keepalived 已停止"|mail -s "DB1 keepalived 已停止,请及时处理！" liyansuper@outlook.com 2>> ${error_log}
            echo -e "\n---------------------------------------------------------\n" >> ${error_log}
        }
         
        ###定义无法执行查询,但mysql服务正常的处理函数
        function query_error {
            echo -e "`date "+%F  %H:%M:%S"`    -----query error, but mysql service ok, retry after 30s-----" >> ${error_log}
            sleep 30
            excute_query
            if [ $? -ne 0 ];then
                echo -e "`date "+%F  %H:%M:%S"`    -----still can't execute query-----" >> ${error_log}
         
                ###对DB1设置read_only属性
                echo -e "`date "+%F  %H:%M:%S"`    -----set read_only = 1 on DB1-----" >> ${error_log}
                mysql_con -e "set global read_only = 1;" 2>> ${error_log}
         
                ###kill掉当前客户端连接
                echo -e "`date "+%F  %H:%M:%S"`    -----kill current client thread-----" >> ${error_log}
                rm -f /tmp/kill.sql &>/dev/null
                ###这里其实是一个批量kill线程的小技巧
                mysql_con -e 'select concat("kill ",id,";") from  information_schema.PROCESSLIST where command="Query" or command="Execute" into outfile "/tmp/kill.sql";'
                mysql_con -e "source /tmp/kill.sql"
                sleep 2    ###给kill一个执行和缓冲时间
                ###关闭本机keepalived       
                echo -e "`date "+%F  %H:%M:%S"`    -----stop keepalived-----" >> ${error_log}
                service keepalived stop &>> ${error_log}
                echo "DB1 keepalived 已停止"|mail -s "DB1 keepalived 已停止,请及时处理！" liyansuper@outlook.com 2>> ${error_log}
                echo -e "\n---------------------------------------------------------\n" >> ${error_log}
            else
                echo -e "`date "+%F  %H:%M:%S"`    -----query ok after 30s-----" >> ${error_log}
                echo -e "\n---------------------------------------------------------\n" >> ${error_log}
            fi
        }
         
        ###检查开始: 执行查询
        excute_query
        if [ $? -ne 0 ];then
            service mysqld status &>/dev/null
            if [ $? -ne 0 ];then
                service_error
            else
                query_error
            fi
        fi    
        
   ##### 2) mysql_cutback.sh
   
           #!/bin/bash
           ###手动执行将主库切换回DB1的操作
            
           mysql_con='mysql -uroot -ppassword'
            
           echo -e "`date "+%F  %H:%M:%S"`    -----change to BACKUP manually-----" >> /etc/keepalived/logs/state_change.log
           echo -e "`date "+%F  %H:%M:%S"`    -----set read_only = 1 on DB2-----" >> /etc/keepalived/logs/state_change.log
           $mysql_con -e "set global read_only = 1;" 2>> /etc/keepalived/logs/state_change.log
            
           ###kill掉当前客户端连接
           echo -e "`date "+%F  %H:%M:%S"`    -----kill current client thread-----" >> /etc/keepalived/logs/state_change.log
           rm -f /tmp/kill.sql &>/dev/null
           ###这里其实是一个批量kill线程的小技巧
           $mysql_con -e 'select concat("kill ",id,";") from  information_schema.PROCESSLIST where command="Query" or command="Execute" into outfile "/tmp/kill.sql";'
           $mysql_con -e "source /tmp/kill.sql" 2>> /etc/keepalived/logs/state_change.log
           sleep 2    ###给kill一个执行和缓冲时间
            
           ###确保DB1已经追上了,下面的repl为复制所用的账户，-h后跟DB1的内网IP
           log_file_pos=`mysql -urepluser -pcTl2021:at#89! -h10.238.0.92 -e "show slave status\G;"|egrep -w "Master_Log_File|Read_Master_Log_Pos|Relay_Master_Log_File|Exec_Master_Log_Pos"`
           Master_Log_File=`echo $log_file_pos|awk '{print $2}'`
           Read_Master_Log_Pos=`echo $log_file_pos|awk '{print $4}'`
           Relay_Master_Log_File=`echo $log_file_pos|awk '{print $6}'`
           Exec_Master_Log_Pos=`echo $log_file_pos|awk '{print $8}'`
           until [ $Read_Master_Log_Pos = $Exec_Master_Log_Pos -a $Master_Log_File = $Relay_Master_Log_File ]
           do
               echo -e "`date "+%F  %H:%M:%S"`    -----DB1 Exec_Master_Log_Pos($exec_pos) is behind Read_Master_Log_Pos($read_pos), wait......" >> /etc/keepalived/logs/state_change.log
               sleep 1
           done
            
           ###然后解除DB1的read_only属性
           echo -e "`date "+%F  %H:%M:%S"`    -----set read_only = 0 on DB1-----" >> /etc/keepalived/logs/state_change.log
           ssh 10.238.0.92 'mysql -uroot -ppassword -e "set global read_only = 0;" && /etc/init.d/keepalived start' 2>> /etc/keepalived/logs/state_change.log
            
           ###重启DB2的keepalived使VIP漂移到DB1
           echo -e "`date "+%F  %H:%M:%S"`    -----make VIP move to DB1-----" >> /etc/keepalived/logs/state_change.log
           /sbin/service keepalived restart &>> /etc/keepalived/logs/state_change.log
            
           echo "DB2 keepalived转为BACKUP状态，线上数据库切换至DB1"|mail -s "DB2 keepalived change to BACKUP" liyansuper@outlook.com 2>> /etc/keepalived/logs/state_change.log
            
           echo -e "--------------------------------------------------\n" >> /etc/keepalived/logs/state_change.log
           
    
   ##### 3) notify_master_mysql.sh
   
        #!/bin/bash
        ###当keepalived监测到本机转为MASTER状态时，执行该脚本
         
        change_log=/etc/keepalived/logs/state_change.log
        mysql_con='mysql -uroot -p123456'
        echo -e "`date "+%F  %H:%M:%S"`   -----keepalived change to MASTER-----" >> $change_log
         
        slave_info() {
            ###统一定义一个函数取得slave的position、running、和log_file等信息
            ###根据函数后面所跟参数来决定取得哪些数据
            if [ $1 = slave_status ];then
                slave_stat=`${mysql_con} -e "show slave status\G;"|egrep -w "Slave_IO_Running|Slave_SQL_Running"`
                Slave_IO_Running=`echo $slave_stat|awk '{print $2}'`
                Slave_SQL_Running=`echo $slave_stat|awk '{print $4}'`
            elif [ $1 = log_file -a $2 = pos ];then
                log_file_pos=`${mysql_con} -e "show slave status\G;"|egrep -w "Master_Log_File|Read_Master_Log_Pos|Relay_Master_Log_File|Exec_Master_Log_Pos"`
                Master_Log_File=`echo $log_file_pos|awk '{print $2}'`
                Read_Master_Log_Pos=`echo $log_file_pos|awk '{print $4}'`
                Relay_Master_Log_File=`echo $log_file_pos|awk '{print $6}'`
                Exec_Master_Log_Pos=`echo $log_file_pos|awk '{print $8}'`
            fi
        }
         
        action() {
            ###经判断'应该&可以'切换时执行的动作
            echo -e "`date "+%F  %H:%M:%S"`    -----set read_only = 0 on DB2-----" >> $change_log
         
            ###解除read_only属性
            ${mysql_con} -e "set global read_only = 0;" 2>> $change_log
         
            echo "DB2 keepalived转为MASTER状态，线上数据库切换至DB2"|mail -s "DB2 keepalived change to MASTER"\
            liyansuper@outlook.com 2>> $change_log
         
            echo -e "---------------------------------------------------------\n" >> $change_log
        }
         
        slave_info slave_status
        if [ $Slave_SQL_Running = Yes ];then
            i=0    #一个计数器
            slave_info log_file pos
                ###判断从master接收到的binlog是否全部在本地执行(这样仍无法完全确定从库已追上主库，因为无法完全保证io_thread没有延时(由网络传输问题导致的从库落后的概率很小)
            until [ $Master_Log_File = $Relay_Master_Log_File -a $Read_Master_Log_Pos = $Exec_Master_Log_Pos ]
             do
                if [ $i -lt 10 ];then    #将等待exec_pos追上read_pos的时间限制为10s
                    echo -e "`date "+%F  %H:%M:%S"`    -----Relay_Master_Log_File=$Relay_Master_Log_File,Exec_Master_Log_Pos=$Exec_Master_Log_Pos is behind Master_Log_File=$Master_Log_File,Read_Master_Log_Pos=$Read_Master_Log_Pos, wait......" >> $change_log    #输出消息到日志，等待exec_pos=read_pos
                    i=$(($i+1))
                    sleep 1
                    slave_info log_file pos
                else
                    echo -e "The waits time is more than 10s,now force change. Master_Log_File=$Master_Log_File Read_Master_Log_Pos=$Read_Master_Log_Pos Relay_Master_Log_File=$Relay_Master_Log_File Exec_Master_Log_Pos=$Exec_Master_Log_Pos" >> $change_log
                    action
                    exit 0
                fi
            done
            action 
         
        else
            slave_info log_file pos
            echo -e "DB2's slave status is wrong,now force change. Master_Log_File=$Master_Log_File Read_Master_Log_Pos=$Read_Master_Log_Pos Relay_Master_Log_File=$Relay_Master_Log_File Exec_Master_Log_Pos=$Exec_Master_Log_Pos" >> $change_log
            action
        fi