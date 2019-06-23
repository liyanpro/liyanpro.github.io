---
title: PL/SQL程序块
date: 2017-04-01 11:04:20
tags:
    - sql
---

最近接触到oracle的pl/sql程序块的书写，对pl/sql的语法做一些总结。
它是可扩展性的SQL语言，将变量、控制结构、过程和函数等结构化程序设计的要素引入到了SQL语言中，解释执行，面向过程，对大小写不敏感。
1. PL/SQL数据类型
（1）基本数据类型
        数字型 NUMBER(n) NUMBER(m,n)   --后面的n表示保留的小数位数
        数值型 INT INTEGER BINARG_INTEGER(带符号的） (+-号)LONG（最长2GB） 
        字符型 CHAR VARCHAR2（最大2000个字符）
        日期型 DATE布尔型 BOOLEAN（TRUE，FALSE，NULL三者之一）
标识符命名规范: 必须以字母开关，不能有空格；不能超过30个字符； 不能与保留字同名；常量（变量）名称不区分大小写，在字母后可带数字和特殊字符#
（2）复合型数据类型
       %TYPE     列
       %ROWTYPE  行

    %TYPE：动态获得表中列（行）的数据类型，该变量的数据类型与指定表中的列（行）的数据类型一致，一旦表中的列（行）的类型变化了它会自动获得新的数据类型。
        TYPE 记录类型名 IS RECORD ( 变量名 类型， 变量名 类型)
记录型是一个新的数据类型，用户自定义的，与JAVA中的类相似，例：
        TYPE PERSON IS RECORD ( ID NUMBER(2), NAME VARCHAR2(20))
2. PL/SQL程序块结构
（3）表达式
常用表达式不再累赘，就函数列举几个：
    日期和字符串转换函数：
       TO_CHAR(date,string)TO_CHAR(string,'yyyy-mm-dd')
    日期加减函数：
       ADDMONTHS(date,number)

 ADDMONTHS(TO_CHAR(‘2016-10-01’,’yyyy,mm,dd’),-12)表示年份减一

 （4）程序块的基本结构
    以FUNCTION函数为例
        CREATE or REPLACE FUNCTION function_name(实参 参数类型) 
        RETURN 返回参数类型 is变量名 变量类型 DECLARE --声明部分 
        BEGIN            --执行部分
        END              --执行部分结束
        RETURN 返回参数   --返回结果 
        END              --函数结束

 ① DECLARE
在该部分定义程序中用的常量，变量，游标 ，异常， PLSQL 用到的所有定义必须在该部份集中定义，不允许在执行过程中定义。
② BEGIN
执行部份必须以Begin开始，end结束 ，该部分是每个 PLSQL必备的，包含了对数据库的操作和流程控制语句
③ EXCEPTION
该部份属于执行部份，对程序执行中产生的异常进行处理，一般放在 PL/SQL 程序体的后半部,结构为 :
EXCEPTION WHEN 异常类型1 THEN 处理代码1 WHEN 异常类型2 THEN 处理代码2 WHEN OTHERS THEN 处理其它异常代码END;
④ 游标概念
游标是从数据表中提取出来的数据，以临时表的形式存放在内存(上下文，工作区)中，在游标中有一个数据指针，在初始状态下指向的是首部，利用fetch语句可以移动该指针，从而对游标中的数据进行操作， 然后将操作后的结果写回表中
CURSOR 游标名 IS SELECT语句; --声明游标OPEN C_EMP; --打开游标FETCH C_EMP INTO V_EMP; --提取数据CLOSE C_EMP; --关闭游标
⑤循环
游标的for循环：
FOR rowtype变量 IN 游标名|查询语句LOOP 循环体END LOOP;
-----------------------------------------------------------------------------------------
    下面附录一段关于项目中绩效奖金计算的完整代码：
        create or replace function getJxjj(xmbh t_zxxm_lxs.n_lxsbh%type, yslx number) 
         return number is zjxjj number(16,2); --总绩效奖级
         sfzxm number(5); --项目中是否包含子项目
        begin select n_sfzxm into sfzxm from t_zxxm_lxs where n_lxsbh = xmbh; 
           select 0 into zjxjj from dual; --将总额初始化为0 --包含子项目  
           if(sfzxm=1) then declare cursor n_zbh is select n_lxsbh from t_zxxm_lxs where n_mainxmbh = xmbh;
          --定义一个游标变量v_cinfo c_emp%ROWTYPE ，该类型为游标c_emp中的一行数据类型 c_row n_zbh%rowtype; 
          begin for c_row in n_zbh loop --游标循环         
           if(judgeXmXz(c_row.n_lxsbh)=1   then 
            --等于1执行新计算方法 
            zjxjj:=zjxjj + nvl(getXTdJxjjByXmbh(c_row.n_lxsbh, yslx), 0) +nvl(getXPmJxjjByXmbh(c_row.n_lxsbh,yslx), 0);
            --调用团队和经理的新奖金计算方法，n_lxsbh为表中的立项书编号
           else 
               zjxjj:=zjxjj + nvl(getTdJxjjByXmbh(c_row.n_lxsbh, yslx), 0) + nvl(getPmJxjjByXmbh(c_row.n_lxsbh, yslx), 0); --调用旧方法计算奖金           
           end if; --条件判断结束 
           end loop; --游标循环结束 
           end; --不包含子项目，没有循环 
        else if(judgeXmXz(xmbh)=1) then               
            zjxjj:=nvl(getXTdJxjjByXmbh(xmbh,yslx), 0) +  nvl(getXPmJxjjByXmbh(xmbh,yslx), 0);     
              else zjxjj:= nvl(getTdJxjjByXmbh(xmbh,yslx), 0) + nvl(getPmJxjjByXmbh(xmbh,yslx), 0); 
             end if; 
         end if; --外层条件判断结束 
       return zjxjj; --返回计算结果end getJxjj;   
