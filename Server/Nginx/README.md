## Nginx

### nginx安装
  - 修改yum源
  ```
  >cat /etc/yum.repos.d/nginx.repo
  [nginx]
  name=nginx repo
  baseurl=http://nginx.org/packages/centos/7/$basearch/
  gpgcheck=0
  enable=1
  ```
  - yum install nginx
  - nginx 启动  
  - nginx -s reload 重启
  - nginx -s stop 停止
  - nginx -V 查看编译参数
  - 注意:配置反向代理要关闭seliunx,setenforc 0
  - http://nginx.org/en/linux_packages.html
  
  
### ngx_http_stub_status监控连接信息
  - 添加配置
  ```
  location= /nginx_status{
      stub_status on;
      access_log off;
      allow 127.0.0.1;
      deny all;
  }
  ```
  - http://nginx.org/en/docs/http/ngx_http_stub_status_module.html

### ngxtop 监控请求信息
  - 安装
    - 安装python-pip
    ```
    >yum install epel-release
    >yum install python-pip 
    ```
    - 安装ngxtop
    ```
    >pip install ngxtop
    ```
    
  - 指定配置文件: ngxtop -c /etc/nginx/nginx.conf
  - 查询状态是200: ngxtop -c /etc/nginx/nginx.conf -i 'status== 200'
  - 查询访问最多ip ngxtop -c /etc/nginx/nginx.conf -g remote_addr
  - 测试配置文件是否正确: nginx -t
  - 官方地址: https://github.com/lebinh/ngxtop

### nginx-rrd 图形化监控

  - 官方地址: http://www.linuxde.net/2012/04/9537.html


### Nginx优化
  - 增加工作线程数和并发连接数   
  ---
     work_processes 4; #cpu
     events{
        worker_connections 10240; #每个进程打开的最大连接数，包含了nginx与客户端和nginx与upstream之间的连接
        multi_accept on; #可以一次建立多个连接
        use epoll;
     }
  - 启用长连接
  ---
     upstream server_pool{
       server localhost:8080 weight=1 max_fails=2 fail_timeout=30s;
       server_localhost:8081 weight=1 max_fails=2 fail_timeout=30s;
       keepalive 300; #300个长连接
     }
     
     location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_uprade;
        proxy_set_header Connection "upgrade";
        proxy_pass http://server_pool/;
     }
  
  - 启用缓存、压缩
  ---
     gzip on;
     gzip_http_version 1.1;
     gzip_disable "MSIE [1-6]\.(?!.*SV1)";
     gzip_proxied any;
     gzip_types text/plain text/css application/javascript application/x-javascript application/json application/xml application/vnd.ms-fontobject application/x-font-ttf application/svf+xml application/x-icon;
     gzip_vary on; #Vary:Accept-Encoding
     gzip_static on;#如果有压缩好的 直接使用
  - 操作系统优化
  ---
     #配置/etc/sysctl.conf
     sysctl -w net.ipv4.tcp_syncookies=1   #防止一个套接字在有过多试图连接到达时引起过载
     sysctl -w net.core.somaxconn=1024  #默认128,连接队列
     sysctl -w net.ipv4.tcp_fin_timeout=10       #timewait的超时时间
     sysctl -w net.ipv4.tcp_tw_reuse=1           #os直接使用timewait的连接
     sysctl -w net.ipv4.tcp_tw_recycle=0         #回收禁用
  - 其化优化
     sendfile  on;  #减少文件在应用和内核之间的拷贝
     tcp_nopush on;   #当数据包达到一定大小再发送
     tcp_nodelay off;  #有数据随时发送


### 官方文档 
  http://nginx.org/en/docs/