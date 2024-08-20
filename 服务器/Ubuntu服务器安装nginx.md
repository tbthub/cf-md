 # Ubuntu服务器安装nginx

![nginx-logo](https://www.w3schools.cn/wp-content/uploads/nginx/nginx-logo.png)[^1] 

当前系统：ubuntu22.04


## 安装Nginx
1. **更新包索引**
```bash
sudo apt update
```

2. **安装 Nginx**
```bash
sudo apt install nginx
```

3. **安装 Nginx**
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

## 删除nginx

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

---



## 静态站点部署

1. 创建存放静态网站文件的目录。以`/var/www/mywebsite`为例：
```bash
sudo mkdir -p /var/www/mywebsite
```

2. 将静态文件复制
```bash
sudo cp /path/to/your/website/* /var/www/mywebsite/
```

3. 打开Nginx配置
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
保存配置文件后，创建配置文件的符号链接并重新加载Nginx
```bash
sudo ln -s /etc/nginx/sites-available/mywebsite /etc/nginx/sites-enabled/
sudo systemctl reload nginx
```

## 远程访问文件

编辑配置文件

```bash
sudo vim /etc/nginx/sites-available/default
```

加上：

```nginx
# /+随便一个名字，这里以"download"为例
location /download {
		#允许访问的文件夹
        alias /var/www/html/blogs/md/;
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

## 致谢
>[^1]: 图像来源: [w3schools](https://www.w3schools.cn)
>
> [如何在 Ubuntu 22.04 上安装、配置、使用 Nginx？-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/article/1443902)
>
> [在 Ubuntu 22.04 上安装和配置 Nginx 的完整指南](https://blog.csdn.net/u011715638/article/details/138670319)



