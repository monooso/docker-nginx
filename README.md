# Nginx for Docker #
My standard Nginx Dockerfile for PHP development. Primarily concerned with [Craft CMS][craft] and [Laravel][laravel] projects, but it's so simple it'll probably work with other frameworks and languages.

[craft]: https://craftcms.com/
[laravel]: https://laravel.com/

## Configuration ##

### Ports ###
Don't forget to map whatever port your site configuration is listening on.

### Volumes ###
The image defines two volumes:

1. `/etc/nginx/conf.d`, which should contain your `site.conf` files.
2. `/var/code`, which should contain your application code.

Example usage:

```bash
docker run \
    -d \
    -p 80:80 \
    -v ~/myapp/code:/var/code \
    -v ~/myapp/nginx/config:/etc/nginx/conf.d \
    monooso/docker-nginx:latest
```

## HTTPS support ##
The image contains a self-signed SSL certificate, valid for all `*.local.vm` domains. Your browser will still issue stern warnings, but if you ignore them your site will happily function over HTTPS.

## Example `site.conf` file ##

```nginx
server {
    listen 80;
    listen 443 ssl;
    server_name example.local.vm;

    charset utf-8;

    # SSL
    ssl_certificate      /etc/ssl/local.vm/local.vm.crt;
    ssl_certificate_key  /etc/ssl/local.vm/local.vm.key;
    ssl on;
    ssl_session_cache  builtin:1000  shared:SSL:10m;
    ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!eNULL:!EXPORT:!CAMELLIA:!DES:!MD5:!PSK:!RC4;
    ssl_prefer_server_ciphers on;

    # The path to the code in the nginx container
    root /var/code/app/public;

    # "Directory index" files
    index index.html index.php;

    # Increase the max body size to allow large file uploads
    client_max_body_size 1024M;

    # Disable reading of hidden files and directories.
    location ~ /\.+ {
        deny all;
    }

    # Do not record attempts to access the favicon.
    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    # Do not record attempts to access the robots.txt.
    location = /robots.txt {
        access_log off;
        log_not_found off;
    }

    # Serve static assets.
    location /assets {
        try_files $uri $uri/ =404;
    }

    # Root directory location handler.
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Pass PHP requests to the application container.
    location ~ \.php$ {
        # The path to the code in the application container.
        root /var/code/public;

        fastcgi_split_path_info ^(.+\.php)(/.+)$;

        # Pass the request to php-fpm, on port 9000.
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
    }

    # Misc settings.
    sendfile off;
}
```

## Building ##
Build the Docker images from the repository root. For example:

```bash
docker build -t monooso/docker-nginx:1.16 -f Dockerfile-1.16 .
```
