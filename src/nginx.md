# nginx

## Resources

* [nginx documentation](https://nginx.org/en/docs/)
* [nginxconfig](https://www.digitalocean.com/community/tools/nginx)
* [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)

## Sane default configurations

### `/etc/nginx/sites-available/example-static.com.conf` for fully static website

```
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name wiki.venn.dev;
	root /var/www/wiki.venn.dev/html;

	# SSL
	ssl_certificate /etc/letsencrypt/live/wiki.venn.dev/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/wiki.venn.dev/privkey.pem;

	# security
	add_header X-Frame-Options "SAMEORIGIN" always;
	add_header X-XSS-Protection "1; mode=block" always;
	add_header X-Content-Type-Options "nosniff" always;
	add_header Referrer-Policy "no-referrer-when-downgrade" always;
	add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

	# dotfiles
	location ~ /\.(?!well-known) {
		deny all;
	}

	# restrict methods
	if ($request_method !~ ^(GET|HEAD|CONNECT|OPTIONS|TRACE)$) {
		return '405';
	}

	# logging
	access_log /var/log/nginx/wiki.venn.dev.access.log;
	error_log /var/log/nginx/wiki.venn.dev.error.log warn;

	# favicon.ico
	location = /favicon.ico {
		log_not_found off;
		access_log off;
	}

	# robots.txt
	location = /robots.txt {
		log_not_found off;
		access_log off;
	}

	# assets, media
	location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm)$ {
		expires 7d;
		access_log off;
	}

	# svg, fonts
	location ~* \.(?:svgz?|ttf|ttc|otf|eot|woff2?)$ {
		add_header Access-Control-Allow-Origin "*";
		expires 7d;
		access_log off;
	}
}

server {
	listen 80;
	listen [::]:80;
	server_name wiki.venn.dev;

	location ^~ /.well-known/acme-challenge/ {
		root /var/www/_letsencrypt;
	}

	location / {
		return 301 https://wiki.venn.dev$request_uri;
	}
}
```

### `/etc/nginx/sites-available/example-uwsgi-django.com.conf` for Django + uWSGI

```
server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name venn.dev;

	# deny requests with invalid host header
	if ( $host !~* ^(venn.dev)$ ) {return 444;}

	# SSL
	ssl_certificate /etc/letsencrypt/live/venn.dev/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/venn.dev/privkey.pem;

	# security headers
	add_header X-Frame-Options "SAMEORIGIN" always;
	add_header X-XSS-Protection "1; mode=block" always;
	add_header X-Content-Type-Options "nosniff" always;
	add_header Referrer-Policy "no-referrer-when-downgrade" always;
	add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

	# . files
	location ~ /\.(?!well-known) {
		deny all;
	}

	# logging
	access_log /var/log/nginx/venn.dev.access.log;
	error_log /var/log/nginx/venn.dev.error.log warn;

	location / {
		include                       uwsgi_params;
		uwsgi_pass                    unix:/run/uwsgi/venn.sock;
		uwsgi_param Host              $host;
		uwsgi_param X-Real-IP         $remote_addr;
		uwsgi_param X-Forwarded-For   $proxy_add_x_forwarded_for;
		uwsgi_param X-Forwarded-Proto $http_x_forwarded_proto;
	}

	# Django static
	location /static/ {
		alias /opt/apps/venn/static/;
	}

	# favicon.ico
	location = /favicon.ico {
		log_not_found off;
		access_log off;
	}

	# robots.txt
	location = /robots.txt {
		log_not_found off;
		access_log off;
	}
}

server {
	listen 80;
	listen [::]:80;
	server_name venn.dev;

	# ACME-challenge
	location ^~ /.well-known/acme-challenge/ {
		root /var/www/_letsencrypt;
	}

	location / {
		return 301 https://venn.dev$request_uri;
	}
}
```

### `/etc/nginx/nginx.conf`

```
user www-data;
pid /run/nginx.pid;
worker_processes auto;
worker_rlimit_nofile 65535;
include /etc/nginx/modules-enabled/*.conf;

events {
	multi_accept on;
	worker_connections 768;
}

http {
	charset uf-8;
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	server_tokens off;
	types_hash_max_size 2048;
	client_max_body_size 16M;
	keepalive_timeout 65;

	server_names_hash_bucket_size 64;
	server_name_in_redirect off;

	# MIME
	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	# Logging
	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	# SSL
	ssl_session_timeout 1d;
	ssl_session_cache shared:SSL:10m;
	ssl_session_tickets off;
	ssl_prefer_server_ciphers on;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;

	# Gzip Settings
	gzip on;
	gzip_vary on;
	gzip_proxied any;
	gzip_comp_level 6;
	gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;

	# Virtual Host Configs
	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```
