# Nginx with Quic/ HTTP3

```nginx
ssl_session_cache   shared:SSL:10m;
ssl_session_timeout 10m;

server {
       http2 on;
       listen 443 ssl;
       listen [::]:443 ssl;

       http3 on;
       listen 443 quic reuseport;
       listen [::]:443 quic reuseport;

       ssl_certificate /etc/nginx/ssl/domain.cert.pem;
       ssl_certificate_key /etc/nginx/ssl/private.key.pem;

       server_name **SERVER_NAME_HERE**;
       keepalive_timeout 70;

       root /var/www/html;
       index index.html index.nginx-debian.html;

       location / {
               add_header Alt-Svc 'h3=":443"; ma=86400';
               try_files $uri $uri/ =404;
       }
}
```
