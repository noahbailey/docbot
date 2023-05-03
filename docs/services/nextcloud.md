# Nextcloud

## Database

Install the "chaotic good" mysql server: 

    sudo apt install mariadb-server

Then, use the cli to create a new user and database for nextcloud: 

    sudo mysql

```sql
CREATE DATABASE nextcloud;
CREATE USER 'nextcloud' IDENTIFIED BY 'xxxxxxxxxxxxxxxxxxxxxxxxx';
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud';
```

## PHP components

Install all of the PHP pieces for the site: 

    sudo apt install php-fpm php8.2-curl php8.2-gd php8.2-xml php8.2-mbstring php8.2-zip php8.2-mysql php8.2-intl php8.2-bcmath php8.2-gmp php8.2-imagick imagemagick


Then, edit the main config to change the defaults: 

    sudo vim /etc/php/8.2/fpm/php.ini

```ini
memory_limit = 2048M

post_max_size = 2G
upload_max_filesize = 2G
```

It's also wise to install a memory cache as well: 

    sudo apt install php-apcu php8.2-apcu

Enable this for the site by adding to config.php

```php
'memcache.local' => '\OC\Memcache\APCu',
```

Then edit `/etc/php/8.2/mods-available/apcu.ini` and add: 

```ini
apc.enable_cli=1
```

This is required for `occ` and the cron jobs. 

## Nginx

Simply install Nginx web server: 

    sudo apt install nginx

And optionally the certbot components: 

    sudo apt install certbot python3-certbot-nginx


The nginx config: 


```ini
upstream php-handler {
    server unix:/var/run/php/php8.2-fpm.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name cloud.onetwoseven.one;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443      ssl http2;
    listen [::]:443 ssl http2;
    server_name cloud.onetwoseven.one;

    # Using LetsEncrypt for TLS:
    ssl_certificate     /etc/letsencrypt/live/cloud.onetwosevn.one/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cloud.onetwosevn.one/privkey.pem;
    ssl_protocols TLSv1.3 TLSv1.2;

    # HSTS settings
    add_header Strict-Transport-Security "max-age=15768000; includeSubDomains;" always;

    # set max upload size and increase upload timeout:
    client_max_body_size 2G;
    client_body_timeout 300s;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/wasm application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # HTTP response headers borrowed from Nextcloud `.htaccess`
    add_header Referrer-Policy                      "no-referrer"   always;
    add_header X-Content-Type-Options               "nosniff"       always;
    add_header X-Download-Options                   "noopen"        always;
    add_header X-Frame-Options                      "SAMEORIGIN"    always;
    add_header X-Permitted-Cross-Domain-Policies    "none"          always;
    add_header X-Robots-Tag                         "none"          always;
    add_header X-XSS-Protection                     "1; mode=block" always;

    # Remove X-Powered-By, which is an information leak
    fastcgi_hide_header X-Powered-By;

    # Path to the root of your installation
    root /var/www/nextcloud;

    index index.php index.html /index.php$request_uri;

    location = / {
        if ( $http_user_agent ~ ^DavClnt ) {
            return 302 /remote.php/webdav/$is_args$args;
        }
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # Make a regex exception for `/.well-known` so that clients can still
    # access it despite the existence of the regex rule
    # `location ~ /(\.|autotest|...)` which would otherwise handle requests
    # for `/.well-known`.
    location ^~ /.well-known {
        # The rules in this block are an adaptation of the rules
        # in `.htaccess` that concern `/.well-known`.

        location = /.well-known/carddav { return 301 /remote.php/dav/; }
        location = /.well-known/caldav  { return 301 /remote.php/dav/; }

        location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
        location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

        # Let Nextcloud's API for `/.well-known` URIs handle all other
        # requests by passing them to the front-end controller.
        return 301 /index.php$request_uri;
    }

    # Rules borrowed from `.htaccess` to hide certain paths from clients
    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)                { return 404; }

    location ~ \.php(?:$|/) {
        # Required for legacy support
        rewrite ^/(?!index|remote|public|cron|core\/ajax\/update|status|ocs\/v[12]|updater\/.+|oc[ms]-provider\/.+|.+\/richdocumentscode\/proxy) /index.php$request_uri;

        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        set $path_info $fastcgi_path_info;

        try_files $fastcgi_script_name =404;

        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $path_info;
        fastcgi_param HTTPS on;

        fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
        fastcgi_param front_controller_active true;     # Enable pretty urls
        fastcgi_pass php-handler;

        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ \.(?:css|js|svg|gif|png|jpg|ico|wasm|tflite)$ {
        try_files $uri /index.php$request_uri;
        expires 6M;         # Cache-Control policy borrowed from `.htaccess`
        access_log off;     # Optional: Don't log access to assets

        location ~ \.wasm$ {
            default_type application/wasm;
        }
    }

    location ~ \.woff2?$ {
        try_files $uri /index.php$request_uri;
        expires 7d;         # Cache-Control policy borrowed from `.htaccess`
        access_log off;     # Optional: Don't log access to assets
    }

    location /remote {
        return 301 /remote.php$request_uri;
    }

    location / {
        try_files $uri $uri/ /index.php$request_uri;
    }
}
```

You should also request a certificate: 

```sh
sudo certbot certonly --nginx
```

### Cron

The automated background job is an important part of how nextcloud tracks internal state. Ajax or web-cron are unreliable, it's best to use the built-in system. 

/etc/cron.d/nextcloud

    MAILTO=""
    */5  *  *  *  *   www-data   php -f /var/www/nextcloud/cron.php

## Antivirus

[Install Clamd](/services/clamav) and configure it.

In NextCloud settings, install and enable the "Antivirus for Files" application. 

Then, in the admin section change the mode to "Clamd Socket" to speed up scans. 


## Redis

Install redis and tools: 

    sudo apt install redis-server php-redis

Make sure the service is enabled on startup: 

    sudo systemctl enable redis-server

Restart php-fpm to use the new modules: 

    sudo systemctl restart php8.2-fpm

Then, modify the config.php file: 

```php
  'memcache.local' => '\OC\Memcache\APCu',
  'memcache.distributed' => '\OC\Memcache\Redis',
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' => [
    'host' => 'localhost',
    'port' => 6379,
  ],
```

After this is configured, the admin status page will update accordingly. 

## Email

The ideal way to configure email is using a local relay service, rather than programming the credentials into NextCloud. This gives you the robust retry queue and other very useful features. 

* [Postfix local relay](/services/postfix)
* [BSD OpenSMTPD relay](/services/opensmtpd)
