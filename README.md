# Let-s-Encrypt
Ubuntu 14.04 上为 Nginx 添加 Let's Encrypt 框架rails 服务puma


一 、 首先安装nginx（略)

二 、在来安装let's encrypt,在安装之前确定80和443端口没有被占用，最好是nginx不启动

  1、首先下载 git clone https://github.com/letsencrypt/letsencrypt/opt/letsencrypt
  2、到letsencrypt目录下执行 
  ```ruby
    ./letsencrypt-auto certonly -d example.com -d www.example.com
  ```
  这是会生成/etc/letsencrypt/live/www.example.com 的目录，里面有四个文件。下面会使用 fullchain.pem 来配置你的服务器作为证书，
  privkey.pem 文件作为证书的 key 文件。
  3、配置dhparams长度
  ``` ruby
  cd /opt
  mkdir dhparam
  cd dhparam
  mkdir keys
  cd keys
  openssl dhparam -out dhparams.pem 2048
  cd ../
  sudo chmod 700 keys

  ```
  4、 生成ssl_ciphers
  ``` ruby
    openssl ciphers # 输出的内容copy一下，下面配置nginx的时候需要用到
  ```
三、配置nginx
 
 ``` ruby
 
 upstream forum {
  server unix:///yourpath/tmp/sockets/puma.sock fail_timeout=0;
}
server {
  listen 443  ssl;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  # ssl_ciphers 是用openssl ciphers 生成的后面参数
  ssl_ciphers 'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:SRP-DSS-AES-256-CBC-SHA:SRP-RSA-AES-256-CBC-SHA:SRP-AES-256-CBC-SHA:DHE-DSS-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA256:DHE-RSA-AES256-SHA:DHE-DSS-AES256-SHA:DHE-RSA-CAMELLIA256-SHA:DHE-DSS-CAMELLIA256-SHA:ECDH-RSA-AES256-GCM-SHA384:ECDH-ECDSA-AES256-GCM-SHA384:ECDH-RSA-AES256-SHA384:ECDH-ECDSA-AES256-SHA384:ECDH-RSA-AES256-SHA:ECDH-ECDSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:CAMELLIA256-SHA:PSK-AES256-CBC-SHA:ECDHE-RSA-DES-CBC3-SHA:ECDHE-ECDSA-DES-CBC3-SHA:SRP-DSS-3DES-EDE-CBC-SHA:SRP-RSA-3DES-EDE-CBC-SHA:SRP-3DES-EDE-CBC-SHA:EDH-RSA-DES-CBC3-SHA:EDH-DSS-DES-CBC3-SHA:ECDH-RSA-DES-CBC3-SHA:ECDH-ECDSA-DES-CBC3-SHA:DES-CBC3-SHA:PSK-3DES-EDE-CBC-SHA:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:SRP-DSS-AES-128-CBC-SHA:SRP-RSA-AES-128-CBC-SHA:SRP-AES-128-CBC-SHA:DHE-DSS-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-SHA256:DHE-DSS-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA:DHE-RSA-SEED-SHA:DHE-DSS-SEED-SHA:DHE-RSA-CAMELLIA128-SHA:DHE-DSS-CAMELLIA128-SHA:ECDH-RSA-AES128-GCM-SHA256:ECDH-ECDSA-AES128-GCM-SHA256:ECDH-RSA-AES128-SHA256:ECDH-ECDSA-AES128-SHA256:ECDH-RSA-AES128-SHA:ECDH-ECDSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:SEED-SHA:CAMELLIA128-SHA:PSK-AES128-CBC-SHA:ECDHE-RSA-RC4-SHA:ECDHE-ECDSA-RC4-SHA:ECDH-RSA-RC4-SHA:ECDH-ECDSA-RC4-SHA:RC4-SHA:RC4-MD5:PSK-RC4-SHA:EDH-RSA-DES-CBC-SHA:EDH-DSS-DES-CBC-SHA:DES-CBC-SHA';
  ssl_dhparam /opt/dhparam/keys/dhparams.pem;
  server_name www.example.com; # change to match your URL
  root /home/yourprojectpath/current/public;
  ssl_certificate /etc/letsencrypt/live/www.example.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/www.example.com/privkey.pem;
  location ~* ^/assets/ {
    access_log off;
    gzip_static on;
    expires max;
  }
  location ~* .(jpg|png|bmp|gif) {
        root /home/yourprojectpath/current/public;
                }
 try_files $uri/index.html $uri @forum;
 location @forum {
      proxy_set_header   Host $host;
      proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto https; ####  这里要特别注意，如果是http的话会不断的重定向以致不能访问网页
      proxy_redirect off;
      proxy_pass         http://forum;
    }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 4G;
  keepalive_timeout 10;
}
 ```
 
