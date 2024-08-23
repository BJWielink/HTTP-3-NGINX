# Nginx with Quic/ HTTP3

## Install Nginx

`cd /tmp`\
`sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring`

```
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
```

`gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg`

```
echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

```
echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx
```

`sudo apt update`\
`sudo apt install nginx`

## SSL

`nano /etc/nginx/domain.cert.pem`\
`nano /etc/nginx/private.key.pem`

## Nginx configuration

`nano /etc/nginx/conf.d/default.conf`

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
