# laravel-docker
php-fpm image of laravel. there is no laravel installed, just bunch of laravel app depedencies that you can mount laravel app into it.
based on this tuturial with trivial modification:
https://www.digitalocean.com/community/tutorials/how-to-containerize-a-laravel-application-for-development-with-docker-compose-on-ubuntu-18-04

## Install
clone this repository and build docker image.
```
cd /var/www/html
https://github.com/amsaravi/laravel-docker.git
cd laravel-docker
docker build . -t laravel-fpm --build-arg user=nginx uid=994
```
Then run docker as a server where /var/www/html/laravel-docker is where your laravel project resides.

```
docker run --name laravel-test -v /var/www/html/laravel-docker:/var/www/html -d -p 9000:9000
```
As this image is based on php-fpm it serves php-fcgi in port 9000. you should configure your web server to use your host and port address (nginx configuration will follow this section)

## Nginx config
Nginx docker configuration is instructed in the base tuturial at https://www.digitalocean.com/community/tutorials/how-to-containerize-a-laravel-application-for-development-with-docker-compose-on-ubuntu-18-04. 
i will show you how to configure host nginx. the two nginx fast_cgi params, "fastcgi_param SCRIPT_FILENAME" and "fastcgi_pass" are the most important params you should play with to configure fastcgi in nginx.

```
    
    server {
        listen       80;
        listen       [::]:80;
        server_name  textograph2.digitaltoxicity.ir;
        root         /var/www/html/laravel-docker/public;
        include /etc/nginx/default.d/*.conf;
        index index.php index.html;
        error_log  /var/log/nginx/error.log;
        access_log /var/log/nginx/access.log;
        location ~ \.php$ {
                try_files $uri =404;	# if you put your laravel files within docker image and cant access them within host you should comment in this line
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME /var/www/html/public/$fastcgi_script_name;
                fastcgi_param PATH_INFO $fastcgi_path_info;
        }
        location / {
        try_files $uri $uri/ /index.php?$query_string;
        gzip_static on;
        }
        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;

    }
```
