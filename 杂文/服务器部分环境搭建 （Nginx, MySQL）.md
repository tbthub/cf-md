# 服务器部分环境搭建 （Nginx, MySQL）

当前系统：**ubuntu22.04**

## Nginx

![nginx-logo](https://www.w3schools.cn/wp-content/uploads/nginx/nginx-logo.png)[^1] 



### 安装

1. **更新包索引**
```bash
sudo apt update
```

2. **安装 Nginx**
```bash
sudo apt install nginx
```

3. **设置启动**
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

4. **检查 Nginx 服务状态**
```bash
sudo systemctl status nginx
```
如果一切正常，Nginx 服务应该会显示为“active (running)”

5. **配置防火墙**
如果服务器启用了防火墙（如 UFW），则需要允许 HTTP 和 HTTPS 流量通过。使用以下命令开启这些服务：
```bash
sudo ufw allow 'Nginx HTTP'
sudo ufw allow 'Nginx HTTPS'
```

6.  **配置 Nginx**

- 默认配置文件路径：/etc/nginx/nginx.conf
- 站点配置文件路径：/etc/nginx/sites-available/ 和 /etc/nginx/sites-enabled/

可以在 /etc/nginx/sites-available/目录下配置站点，并在/etc/nginx/sites-enabled/中创建符号链接。

7. **验证 Nginx 是否正常工作**
访问服务器的 IP 地址或域名，确认 Nginx 默认的欢迎页面是否显示。
使用 nginx -t 命令测试 Nginx 配置文件的语法是否正确。





### 删除

1. **停止Nginx服务**

```bash
sudo systemctl stop nginx
```

2. **卸载 Nginx 及其相关包**

```bash
sudo apt nginx nginx-common nginx-core
```

3. **删除不再需要的依赖包**

```bash
sudo apt-get autoremove
sudo apt-get autoclean
```





### 静态站点部署

1. 创建存放静态网站文件的目录。以`/var/www/mywebsite`为例：
```bash
sudo mkdir -p /var/www/mywebsite
```

2. 将静态文件复制
```bash
sudo cp /path/to/your/website/* /var/www/mywebsite/
```

3. 编辑Nginx配置
```bash
sudo vim /etc/nginx/sites-available/mywebsite
```

在server块中添加以下配置，指向刚刚创建的静态网站目录
```nginx
server {
   
   
    listen 80;
    server_name mywebsite.com www.mywebsite.com;

    location / {
        root /var/www/mywebsite;
        index index.html;
    }
}
```
4. 保存配置文件后，创建配置文件的符号链接并重新加载Nginx

```bash
sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```





### 其他

#### 使文件可网络下载

```bash
sudo vim /etc/nginx/sites-available/default
```

加上：

```nginx
# /+随便一个名字，这里以"download"为例
location /download {
		#允许访问的文件夹
        alias /var/www/html/doc/;
        autoindex on;
}
```

浏览器输入“http://你的ip/download”即可查看,结果类似：

```http
Index of /download/
../
abc.md                  02-Aug-2024 16:12                8170
def.md					02-Aug-2024 16:18 				 1870
```



#### 根据后缀转发端口

```ini
location /code/ {
       proxy_pass http://localhost:20000/;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;        
       proxy_set_header Connection $http_connection;
}
```



#### 限制特定IP访问

在 Nginx 中配置 `allow` 和 `deny` 指令来控制访问：

```nginx
server {
    listen 5593;
    server_name example.com;

    location / {
        proxy_pass http://localhost:5593;
        allow 192.168.1.0/24;
        deny all;
    }
}
```





---







## MySQL

<img src="https://www.mysql.com/common/logos/logo-mysql-170x115.png#pic_center" style="zoom:150%;" />[^2]





### 安装

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

5. **打开MySQL控制台设置root用户密码**
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

6. 重新登录

```bash
mysql -u root -p
```

7. 开机自启

```bash
sudo systemctl enable mysql
```





### 删除

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





### 其他

#### 允许远程访问

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






>参考：
>
>[如何在 Ubuntu 22.04 上安装、配置、使用 Nginx？-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1443902)
>
>[在 Ubuntu 22.04 上安装和配置 Nginx 的完整指南](https://blog.csdn.net/u011715638/article/details/138670319)
>[在Ubuntu上安装和配置MySQL保姆级教程](https://www.51cto.com/article/718700.html)
>
>[在Ubuntu 22.04 LTS 上安装 MySQL两种方式：在线方式和离线方式](https://blog.csdn.net/weixin_45626288/article/details/133220238)
>
>图像来源 :
>
>[^1]: 图像来源: [w3schools](https://www.w3schools.cn)
>
>[^2]: 图片来源于：[MySQL Logo](https://www.mysql.com/about/legal/logos.html)
