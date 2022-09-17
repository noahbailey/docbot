# Nginx

Typically used as a reverse proxy server, though it can certainly be used to serve up static content or use PHP or any other plugin. 

## Test-and-reload

A quick and convenient one-liner to safely reload nginx: 

    sudo nginx -t && sudo systemctl reload nginx

## Reverse Proxy

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name mycooldomain.net; 
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name mycooldomain.net; 

    ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_http_version 1.1;
        proxy_pass http://localhost:8080;
    }
}
```

## With Caching

```nginx
proxy_cache_path  /var/www/cache levels=1:2 keys_zone=static-cache:8m max_size=99m inactive=600m;
proxy_temp_path /var/www/cache/tmp; 

server {
    listen 80;
    listen [::]:80;
    server_name mycooldomain.net; 
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name mycooldomain.net; 

    ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    location / {
        proxy_http_version 1.1;
        proxy_pass http://localhost:8080;
        proxy_cache static-cache;
      	proxy_cache_valid  200 302  300s;
      	proxy_cache_valid  404      60s;
    }
}
```

## PHP-FPM 

Install php goodies (Usually needed for LEMP stack):

    sudo apt install php7.4 php7.4-fpm php7.4-mbstring php7.4-json php7.4-xml php-mysql php7.4-mysql php7.4-gd php7.4-zip php7.4-curl

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name mycooldomain.net; 

    ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    root           /var/www/myapp/public;
    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## Authentication

Install the PAM module:

    sudo apt install libnginx-mod-http-auth-pam

Create the PAM config at `/etc/pam.d/nginx`

    auth       include      common-auth
    account    include      common-account

Allow nginx to read the shadow file:

    usermod -aG shadow www-data 

Add lines to the proxy config:

```
    location / {
        auth_pam "secure";
        auth_pam_service_name "nginx";
        proxy_http_version 1.1;
        proxy_pass http://localhost:8008;
    }
```

## The Go-Away-Vhost

Default vhost to discourage spam. This should be added to the bottom of `/etc/nginx/nginx.conf`

```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    server_name_in_redirect off;

    location /healthz {
        return 200;
        access_log off; 
    }

    location / {
        return 301 http://$remote_addr$request_uri;
        access_log /var/log/nginx/spam.log; 
    }
}

server {
    listen 443 ssl  default_server;
    listen [::]:443 ssl default_server;
    server_name _;
    ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
    ssl_protocols TLSv1.3 TLSv1.2;
    server_name_in_redirect off;

    location /healthz {
        return 200;
        access_log off; 
    }

    location / {
        return 301 http://$remote_addr$request_uri;
        access_log /var/log/nginx/spam.log; 
    }
}
```

This configuration also provides a `/health` endpoint that returns an empty success response, which helps keep monitoring tools happy.  