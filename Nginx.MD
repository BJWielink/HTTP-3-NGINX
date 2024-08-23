# Nginx with Quic/ HTTP3

## Install Nginx

`sudo apt update && sudo apt upgrade`\
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

`mkdir ssl`\
`nano /etc/nginx/ssl/domain.cert.pem`\
`nano /etc/nginx/ssl/private.key.pem`

## Nginx configuration

`cd /etc/nginx`\
`nano nginx.conf`\

```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	# include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}
```

`mkdir sites-available`\
`mkdir sites-enabled`\
`nano sites-available/**SERVER_NAME_HERE**`

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

       root /usr/share/nginx/html;
       index index.html index.html;

       location / {
               add_header Alt-Svc 'h3=":443"; ma=86400';
               try_files $uri $uri/ =404;
       }
}
```

`sudo ln -s /etc/nginx/sites-available/**SERVER_NAME_HERE** /etc/nginx/sites-enabled/**SERVER_NAME_HERE**`
