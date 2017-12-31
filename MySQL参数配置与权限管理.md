# MySQL参数配置与权限管理
## MySQL客户端连接
### 配置文件简单介绍
参考：https://github.com/jdaaaaaavid/mysql_best_configuration/blob/master/my.cnf

```
//客户端登录mysql服务的参数配置
[client]
user=root


//通过mysql命令登录服务的参数配置
[mysql]

//mysql服务启动时需要的参数配置
[mysqld]    

```


### 命令介绍
```
//列出配置参数
(root@localhost)[(none)]> show variables\G
(root@localhost)[(none)]> show variables like 'log_error'\G
(root@localhost)[(none)]> show variables like 'innodb%';
(root@localhost)[(none)]> show variables like '%log%';
(root@localhost)[(none)]> show variables like '%data%'; 

(root@localhost)[(none)]> set session long_query_time = 1.5;
(root@localhost)[(none)]> set long_query_time = 1.5;
```

```
 //查看每个参数对应的默认值
 ✗ mysqld --help --verbose
```

### MySQL配置参数
1. 从作用域上可 分为global（全局）和session （会话级别）

通过global修改参数不影响当前session的参数值！！！！！！

|命令|说明|
|:---|:---|
|show [session]\|[global] variables [like 'pattern']| 查看变量 可通过lIke进行过滤|
|set [session]\|[global] variables_key = xxx |修改global或者session的参数值|

```
(root@localhost)[(none)]> set session long_query_time = 1.5;
(root@localhost)[(none)]> set long_query_time = 1.5;
(root@localhost)[(none)]> show variables like 'long_%' ;
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 1.500000 |
+-----------------+----------+
1 row in set (0.00 sec)

(root@localhost)[(none)]> show global variables like 'long_%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)

(root@localhost)[(none)]> set global long_query_time = 3;


5.7版本的库可以从performance_schema来查看这些信息
(root@localhost)[(none)]> use performace_schema;
(root@localhost)[performance_schema]> show tables like '%variables%';
+--------------------------------------------+
| Tables_in_performance_schema (%variables%) |
+--------------------------------------------+
| global_variables                           |
| session_variables                          |
| user_variables_by_thread                   |
| variables_by_thread                        |
+--------------------------------------------+
4 rows in set (0.00 sec)

(root@localhost)[performance_schema]> select * from threads where PROCESSLIST_ID = 7\G
*************************** 1. row ***************************
          THREAD_ID: 35  ------线程ID
               NAME: thread/sql/one_connection
               TYPE: FOREGROUND
     PROCESSLIST_ID: 7 -------show processlist 里的 id
   PROCESSLIST_USER: root
   PROCESSLIST_HOST: localhost
     PROCESSLIST_DB: NULL
PROCESSLIST_COMMAND: Sleep
   PROCESSLIST_TIME: 393
  PROCESSLIST_STATE: NULL
   PROCESSLIST_INFO: NULL
   PARENT_THREAD_ID: 1----------
               ROLE: NULL
       INSTRUMENTED: YES
            HISTORY: YES
    CONNECTION_TYPE: Socket
       THREAD_OS_ID: 2433845 -------当前线程所在的进程ID
1 row in set (0.00 sec)


```
2. 从类型上可分为 可修改参数和只读参数

```
(root@localhost)[(none)]> set global datadir = '/tmp/';
ERROR 1238 (HY000): Variable 'datadir' is a read only variable
```
3. 用户可在线修改非只读参数
4. 只读参数只能通过修改配置文件修改并重启生效
5. 所有参数的修改都不持久化

##  MySQL权限管理

用户名和IP是否允许 ---> 查看mysql.user表 -----> 查看mysql.db表 -----> 查看mysql.table_priv表 -----> 查看mysql.column_priv表 ----->  提示用户没有权限

文档：https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html
### 注意：
1. 删除所有用户名为空的用户
2. 不允许密码为空的用户存在
3. 管理员可以有所有库的权限
4. 开发权限只需要给相应库的权限即可

### 创建用户
文档：https://dev.mysql.com/doc/refman/5.7/en/create-user.html

```

//用户名@IP identified by 密码
(root@localhost)[performance_schema]> create user 'test1'@'%' identified by '123';
Query OK, 0 rows affected (0.10 sec)


//删除某个用户
(root@localhost)[performance_schema]> drop user 'test1';
Query OK, 0 rows affected (0.01 sec) 

//查看当前用户的权限
(root@localhost)[performance_schema]> show grants;
(root@localhost)[performance_schema]> show grants for 'test1';
+-----------------------------------+
| Grants for test1@%                |
+-----------------------------------+
| GRANT USAGE ON *.* TO 'test1'@'%' |
+-----------------------------------+
1 row in set (0.00 sec)

//grant 权限关键字 on 库.表 to 用户
(root@localhost)[performance_schema]> grant select,insert,update,delete on test.* to 'test1';
Query OK, 0 rows affected (0.02 sec)

(root@localhost)[performance_schema]> show grants for 'test1';
+-----------------------------------------------------------------+
| Grants for test1@%                                              |
+-----------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test1'@'%'                               |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `test`.* TO 'test1'@'%' |
+-----------------------------------------------------------------+
2 rows in set (0.00 sec)

//继续新增权限
(root@localhost)[test]> grant create,index on test.* to test1;
Query OK, 0 rows affected (0.00 sec)

(root@localhost)[test]> show grants for test1;
+--------------------------------------------------------------------------------+
| Grants for test1@%                                                             |
+--------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test1'@'%'                                              |
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, INDEX ON `test`.* TO 'test1'@'%' |
+--------------------------------------------------------------------------------+
2 rows in set (0.00 sec)


//删掉权限
//revoke 权限关键字 on 库.表 from 用户 (revoke仅仅是收回权限 但是不会删除用户)
(root@localhost)[test]> revoke create,index on test.* from test1;
Query OK, 0 rows affected (0.01 sec)
(root@localhost)[test]> show grants for test1;
+-----------------------------------------------------------------+
| Grants for test1@%                                              |
+-----------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test1'@'%'                               |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `test`.* TO 'test1'@'%' |
+-----------------------------------------------------------------+

//删除所有权限 revoke all 
(root@localhost)[test]> revoke all on test.* from test1;
Query OK, 0 rows affected (0.00 sec)

(root@localhost)[test]> show grants for test1;
+-----------------------------------+
| Grants for test1@%                |
+-----------------------------------+
| GRANT USAGE ON *.* TO 'test1'@'%' |
+-----------------------------------+


//5.6之前可以用这个，两个语句连起来创建用户和密码 5.7开始不推荐这种做法 所以会出现一个warning
(root@localhost)[test]> grant select,insert,update,delete on test.* to 'test2' identified by '123' ;
Query OK, 0 rows affected, 1 warning (0.00 sec)

(root@localhost)[test]> show warnings\G
*************************** 1. row ***************************
  Level: Warning
   Code: 1287
Message: Using GRANT for creating new user is deprecated and will be removed in future release. Create new user with CREATE USER statement.
1 row in set (0.01 sec)

//修改密码
(root@localhost)[test]> alter user test2 identified by '456';

//with grant option
(root@localhost)[test]> grant select on test.* to test2 with grant option;
Query OK, 0 rows affected (0.00 sec)

(root@localhost)[test]> show grants for test2 ;
+-----------------------------------------------------------------------------------+
| Grants for test2@%                                                                |
+-----------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test2'@'%'                                                 |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `test`.* TO 'test2'@'%' WITH GRANT OPTION |
+-----------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

```

### mysql库权限管理

```
(root@localhost)[(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
//全局的
(root@localhost)[mysql]> show tables like 'user';


(root@localhost)[mysql]> show tables like 'db';


(root@localhost)[mysql]> show tables like 'tables_priv';


(root@localhost)[mysql]> show tables like 'columns_priv';

(root@localhost)[mysql]> desc user;
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Field                  | Type                              | Null | Key | Default               | Extra |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
| Host                   | char(60)                          | NO   | PRI |                       |       |
| User                   | char(32)                          | NO   | PRI |                       |       |
| Select_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Insert_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Update_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Delete_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Create_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Drop_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Reload_priv            | enum('N','Y')                     | NO   |     | N                     |       |
| Shutdown_priv          | enum('N','Y')                     | NO   |     | N                     |       |
| Process_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| File_priv              | enum('N','Y')                     | NO   |     | N                     |       |
| Grant_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| References_priv        | enum('N','Y')                     | NO   |     | N                     |       |
| Index_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Show_db_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Super_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tmp_table_priv  | enum('N','Y')                     | NO   |     | N                     |       |
| Lock_tables_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Execute_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_slave_priv        | enum('N','Y')                     | NO   |     | N                     |       |
| Repl_client_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Create_view_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Show_view_priv         | enum('N','Y')                     | NO   |     | N                     |       |
| Create_routine_priv    | enum('N','Y')                     | NO   |     | N                     |       |
| Alter_routine_priv     | enum('N','Y')                     | NO   |     | N                     |       |
| Create_user_priv       | enum('N','Y')                     | NO   |     | N                     |       |
| Event_priv             | enum('N','Y')                     | NO   |     | N                     |       |
| Trigger_priv           | enum('N','Y')                     | NO   |     | N                     |       |
| Create_tablespace_priv | enum('N','Y')                     | NO   |     | N                     |       |
| ssl_type               | enum('','ANY','X509','SPECIFIED') | NO   |     |                       |       |
| ssl_cipher             | blob                              | NO   |     | NULL                  |       |
| x509_issuer            | blob                              | NO   |     | NULL                  |       |
| x509_subject           | blob                              | NO   |     | NULL                  |       |
| max_questions          | int(11) unsigned                  | NO   |     | 0                     |       |
| max_updates            | int(11) unsigned                  | NO   |     | 0                     |       |
| max_connections        | int(11) unsigned                  | NO   |     | 0                     |       |
| max_user_connections   | int(11) unsigned                  | NO   |     | 0                     |       |
| plugin                 | char(64)                          | NO   |     | mysql_native_password |       |
| authentication_string  | text                              | YES  |     | NULL                  |       |
| password_expired       | enum('N','Y')                     | NO   |     | N                     |       |
| password_last_changed  | timestamp                         | YES  |     | NULL                  |       |
| password_lifetime      | smallint(5) unsigned              | YES  |     | NULL                  |       |
| account_locked         | enum('N','Y')                     | NO   |     | N                     |       |
+------------------------+-----------------------------------+------+-----+-----------------------+-------+
45 rows in set (0.01 sec)

//5.6之前是 password 5.7开始是 authentication_string

(root@localhost)[mysql]> select Host,User,authentication_string from user limit 10;
+-----------+-----------+-------------------------------------------+
| Host      | User      | authentication_string                     |
+-----------+-----------+-------------------------------------------+
| localhost | root      |                                           |
| localhost | mysql.sys | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| %         | test1     | *23AE809DDACAF96AF0FD78ED04B6A265E05AA257 |
| %         | test2     | *531E182E2F72080AB0740FE2F2D689DBE0146E04 |
+-----------+-----------+-------------------------------------------+
//密码不可逆
(root@localhost)[mysql]> select password('456');
+-------------------------------------------+
| password('456')                           |
+-------------------------------------------+
| *531E182E2F72080AB0740FE2F2D689DBE0146E04 |
+-------------------------------------------+
1 row in set, 1 warning (0.01 sec)

那既然有了表，我们是不是可以通过修改表数据来达到修改权限的目的呢？？
(root@localhost)[mysql]> update db set Index_priv = 'Y' where User = 'test2';
Query OK, 0 rows affected (0.00 sec)
Rows matched: 1  Changed: 0  Warnings: 0

先修改 在刷新权限。
(root@localhost)[mysql]> flush privileges;

(root@localhost)[mysql]> show grants for test2;
+------------------------------------------------------------------------------------------+
| Grants for test2@%                                                                       |
+------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'test2'@'%'                                                        |
| GRANT SELECT, INSERT, UPDATE, DELETE, INDEX ON `test`.* TO 'test2'@'%' WITH GRANT OPTION |
+------------------------------------------------------------------------------------------+


答案是可以的，但是 不建议！！！！尽量不要直接去操作mysql下的表,使用命令来操作达到目的。

其他几个表 都可以这么查看数据。来查看用户权限的元数据表。

```

## MySQL资源管理

```shell

GRANT
    priv_type [(column_list)]
      [, priv_type [(column_list)]] ...
    ON [object_type] priv_level
    TO user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH {GRANT OPTION | resource_option} ...]

GRANT PROXY ON user
    TO user [, user] ...
    [WITH GRANT OPTION]

object_type: {
    TABLE
  | FUNCTION
  | PROCEDURE
}

priv_level: {
    *
  | *.*
  | db_name.*
  | db_name.tbl_name
  | tbl_name
  | db_name.routine_name
}

user:
    (see Section 6.2.3, “Specifying Account Names”)

auth_option: {
    IDENTIFIED BY 'auth_string'
  | IDENTIFIED WITH auth_plugin
  | IDENTIFIED WITH auth_plugin BY 'auth_string'
  | IDENTIFIED WITH auth_plugin AS 'hash_string'
  | IDENTIFIED BY PASSWORD 'hash_string'
}

tls_option: {
    SSL
  | X509
  | CIPHER 'cipher'
  | ISSUER 'issuer'
  | SUBJECT 'subject'
}

//资源管理
resource_option: {
  | MAX_QUERIES_PER_HOUR count //每小时最大查询次数
  | MAX_UPDATES_PER_HOUR count //每小时最大更新次数
  | MAX_CONNECTIONS_PER_HOUR count //每个小时最大连接次数
  | MAX_USER_CONNECTIONS count //同时连接的个数
}
```

```
(root@localhost)[mysql]> alter user test1 with max_connections_per_hour 1;
Query OK, 0 rows affected (0.00 sec)

ShaoGaoJie@MacBook-Air-2 ~> mysql -utest1 -p
Enter password:
ERROR 1226 (42000): User 'test1' has exceeded the 'max_connections_per_hour' resource (current value: 1)

场景在哪儿？？？粒度是不是有点粗？？？

```

```
(root@localhost)[mysql]> show grants for current_user();
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
| GRANT PROXY ON ''@'' TO 'root'@'localhost' WITH GRANT OPTION        |
+---------------------------------------------------------------------+ 
```

