# Ubuntu服务器安装MySQL



<img src="https://www.mysql.com/common/logos/logo-mysql-170x115.png#pic_center" style="zoom:150%;" />[^1]





## 安装MySQL服务器

1. **更新包索引**
```bash
sudo apt update
```

2. **安装MySQL服务器**
```bash
sudo apt-get install mysql-server
```

3. **检查服务状态**
```bash
sudo systemctl status mysql
```

4. **运行 MySQL 安全配置脚本**
```bash
sudo mysql_secure_installation
```

```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No: 

Skipping password set for root as authentication with auth_socket is used by default.
If you would like to use password authentication instead, this can be done with the "ALTER_USER" command.
See https://dev.mysql.com/doc/refman/8.0/en/alter-user.html#alter-user-password-management for more information.

By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) :y 
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done!
(貌似没有看到root密码相关的)
```
4. **打开MySQL控制台设置root用户密码**
```bash
sudo mysql
```

查看不同用户使用的身份验证方法：

```sql
SELECT user,authentication_string,plugin,host FROM mysql.user;
```

可以看到root使用了auth_socket进行身份验证，下面使用ALTER更改密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'your_password';
```

刷新一下授权表

```sql
FLUSH PRIVILEGES;
```

确认一下

```sql
SELECT user,authentication_string,plugin,host FROM mysql.user;
```

重新登录

```bash
mysql -u root -p
```

开机自启

```bash
sudo systemctl enable mysql
```



---

## 删除MySQL服务器及其相关包

1. **停止MySQL服务**

```bash
sudo systemctl stop mysql
```

2. **卸载MySQL服务器及其相关包**

```bash
sudo apt purge mysql-server
sudo apt autoremove
sudo apt-get autoclean
```



---



## 其他

允许远程访问

更改绑定bind-address 127.0.0.1->0.0.0.0，公网可访问

```bash
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
sudo service mysql restart
```

把需要远程连接的用户localhost改成%

```sql
use mysql;
#这里以root为例
update user set host='%' where user='root';
flush privileges;
```



## 参考

> [^1]:图片来源于：[MySQL Logo](https://www.mysql.com/about/legal/logos.html)
>
> [在Ubuntu上安装和配置MySQL保姆级教程](https://www.51cto.com/article/718700.html)
> 
> [在Ubuntu 22.04 LTS 上安装 MySQL两种方式：在线方式和离线方式](https://blog.csdn.net/weixin_45626288/article/details/133220238)
