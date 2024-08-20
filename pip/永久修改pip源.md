永久修改pip源8062b5ff5013c5a1d54bae1b1960144c

```bash
 pip config set global.index-url https://mirrors.aliyun.com/pypi/simple
 pip config set install.trusted-host mirrors.aliyun.com
```

或者

修改  ~/.config/pip/pip.conf

```ini
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
 
[install]
trusted-host=mirrors.aliyun.com
```

```
清华：https://pypi.tuna.tsinghua.edu.cn/simple
阿里：https://mirrors.aliyun.com/pypi/simple
```

