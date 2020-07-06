### mysql命令行参数

```
Usage: mysql [OPTIONS] [database]   //命令方式
 -?, --help          //显示帮助信息并退出
 -I, --help          //显示帮助信息并退出
 --auto-rehash       //自动补全功能，就像linux里面，按Tab键出提示差不多，下面有例子

 -A, --no-auto-rehash  //默认状态是没有自动补全功能的。-A就是不要自动补全功能
 -B, --batch         //ysql不使用历史文件，禁用交互
 (Enables --silent)
 --character-sets-dir=name   //字体集的安装目录                    
 --default-character-set=name    //设置数据库的默认字符集
 -C, --compress      //在客户端和服务器端传递信息时使用压缩
 -#, --debug[=#]     //bug调用功能
 -D, --database=name //使用哪个数据库
 --delimiter=name    //mysql默认命令结束符是分号，下面有例子
 -e, --execute=name  //执行mysql的sql语句
 -E, --vertical      //垂直打印查询输出
 -f, --force         //如果有错误跳过去，继续执行下面的
 -G, --named-commands
 /*Enable named commands. Named commands mean this program's
 internal commands; see mysql> help . When enabled, the
 named commands can be used from any line of the query,
 otherwise only from the first line, before an enter.
 Disable with --disable-named-commands. This option is
 disabled by default.*/
 -g, --no-named-commands
 /*Named commands are disabled. Use \* form only, or use
 named commands only in the beginning of a line ending
 with a semicolon (;) Since version 10.9 the client now
 starts with this option ENABLED by default! Disable with
 '-G'. Long format commands still work from the first
 line. WARNING: option deprecated; use
 --disable-named-commands instead.*/
 -i, --ignore-spaces //忽视函数名后面的空格.
 --local-infile      //启动/禁用 LOAD DATA LOCAL INFILE.
 -b, --no-beep       //sql错误时，禁止嘟的一声
 -h, --host=name     //设置连接的服务器名或者Ip
 -H, --html          //以html的方式输出
 -X, --xml           //以xml的方式输出
 --line-numbers      //显示错误的行号
 -L, --skip-line-numbers  //忽略错误的行号
 -n, --unbuffered    //每执行一次sql后，刷新缓存
 --column-names      //查寻时显示列信息，默认是加上的
 -N, --skip-column-names  //不显示列信息
 -O, --set-variable=name  //设置变量用法是--set-variable=var_name=var_value
 --sigint-ignore     //忽视SIGINT符号(登录退出时Control-C的结果)
 -o, --one-database  //忽视除了为命令行中命名的默认数据库的语句。可以帮跳过日志中的其它数据库的更新。
 --pager[=name]      //使用分页器来显示查询输出，这个要在linux可以用more,less等。
 --no-pager          //不使用分页器来显示查询输出。
 -p, --password[=name] //输入密码
 -P, --port=#        //设置端口
 --prompt=name       //设置mysql提示符
 --protocol=name     //使用什么协议
 -q, --quick         //不缓存查询的结果，顺序打印每一行。如果输出被挂起，服务器会慢下来，mysql不使用历史文件。
 -r, --raw           //写列的值而不转义转换。通常结合--batch选项使用。
 --reconnect         //如果与服务器之间的连接断开，自动尝试重新连接。禁止重新连接，使用--disable-reconnect。
 -s, --silent        //一行一行输出，中间有tab分隔
 -S, --socket=name   //连接服务器的sockey文件
 --ssl               //激活ssl连接，不激活--skip-ssl
 --ssl-ca=name       //CA证书
 --ssl-capath=name   //CA路径
 --ssl-cert=name     //X509 证书
 --ssl-cipher=name   //SSL cipher to use (implies --ssl).
 --ssl-key=name      //X509 密钥名
 --ssl-verify-server-cert //连接时审核服务器的证书
 -t, --table         //以表格的形势输出
 --tee=name          //将输出拷贝添加到给定的文件中，禁时用--disable-tee
 --no-tee            //根--disable-tee功能一样
 -u, --user=name     //用户名
 -U, --safe-updates  //Only allow UPDATE and DELETE that uses keys.
 -U, --i-am-a-dummy  //Synonym for option --safe-updates, -U.
 -v, --verbose       //输出mysql执行的语句
 -V, --version       //版本信息
 -w, --wait          //服务器down后，等待到重起的时间
 --connect_timeout=# //连接前要等待的时间
 --max_allowed_packet=# //服务器接收／发送包的最大长度
 --net_buffer_length=# //TCP / IP和套接字通信缓冲区大小。
 --select_limit=#    //使用--safe-updates时SELECT语句的自动限制
 --max_join_size=#   //使用--safe-updates时联接中的行的自动限制
 --secure-auth       //拒绝用(pre-4.1.1)的方式连接到数据库
 --server-arg=name   //Send embedded server this as a parameter.
 --show-warnings     //显示警告
```

### mysql命令行实例

1，auto-rehash自动补全

```
mysqld_safe --help |grep rehash
```

2，－B的用法

```
D:\xampp\mysql\bin>mysql.exe -uroot -D bak_test -e "show tables;" -B  
Tables_in_bak_test  
comment  
user
```

3，－E的用法

```
D:\xampp\mysql\bin>mysql.exe -uroot bak_test -e "show tables;" -E  
*************************** 1. row ***************************  
Tables_in_bak_test: comment  
*************************** 2. row ***************************  
Tables_in_bak_test: user
```

4，－D的用法

```
进入后默认就在test数据库里面，不要用use test;
[root@BlackGhost zhangy]# mysql -u root -D test
```

5，--default-character-set设置默认字符集

```
mysql -u root -D test  --default-character-set=utf8
```

6，--delimiter设置mysql命令结束符

```
mysql -u root -D test   --delimiter=\|
```

7，-e的用法

```
这个很有用的，因为我不用进入mysql客户里面去，就能把我要的数据取出来，这个可以配合shell脚本的话，能发挥很大的功能

D:\xampp\mysql\bin>mysql.exe -uroot -D bak_test -e "show tables;"
```

8，-f的用法

```
D:\xampp\mysql\bin>mysql.exe -uroot bak_test -e "show databaseds;show tables;" -  
f  
ERROR 1064 (42000) at line 1: You have an error in your SQL syntax; check the ma  
nual that corresponds to your MySQL server version for the right syntax to use n  
ear 'databaseds' at line 1  
+--------------------+  
| Tables_in_bak_test |  
+--------------------+  
| comment            |  
| user               |  
+--------------------+
```

9，-N的用法

```
D:\xampp\mysql\bin>mysql.exe -uroot bak_test -e "select * from user" -N  
+---+------+---+  
| 1 |   bb | 0 |  
| 2 | tank | 0 |  
+---+------+---+
```

10，-p的用法

```
mysql -u root -o test -p   -S /tmp/mysql.sock  
```

11，-h的用法

```
mysql -u root -h 192.168.1.102
```

12，-H的用法

```
D:\xampp\mysql\bin>mysql.exe -uroot bak_test -e "show tables  " -H  
<TABLE BORDER=1><TR><TH>Tables_in_bak_test</TH></TR><TR><TD>comment</TD></TR><TR  
><TD>user</TD></TR></TABLE>
```

13，-X的用法

```
D:\xampp\mysql\bin>mysql.exe -uroot bak_test -e "show tables  " -X  
<?xml version="1.0"?>   
 
<resultset statement="show tables 
" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">  
<row>  
<field name="Tables_in_bak_test">comment</field>  
</row>   
 
<row>  
<field name="Tables_in_bak_test">user</field>  
</row>  
</resultset>
```

14，--prompt的用法

```
mysql -u root --prompt=\^\_\^  
^_^show databases;  
+--------------------+  
| Database           |  
+--------------------+  
| information_schema |  
| biztojie           |
```

15，-S的用法

```
mysql -u root -D test   -S /tmp/mysql.sock
```

16，-v的用法

```
mysql -u root -D test -e "show tables;"   -v  
--------------  
show tables  
--------------
```

17，-P的用法

```
mysql -u root -o test  -P 13306  -S /tmp/mysql.sock
```