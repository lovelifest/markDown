# ON DUPLICATE KEY UPDATE作用

**先声明一点，ON DUPLICATE KEY UPDATE为Mysql特有语法，这是个坑**
语句的作用，当insert已经存在的记录时，执行Update

### 用法

**什么意思？举个例子：**
user_admin_t表中有一条数据如下

![user_admin_t](https://img-blog.csdn.net/20171105150445430?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenliMjAxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

表中的主键为id，现要插入一条数据，id为‘1’，password为‘第一次插入的密码’，正常写法为：

```
INSERT INTO user_admin_t (_id,password) 
VALUES ('1','第一次插入的密码') 12
```

执行后刷新表数据，我们来看表中内容

![执行insert后](https://img-blog.csdn.net/20171105150809525?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenliMjAxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

此时表中数据增加了一条主键’_id’为‘1’，‘password’为‘第一次插入的密码’的记录，当我们再次执行插入语句时，会发生什么呢？

```
-- 执行
INSERT INTO user_admin_t (_id,password) 
VALUES ('1','第一次插入的密码') 123
[SQL]INSERT INTO user_admin_t (_id,password) 
VALUES ('1','第一次插入的密码') 

[Err] 1062 - Duplicate entry '1' for key 'PRIMARY'
12345
```

Mysql告诉我们，我们的主键冲突了，看到这里我们是不是可以改变一下思路，当插入已存在主键的记录时，将插入操作变为修改：

```
-- 在原sql后面增加 ON DUPLICATE KEY UPDATE 
INSERT INTO user_admin_t (_id,password) 
VALUES ('1','第一次插入的密码') 
ON DUPLICATE KEY UPDATE 
_id = 'UpId',
password = 'upPassword';123456
```

我们再一次执行：

```
[SQL]INSERT INTO user_admin_t (_id,password) 
VALUES ('1','第一次插入的密码') 
ON DUPLICATE KEY UPDATE 
_id = 'UpId',
password = 'upPassword';
受影响的行: 2
时间: 0.131s1234567
```

可以看到 受影响的行为2，这是因为将原有的记录修改了，而不是执行插入，看一下表中数据：

![DUPLICATE后](https://img-blog.csdn.net/20171105151541091?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenliMjAxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

原本‘id’为‘1’的记录，改为了‘UpId’，‘password’也变为了‘upPassword’，很好的解决了重复插入问题

### 扩展

当插入多条数据，其中不只有表中已存在的，还有需要新插入的数据，Mysql会如何执行呢？会不会报错呢？

其实Mysql远比我们想象的强大，他会智能的选择更新还是插入，我们尝试一下：

```
INSERT INTO user_admin_t (_id,password) 
VALUES 
('1','第一次插入的密码') ,
('2','第二条记录')
ON DUPLICATE KEY UPDATE 
_id = 'UpId',
password = 'upPassword';1234567
```

运行sql

```
[SQL]INSERT INTO user_admin_t (_id,password) 
VALUES 
('1','第一次插入的密码') ,
('2','第二条记录')
ON DUPLICATE KEY UPDATE 
_id = 'UpId',
password = 'upPassword';
受影响的行: 3
时间: 0.045s

1234567891011
```

Mysql执行了一次修改，一次插入，表中数据为：

![多记录插入](https://img-blog.csdn.net/20171105153119038?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenliMjAxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### VALUES修改

那么问题又来了，有人会说我ON DUPLICATE KEY UPDATE 后面跟的是固定的值，如果我想要分别给不同的记录插入不同的值怎么办呢？

```
INSERT INTO user_admin_t (_id,password) 
VALUES 
('1','多条插入1') ,
('UpId','多条插入2')
ON DUPLICATE KEY UPDATE 
password =  VALUES(password);
1234567
```

方法之一可以将后面的修改条件改为VALUES(password)，动态的传入要修改的值，执行以下：

```
[SQL]INSERT INTO user_admin_t (_id,password) 
VALUES 
('1','多条插入1') ,
('UpId','多条插入2')
ON DUPLICATE KEY UPDATE 
password =  VALUES(password);
受影响的行: 4
时间: 0.187s

12345678910
```

成功的修改了两条记录，刷新一下表

![多条修改](https://img-blog.csdn.net/20171105153701640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvenliMjAxNw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们成功的为不同id的password修改成了不同的值

### 总结

其实修改的方法有很多种，包括SET或用REPLACE，连事务都省的做，ON DUPLICATE KEY UPDATE能够让我们便捷的完成重复插入的开发需求，但它是Mysql的特有语法，使用时应多注意主键和插入值是否是我们想要插入或修改的key、Value。



# [MYSQL中replace into的用法](https://www.cnblogs.com/c-961900940/p/6197878.html)

**新建一个test表，三个字段，id，title，uid, id是自增的主键，uid是唯一索引；**

插入两条数据

```
insert into  test(title,uid) VALUES ('123465','1001');
insert into  test(title,uid) VALUES ('123465','1002');执行单条插入数据可以看到，执行结果如下：
[SQL]insert into  test(title,uid) VALUES ('123465','1001');
受影响的行: 1
时间: 0.175s
```



使用 replace into插入数据时：

```
REPLACE INTO test(title,uid) VALUES ('1234657','1003');

执行结果：
[SQL]REPLACE INTO test(title,uid) VALUES ('1234657','1003');
受影响的行: 1
时间: 0.035s
```

当前数据库test表所有数据如下：

![img](https://images2015.cnblogs.com/blog/553900/201612/553900-20161219160539932-1025929232.png)

 

当uid存在时，使用replace into 语句

```
REPLACE INTO test(title,uid) VALUES ('1234657','1001');

[SQL]REPLACE INTO test(title,uid) VALUES ('1234657','1001');
受影响的行: 2
时间: 0.140s
```

![img](https://images2015.cnblogs.com/blog/553900/201612/553900-20161219160702807-1188018521.png)

 

replace into t(id, update_time) values(1, now());

或

replace into t(id, update_time) select 1, now();

replace into 跟 insert 功能类似，不同点在于：replace into 首先尝试插入数据到表中， **1. 如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据。 2. 否则，直接插入新数据。**

要注意的是：插入数据的表必须有主键或者是唯一索引！否则的话，replace into 会直接插入数据，这将导致表中出现重复的数据。

MySQL replace into 有三种形式：

\1. replace into tbl_name(col_name, ...) values(...)

\2. replace into tbl_name(col_name, ...) select ...

\3. replace into tbl_name set col_name=value, ...

第一种形式类似于insert into的用法，

第二种replace select的用法也类似于insert select，这种用法并不一定要求列名匹配，事实上，MYSQL甚至不关心select返回的列名，它需要的是列的位置。例如，replace into tb1( name, title, mood) select rname, rtitle, rmood from tb2;?这个例子使用replace into从?tb2中将所有数据导入tb1中。

第三种replace set用法类似于update set用法，使用一个例如“SET col_name = col_name + 1”的赋值，则对位于右侧的列名称的引用会被作为DEFAULT(col_name)处理。因此，该赋值相当于SET col_name = DEFAULT(col_name) + 1。

前两种形式用的多些。其中 “into” 关键字可以省略，不过最好加上 “into”，这样意思更加直观。另外，对于那些没有给予值的列，MySQL 将自动为这些列赋上默认值。

