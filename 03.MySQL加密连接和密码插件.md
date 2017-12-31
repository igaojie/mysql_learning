#MySQL加密连接和密码插件
## 连接MySQL示例
### 连接方式
1. socket方式连接

```
//-S socket 默认
ShaoGaoJie@MacBook-Air-2 ~> mysql -utest2 -p -S /tmp/mysql.sock


(root@localhost)[mysql]> show variables like 'socket';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| socket        | /tmp/mysql.sock |
+---------------+-----------------+
1 row in set (0.00 sec)



```

2. 通过tcp/ip方式连接
 
```
 mysql -h 127.0.0.1 -uroot -p
```

 如何查看当前用户是通过什么方式进行连接的呢？
 
 ```
 (test2@localhost)[(none)]> \s   或者 status 命令
--------------
mysql  Ver 14.14 Distrib 5.7.20, for osx10.13 (x86_64) using  EditLine wrapper

Connection id:		23
Current database:
Current user:		test2@localhost
SSL:			Not in use
Current pager:		less
Using outfile:		''
Using delimiter:	;
Server version:		5.7.20 Homebrew
Protocol version:	10
//从这里来观察连接方式
Connection:		Localhost via UNIX socket  标志着是socket连接方式
Connection:		127.0.0.1 via TCP/IP   标志着是tcp/ip连接方式
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
UNIX socket:		/tmp/mysql.sock        
Uptime:			3 hours 30 min 7 sec

Threads: 7  Questions: 992  Slow queries: 0  Opens: 360  Flush tables: 1  Open tables: 353  Queries per second avg: 0.078
 ```

#### homebrew 安装mysql5.7 会自动启用 SSL方式连接
```
(root@127.0.0.1)[(none)]> show variables like '%ssl%';
+--------------------+-----------------+
| Variable_name      | Value           |
+--------------------+-----------------+
| have_openssl       | YES             |
| have_ssl           | YES             |
| mysqlx_ssl_ca      |                 |
| mysqlx_ssl_capath  |                 |
| mysqlx_ssl_cert    |                 |
| mysqlx_ssl_cipher  |                 |
| mysqlx_ssl_crl     |                 |
| mysqlx_ssl_crlpath |                 |
| mysqlx_ssl_key     |                 |
| ssl_ca             | ca.pem          |
| ssl_capath         |                 |
| ssl_cert           | server-cert.pem |
| ssl_cipher         |                 |
| ssl_crl            |                 |
| ssl_crlpath        |                 |
| ssl_key            | server-key.pem  |
+--------------------+-----------------+
16 rows in set (0.00 sec)

(root@127.0.0.1)[(none)]> \s
--------------
mysql  Ver 14.14 Distrib 5.7.20, for osx10.13 (x86_64) using  EditLine wrapper

Connection id:		4
Current database:
Current user:		root@localhost
SSL:			Cipher in use is DHE-RSA-AES128-GCM-SHA256
Current pager:		less
Using outfile:		''
Using delimiter:	;
Server version:		5.7.20 Homebrew
Protocol version:	10
Connection:		127.0.0.1 via TCP/IP
Server characterset:	utf8
Db     characterset:	utf8
Client characterset:	utf8
Conn.  characterset:	utf8
TCP port:		3307
Uptime:			6 min 12 sec

Threads: 1  Questions: 7  Slow queries: 0  Opens: 105  Flush tables: 1  Open tables: 98  Queries per second avg: 0.018
--------------

```

### 如果安装的时候没有执行ssl的相关操作，之后怎么让ssl有效呢？
```
//注意：5.6版本的启用ssl 的文档：https://dev.mysql.com/doc/refman/5.6/en/creating-ssl-files-using-openssl.html
//注意：安装5.7的过程中有这么一步骤，就是启用ssl的。如果安装时没有执行这么一步骤，接下来就需要另外操作了。

shell> bin/mysql_ssl_rsa_setup

//执行完毕 会在mysql 的 datadir里 生成以下文件：
@MacBook-Air-2 /u/l/v/mysql> ll *.pem
-rw-------  1 ShaoGaoJie  admin   1.6K  2 26  2017 ca-key.pem
-rw-r--r--  1 ShaoGaoJie  admin   1.0K  2 26  2017 ca.pem
-rw-r--r--  1 ShaoGaoJie  admin   1.1K  2 26  2017 client-cert.pem
-rw-------  1 ShaoGaoJie  admin   1.6K  2 26  2017 client-key.pem
-rw-------  1 ShaoGaoJie  admin   1.6K  2 26  2017 private_key.pem
-rw-r--r--  1 ShaoGaoJie  admin   452B  2 26  2017 public_key.pem
-rw-r--r--  1 ShaoGaoJie  admin   1.1K  2 26  2017 server-cert.pem
-rw-------  1 ShaoGaoJie  admin   1.6K  2 26  2017 server-key.pem              
接下来修改权限 并重启mysql服务 即可。
```

如果远程连接的时候，不想启用ssl，怎么操作？
```
mysql -h 127.0.0.1 -uroot -p -P3307 --ssl-mode=DISABLED
```

生产环境使用ssl模式会稍微影响一些性能，但是影响不大。毕竟数据库和业务都是通过内网连接的，不打开ssl模式也是可以的。 

如果强制一个用户必须使用ssl模式连接 怎么办？

```shell
CREATE USER [IF NOT EXISTS]
    user [auth_option] [, user [auth_option]] ...
    [REQUIRE {NONE | tls_option [[AND] tls_option] ...}]
    [WITH resource_option [resource_option] ...]
    [password_option | lock_option] ...

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
 | X509    ------------不仅仅需要 用户名+密码+ip+ssl，还需要提供client.pem等文件
 | CIPHER 'cipher'
 | ISSUER 'issuer'
 | SUBJECT 'subject'
}

resource_option: {
    MAX_QUERIES_PER_HOUR count
  | MAX_UPDATES_PER_HOUR count
  | MAX_CONNECTIONS_PER_HOUR count
  | MAX_USER_CONNECTIONS count
}

password_option: {
    PASSWORD EXPIRE
  | PASSWORD EXPIRE DEFAULT
  | PASSWORD EXPIRE NEVER
  | PASSWORD EXPIRE INTERVAL N DAY
}

lock_option: {
    ACCOUNT LOCK
  | ACCOUNT UNLOCK
}
```

```
(root@127.0.0.1)[(none)]> create user 'test2_ssl'@'%' identified by '123' require ssl;
Query OK, 0 rows affected (0.00 sec)

//如果require ssl 那么 连接的时候如果ssl_mode=disabled 那么就连接不上了
@MacBook-Air-2 ~> mysql -h 127.0.0.1 -utest2_ssl -p -P3307 --ssl_mode=DISABLED
Enter password:
ERROR 1045 (28000): Access denied for user 'test2_ssl'@'localhost' (using password: YES)

```


## 密码插件

文档：https://dev.mysql.com/doc/refman/5.6/en/validate-password-plugin-installation.html

文档：https://dev.mysql.com/doc/refman/5.7/en/validate-password-plugin-installation.html

```
//查看当前已经安装过的插件
(root@127.0.0.1)[(none)]> show plugins;
+----------------------------+----------+--------------------+-----------+---------+
| Name                       | Status   | Type               | Library   | License |
+----------------------------+----------+--------------------+-----------+---------+
| binlog                     | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| mysql_native_password      | ACTIVE   | AUTHENTICATION     | NULL      | GPL     |
| sha256_password            | ACTIVE   | AUTHENTICATION     | NULL      | GPL     |
| CSV                        | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| MEMORY                     | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| InnoDB                     | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| INNODB_TRX                 | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_LOCKS               | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_LOCK_WAITS          | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_CMP                 | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_CMP_RESET           | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_CMPMEM              | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_CMPMEM_RESET        | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_CMP_PER_INDEX       | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_CMP_PER_INDEX_RESET | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_BUFFER_PAGE         | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_BUFFER_PAGE_LRU     | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_BUFFER_POOL_STATS   | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_TEMP_TABLE_INFO     | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_METRICS             | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_FT_DEFAULT_STOPWORD | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_FT_DELETED          | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_FT_BEING_DELETED    | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_FT_CONFIG           | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_FT_INDEX_CACHE      | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_FT_INDEX_TABLE      | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_TABLES          | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_TABLESTATS      | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_INDEXES         | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_COLUMNS         | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_FIELDS          | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_FOREIGN         | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_FOREIGN_COLS    | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_TABLESPACES     | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_DATAFILES       | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| INNODB_SYS_VIRTUAL         | ACTIVE   | INFORMATION SCHEMA | NULL      | GPL     |
| MyISAM                     | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| MRG_MYISAM                 | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| PERFORMANCE_SCHEMA         | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| ARCHIVE                    | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| BLACKHOLE                  | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| FEDERATED                  | DISABLED | STORAGE ENGINE     | NULL      | GPL     |
| partition                  | ACTIVE   | STORAGE ENGINE     | NULL      | GPL     |
| ngram                      | ACTIVE   | FTPARSER           | NULL      | GPL     |
| mysqlx                     | ACTIVE   | DAEMON             | mysqlx.so | GPL     |
+----------------------------+----------+--------------------+-----------+---------+
45 rows in set (0.03 sec)

```

### 安装validate_password插件
```
(root@127.0.0.1)[(none)]> INSTALL PLUGIN validate_password SONAME 'validate_password.so';

(root@127.0.0.1)[(none)]> SELECT PLUGIN_NAME, PLUGIN_STATUS
    ->        FROM INFORMATION_SCHEMA.PLUGINS
    ->        WHERE PLUGIN_NAME LIKE 'validate%';
+-------------------+---------------+
| PLUGIN_NAME       | PLUGIN_STATUS |
+-------------------+---------------+
| validate_password | ACTIVE        |
+-------------------+---------------+

//如果密码过于简单 就将无法设置密码
(root@127.0.0.1)[(none)]> alter user test1 identified by '123';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

//查看当前的密码安全策略
(root@127.0.0.1)[(none)]> show variables like 'validate%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password_check_user_name    | OFF    |-------- 密码中不能出现用户名中的字符串
| validate_password_dictionary_file    |        |-------- 字典文件 
| validate_password_length             | 8      |--------长度8位
| validate_password_mixed_case_count   | 1      |--------大小写混合
| validate_password_number_count       | 1      |--------数字至少出现一次
| validate_password_policy             | MEDIUM |  -----中级策略
| validate_password_special_char_count | 1      |--------特殊字符最少出现一次 
+--------------------------------------+--------+
7 rows in set (0.00 sec)

```

#### 使用validate password policy
| Policy |Tests Performed|
|:--|:--|
|0 or LOW	|0 or LOW	|
|1 or MEDIUM|Length; numeric, lowercase/uppercase, and special characters|
|2 or STRONG	|Length; numeric, lowercase/uppercase, and special characters; dictionary file|

dictionary file：出现在这个字典文件里的字符串都不能作为密码使用。

#### dictionary file 字典文件的使用

```
//设置密码字典文件
➜  #/tmp cat /tmp/dict.file
admin
test
root
abc

//设置文件字典 validate_password_dictionary_file
(root@127.0.0.1)[(none)]> set global validate_password_dictionary_file = '/tmp/dict.file';

//设置密码安全策略为STRONG 或者 2
(root@127.0.0.1)[(none)]> set global validate_password_policy = STRONG;

//这样仍然是设置不了密码的。因为出现了字典文件里的字符串
(root@127.0.0.1)[(none)]> alter user test1 identified by '13adminT--';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

//这个就好了。 对吧。。。
(root@127.0.0.1)[(none)]> alter user test1 identified by '13a1d3minT--';
Query OK, 0 rows affected (0.01 sec)

```
