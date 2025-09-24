# ps-web
在线ps网页版，添加本地字体库

```
services:
  photopea:
    image: dd995/ps-web:v1.1   
    container_name: ps-web
    restart: unless-stopped
    ports:
      - "8887:8887"            
```
```
docker run -d \
  --name ps-web \
  --restart unless-stopped \
  -p 127.0.0.1:8887:8887 \
  dd995/ps-web:v1.1


```


## 端口号：8887

## 方案一：
### 字体包
rsrc压缩包需要在站点**根目录**下解压

### 配置项
```
# 优先从本地文件读取字体
location ^~ /ps/rsrc/fonts/ {
    root /www/wwwroot;
    access_log off;

    # 字体通常需要 CORS 跨域支持
    add_header Access-Control-Allow-Origin "*" always;
    add_header Access-Control-Allow-Methods "GET, OPTIONS" always;
    add_header Access-Control-Allow-Headers "Range,Accept,Origin,Authorization,Cache-Control" always;

    # OPTIONS 预检请求直接返回
    if ($request_method = OPTIONS) {
        add_header Access-Control-Max-Age 1728000;
        add_header Content-Type "text/plain charset=UTF-8";
        add_header Content-Length 0;
        return 204;
    }

    # 尝试从本地取文件，如果没有，就回退到容器
    try_files $uri $uri/ @ps_font_proxy;
}

# 回退代理（如果本地没字体，就去容器拿）
location @ps_font_proxy {
    # 去掉 /ps 前缀再转发
    rewrite ^/ps/(.*)$ /$1 break;
    proxy_pass http://127.0.0.1:8887;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
}

# 其它 Photopea 请求还是走容器
location /ps/ {
    rewrite ^/ps/(.*)$ /$1 break;
    proxy_pass http://127.0.0.1:8887;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
}

```

## 方案二：
### 配置项
```
# 字体代理
location ^~ /ps/rsrc/fonts/ {
    proxy_pass https://www.photopea.com/rsrc/fonts/;
    proxy_set_header Host www.photopea.com;
    proxy_ssl_server_name on;

    # 字体通常需要跨域头
    add_header Access-Control-Allow-Origin "*" always;
    add_header Access-Control-Allow-Methods "GET, OPTIONS" always;
    add_header Access-Control-Allow-Headers "Range,Accept,Origin,Authorization,Cache-Control" always;

    # OPTIONS 预检请求直接返回
    if ($request_method = OPTIONS) {
        add_header Access-Control-Max-Age 1728000;
        add_header Content-Type "text/plain charset=UTF-8";
        add_header Content-Length 0;
        return 204;
    }
}
```
