## MySQL安装和基本命令以及升级
### 安装
参考：

https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html
https://dev.mysql.com/doc/refman/5.6/en/binary-installation.html

### 开机启动配置
```
#chkconfig --add mysql.server
#cchkconfig —list  mysqld

```
### 修改密码，使得直接输入mysql命令就可以登录mysql服务，省去输入密码操作
```
//找到my.cnf文件位置 如果不知道怎么找 可以通过 --help来查找
# /Applications/XAMPP/xamppfiles/bin/mysql —help
/Applications/XAMPP/xamppfiles/etc/xampp/my.cnf /Applications/XAMPP/xamppfiles/etc/my.cnf ~/.my.cnf

# vim /Applications/XAMPP/xamppfiles/etc/my.cnf
 [client]
 user=root
 password=123123
```

### 在my.cnf里添加如下代码 可以达到下面的效果：
```
 [mysql]
 prompt=(\\u@\\h)[\\d]>\\_
 
 //效果如下：
  (root@localhost)[test]>
```

### 修改数据库用户密码的方法
```
//mysql 5.7
# (root@localhost)[test]>set password = '123';
//mysql 5.6
# (root@localhost)[test]>set password = password('123');
```

### 查看当前版本
```
(root@localhost)[test]> select version();
+------------+
| version()  |
+------------+
| 5.6.12-log |
+------------+
1 row in set (0.01 sec)
```

### 查看mysql当前的配置信息
```
mysql --help --verbose

mysqladmin variables -u root -p


```


### 版本升级
如何升级当前的mysql版本呢？
导出导入？代价太高

原地升级：

```
0. 建议备份当前完整的数据目录
1. 停服务 /etc/init.d/mysql.server stop
2. 删软连 unlink mysql
3. 将mysql命令指向新版本 ln -s 新版本 mysql
4. 启动mysql  /etc/init.d/mysql.server start
5. mysql 进入 查看版本号
6. 查看error.log 看看是否有Error类型的错误信息
7. mysql_upgrade -s 升级系统表 但是不要更新数据表
8. 

```

跨版本升级(5.1-5.7.x)：

https://mysqlserverteam.com/upgrading-directly-from-mysql-5-0-to-5-7-using-an-in-place-upgrade/

跨版本降级（只能在小版本使用）

