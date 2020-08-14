# Certbot for Nginx

## Install

    sudo apt install certbot python3-certbot-nginx

## Request a cert

    sudo certbot certonly --nginx


## Install the cert

Certs are stored: 

```
/etc/letsencrypt/live/my-cool-domain.net/fullchain.pem
/etc/letsencrypt/live/my-cool-domain.net/privkey.pem
```

Update the nginx config: 

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    location / {
        proxy_http_version 1.1;
        proxy_pass http://localhost:8080;
    }

    ssl_certificate     /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
}
```
