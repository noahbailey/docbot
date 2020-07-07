# Nginx

Typically used as a reverse proxy server, though it can certainly be used to serve up static content or use PHP or any other plugin. 

## Reverse Proxy

```nginx
server {
    listen 80;
    listen [::]:80;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    location / {
        proxy_http_version 1.1;
        proxy_pass http://localhost:8080;
    }

    ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
}
```

## With Caching

```nginx
proxy_cache_path  /var/www/cache levels=1:2 keys_zone=static-cache:8m max_size=99m inactive=600m;
proxy_temp_path /var/www/cache/tmp; 

server {
    listen 80;
    listen [::]:80;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    location / {
        proxy_http_version 1.1;
        proxy_pass http://localhost:8080;
        proxy_cache static-cache;
      	proxy_cache_valid  200 302  300s;
      	proxy_cache_valid  404      60s;
    }

    ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
    ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
}
```