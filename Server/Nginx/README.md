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
  - https://github.com/lebinh/ngxtop

### nginx-rrd 图形化监控




### Nginx优化



### 官方文档 
http://nginx.org/en/docs/