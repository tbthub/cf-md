# 只允许内网IP访问网络请求（python Flask为例）



## 1. 配置 Flask 只绑定内网 IP

修改 Flask 应用的运行参数，使其只绑定到内网 IP 地址。可以在 Flask 的启动命令中指定 `host` 参数为内网 IP 地址。假设内网 IP 地址是 `192.168.1.100`，可以如下配置：

```bash
flask run --host=192.168.1.100
```

如果使用 Gunicorn 作为 WSGI 服务器，启动时指定绑定到内网 IP 地址：

```bash
gunicorn --bind 192.168.1.100:5593 main:app
```

## 2. **使用防火墙配置**

配置防火墙规则以允许仅内网 IP 地址访问 Flask 应用。使用 `iptables` 或 `ufw` 可以限制对特定端口的访问。

### 使用 `ufw`

1. **允许内网 IP 地址访问特定端口（假设端口是 5593）**

   ```bash
   sudo ufw allow from 192.168.1.0/24 to any port 5593
   ```

2. **拒绝外网对该端口的访问**

   ```bash
   sudo ufw deny 5593
   ```

   这样配置后，只有内网中的 IP 地址才能访问 Flask 应用，外网则被拒绝访问。

### 使用 `iptables`

1. **允许内网 IP 地址访问特定端口**

   ```bash
   sudo iptables -A INPUT -p tcp -s 192.168.1.0/24 --dport 5593 -j ACCEPT
   ```

2. **拒绝外网对该端口的访问**

   ```bash
   sudo iptables -A INPUT -p tcp --dport 5593 -j DROP
   ```

## 3. **通过 Flask 中间件或装饰器限制访问**

在 Flask 应用中使用装饰器或中间件来限制访问，检查请求的来源 IP 地址是否在允许的范围内。

### 示例代码：

```python
from flask import Flask, request, abort

app = Flask(__name__)

#允许访问的内网 IP 地址范围
ALLOWED_IPS = ["192.168.1.0/24"]

def ip_in_allowed_range(ip, ranges):
    from ipaddress import ip_address, ip_network
    ip = ip_address(ip)
    return any(ip in ip_network(r) for r in ranges)

@app.before_request
def limit_remote_addr():
    if not ip_in_allowed_range(request.remote_addr, ALLOWED_IPS):
        abort(403)  # Forbidden

@app.route('/')
def home():
    return "Welcome to the internal network!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5593)
```

在这个例子中，`ip_in_allowed_range` 函数检查请求的 IP 地址是否在允许的范围内。如果不在范围内，则返回 403 错误。

## 4. **使用反向代理**

如果的 Flask 应用在一个反向代理（如 Nginx）后面，可以配置反向代理来限制访问。例如，在 Nginx 中配置 `allow` 和 `deny` 指令来控制访问：

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

