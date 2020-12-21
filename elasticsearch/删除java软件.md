删除java软件

```shell
rpm -qa | grep java | xargs sudo rpm -e --nodeps
```



两个关键点：
 1.如果提示有xxx.rpm包已经被installed了，那么先用rpm -e –nodeps xxx来卸载
 2.如果存在多个版本的话，用rpm -e –allmatches来卸载

```shell
[root@mysqldb2 ~]# rpm -e –nodeps perl-URI
 [root@mysqldb2 ~]# rpm -e –nodeps perl-DBD-MySQL
 [root@mysqldb2 ~]# rpm -e –nodeps perl-Compress-Zlib
 [root@mysqldb2 ~]# rpm -e –nodeps perl-HTML-Tagset
 [root@mysqldb2 ~]# rpm -e –nodeps perl-HTML-Parser
 [root@mysqldb2 ~]# rpm -e –nodeps perl-libwww-perl
 [root@mysqldb2 ~]# rpm -e –nodeps mysql
 error: “mysql” specifies multiple packages
 [root@mysqldb2 ~]#
 [root@mysqldb2 ~]# rpm -qa | grep mysql
 mysql-5.0.77-4.el5_6.6
 libdbi-dbd-mysql-0.8.1a-1.2.2
 mysql-5.0.77-4.el5_6.6
 mysql-server-5.0.77-4.el5_6.6
 mysql-connector-odbc-3.51.26r1127-2.el5
 [root@mysqldb2 ~]# rpm -e –allmatches libdbi-dbd-mysql-0.8.1a-1.2.2
 [root@mysqldb2 ~]# rpm -e –allmatches mysql-connector-odbc-3.51.26r1127-2.el5
 [root@mysqldb2 ~]# rpm -e –allmatches mysql-server-5.0.77-4.el5_6.6
 [root@mysqldb2 ~]# rpm -e –allmatches mysql-5.0.77-4.el5_6.6
 error: Failed dependencies:
 libmysqlclient_r.so.15()(64bit) is needed by (installed) MySQL-python-1.2.3-0.1.c1.el5.x86_64
 libmysqlclient_r.so.15(libmysqlclient_15)(64bit) is needed by (installed) MySQL-python-1.2.3-0.1.c1.el5.x86_64
 [root@mysqldb2 ~]# rpm -e –allmatches MySQL-python-1.2.3-0.1.c1.el5.x86_64
 [root@mysqldb2 ~]# rpm -e –allmatches mysql-5.0.77-4.el5_6.6
 [root@mysqldb2 ~]# rpm -qa | grep mysql
 [root@mysqldb2 ~]
```

