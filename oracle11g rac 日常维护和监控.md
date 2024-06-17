# oracle11g rac 日常维护和监控



![image-20240615095334257](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615095334257.png) 

![image-20240615095546755](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615095546755.png) 

```
alter system set audit_trail=none scope=spfile;

su - root 
cd /backup
tar -zcvf oracle.tar.gz /oracle

du -sh /oracle
13G	/oracle



```



![image-20240615095640894](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615095640894.png) 

```
alter tablespace tjdata01 add datafile '+dgdata01' size 50M autoextend off;
```



数据文件，生产环境20g一个

![image-20240615095708873](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615095708873.png) 

![image-20240615095720692](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615095720692.png) 

![image-20240615095740568](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615095740568.png) 





## 180天密码过期

```
select * from dba_profiles where profile='DEFAULT';
DEFAULT 		       PASSWORD_LIFE_TIME		PASSWORD
180

alter profile default limit PASSWORD_LIFE_TIME UNLIMITED;


```

## 关闭审计

```
alter system set audit_trail=none scope=spfile;

show parameter audit_trail;

SQL> show parameter audit_trail;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
audit_trail			     string	 DB

SQL> show parameter audit_trail;

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
audit_trail			     string	 NONE


修改审计，需要重启数据库才能生效
srvctl start db -d yxdb
srvctl stop db -d yxdb
```

## asm使用

![image-20240615101426566](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615101426566.png) 

![image-20240615101958150](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615101958150.png) 

![image-20240615102119328](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615102119328.png) 





```
创建表空间
create tablespace tjdata01 datafile '+dgdata01' size 50M autoextend off extent management local segment space management auto;
查询表空间
select name from v$tablespace;

给表空间添加数据文件
alter tablespace tjdata01 add datafile '+dgdata01' size 50M autoextend off;

在grid查看数据文件
ASMCMD> cd DATAFILE
ASMCMD> ls
TJDATA01.263.1171707385
TJDATA01.264.1171707531
创建用户
create user itpux01 identified by itpux01 default tablespace tjdata01;
grant dba to itpux01;
插入数据
conn itpux01/itpux01;
create table fgtable01(
id number,
name varchar2(20)
);
insert into fgtable01 values(1,'itpux1');
insert into fgtable01 values(2,'itpux2');
commit;
select * from fgtable01;

```

## 日志文件

![image-20240615102543062](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615102543062.png)

```
SQL> show parameter dump

NAME				     TYPE	 VALUE
------------------------------------ ----------- ------------------------------
background_core_dump		     string	 partial
background_dump_dest		     string	 /oracle/app/oracle/diag/rdbms/
						 yxdb/yxdb_1/trace
core_dump_dest			     string	 /oracle/app/oracle/diag/rdbms/
						 yxdb/yxdb_1/cdump
max_dump_file_size		     string	 unlimited
shadow_core_dump		     string	 partial
user_dump_dest			     string	 /oracle/app/oracle/diag/rdbms/
						 yxdb/yxdb_1/trace
SQL> 

部分的 partial
无限的 unlimited
查出 trace
/oracle/app/oracle/diag/rdbms/yxdb/yxdb_1/trace
/oracle/app/oracle/diag/rdbms/yxdb/yxdb_1/cdump
/oracle/app/oracle/diag/rdbms/yxdb/yxdb_1/trace


ls /oracle/app/oracle/diag/rdbms/yxdb/yxdb_1/trace/*|grep alert
/oracle/app/oracle/diag/rdbms/yxdb/yxdb_1/trace/alert_yxdb_1.log

[oracle@yxdb81:/oracle/app/oracle/diag/rdbms/yxdb/yxdb_1/trace]$ls -als al*
328 -rw-r----- 1 oracle asmadmin 331597 Jun 15 10:18 alert_yxdb_1.log

搜索ORA-开头的信息
vim alert_yxdb_1.log
ORA-1109 signalled during: ALTER DATABASE CLOSE NORMAL...
ORA-30013: undo tablespace 'UNDOTBS3' is currently in use
/ORA-


第3台机器的

cd /oracle/app/oracle/diag/rdbms/yxdb


[oracle@yxdb83:/root]$cd /oracle/app/oracle/diag/rdbms/yxdb
[oracle@yxdb83:/oracle/app/oracle/diag/rdbms/yxdb]$ls
i_1.mif  yxdb_3
[oracle@yxdb83:/oracle/app/oracle/diag/rdbms/yxdb]$cd yxdb_3/
[oracle@yxdb83:/oracle/app/oracle/diag/rdbms/yxdb/yxdb_3]$ls
alert  hm        incpkg  lck       metadata_dgif  stage  trace
cdump  incident  ir      metadata  metadata_pv    sweep
[oracle@yxdb83:/oracle/app/oracle/diag/rdbms/yxdb/yxdb_3]$cd trace/
[oracle@yxdb83:/oracle/app/oracle/diag/rdbms/yxdb/yxdb_3/trace]$pwd
/oracle/app/oracle/diag/rdbms/yxdb/yxdb_3/trace
[oracle@yxdb83:/oracle/app/oracle/diag/rdbms/yxdb/yxdb_3/trace]$ls -las al*
280 -rw-r----- 1 oracle asmadmin 282400 Jun 15 10:08 alert_yxdb_3.log


监听的日志

lsnrctl status

Listener Parameter File   /oracle/app/11.2.0/grid/network/admin/listener.ora
Listener Log File         /oracle/app/grid/diag/tnslsnr/yxdb82/listener/alert/log.xml

监听参数文件
vim /oracle/app/11.2.0/grid/network/admin/listener.ora

LISTENER_SCAN1=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=IPC)(KEY=LISTENER_SCAN1))))                # line added by Agent
LISTENER=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=IPC)(KEY=LISTENER))))            # line added by Agent
ENABLE_GLOBAL_DYNAMIC_ENDPOINT_LISTENER=ON              # line added by Agent
ENABLE_GLOBAL_DYNAMIC_ENDPOINT_LISTENER_SCAN1=ON                # line added by Agent
~                                     


监听的日志文件

vim /oracle/app/grid/diag/tnslsnr/yxdb82/listener/alert/log.xml

ls /oracle/app/grid/diag/tnslsnr/yxdb82/listener/alert/
log.xml




集群的日志
[root@yxdb83 ~]# su grid
[grid@yxdb83:/root]$sqlplus "/as sysasm"

SQL> show parameter dump

NAME				     TYPE
------------------------------------ ----------------------
VALUE
------------------------------
background_core_dump		     string
partial
background_dump_dest		     string
/oracle/app/grid/diag/asm/+asm
/+ASM3/trace
core_dump_dest			     string
/oracle/app/grid/diag/asm/+asm
/+ASM3/cdump
max_dump_file_size		     string

NAME				     TYPE
------------------------------------ ----------------------
VALUE
------------------------------
unlimited
shadow_core_dump		     string
partial
user_dump_dest			     string
/oracle/app/grid/diag/asm/+asm
/+ASM3/trace

asm的日志
/oracle/app/grid/diag/asm/+asm/+ASM3/trace

[grid@yxdb83:/root]$ls /oracle/app/grid/diag/asm/+asm/+ASM3/trace/al* -lsa
84 -rw-r----- 1 grid oinstall 81036 Jun 15 10:02 /oracle/app/grid/diag/asm/+asm/+ASM3/trace/alert_+ASM3.log


集群的日志

more /oracle/app/11.2.0/grid/log/yxdb83/alertyxdb83.log
tail -100f /oracle/app/11.2.0/grid/log/yxdb83/alertyxdb83.log

[ctssd(3756)]CRS-2408:The clock on host yxdb83 has been updated by the Cluster Time Synchronization Service to be synchronous with the mean cluster time.

集群软件asm监听数据库实例

em 日志
/oracle/app/oracle/product/11.2.0/db_1/yxdb81_yxdb/sysman/log


```

## em管理器





```
root@yxdb81:/oracle]$su oracle
[oracle@yxdb81:/oracle]$emctl status dbconsole
Oracle Enterprise Manager 11g Database Control Release 11.2.0.4.0 
Copyright (c) 1996, 2013 Oracle Corporation.  All rights reserved.
https://yxdb81:1158/em/console/aboutApplication
Oracle Enterprise Manager 11g is not running. 
------------------------------------------------------------------
Logs are generated in directory /oracle/app/oracle/product/11.2.0/db_1/yxdb81_yxdb/sysman/log

启动em
emctl start dbconsole

https://192.168.2.181/em/console

sys
oracle
sysdba


```

 ![image-20240615124849785](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615124849785.png)



## 启动和停止

关闭数据库

```
1 先切换日志
alter system switch logfile;
/
/
/
将目前没有写的数据库写进去。

2： alter system checkpoint;

3 shutdown immediate;

3 crsctl disable crs  禁用开机启动，集群软件，因为启动主机，但是存储还没有连上，启动会导致出现问题，启动不了。

4 crsctl stop crs 
5 shutdown -h 0




```

### 启动

![image-20240615130602757](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615130602757.png) 

![image-20240615130644891](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615130644891.png) 

这4个全部变成online状态才可以启动

![image-20240615130820315](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615130820315.png)

![image-20240615130924912](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615130924912.png) 



![image-20240615130940252](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615130940252.png) 

建议数据库一个一个的启动



```
1 检查网卡
2 检查磁盘 共享存储

3 检测 crsctl check crs 

4 启动 crsctl start crs
5 sqlplus "/as sysdba"  startup


```





## 命令

## crsctl

```
crsctl stat res -t
crsctl start crs
crsctl disable crs
crsctl status  crs
crsctl stop crs 
```





### srvctl 

![image-20240615131109709](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615131109709.png)

![image-20240615131128408](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615131128408.png) 

 ![image-20240615131143034](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615131143034.png)





### orchcheck



### crs_stat

![image-20240615131241451](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615131241451.png)



### asmcmd

![image-20240615131311843](C:\Users\m1861\AppData\Roaming\Typora\typora-user-images\image-20240615131311843.png) 