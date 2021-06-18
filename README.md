# Nginx-PHPFPM containerization

Here is a simple project discussing about configuring a PHP website on a docker container using Nginx and php-fpm. 
Also I installed a self signed SSL certificate to it. 

I'm using docker compose to build the structure

#### Features:

 Since the processes are in separate containers, they are isolated by each other.

#### Requirements
1. Docker compose:
 
   Docker compose installation - [docker-compose](https://docs.docker.com/compose/install/)

   Docker Compose is used for running multiple containers as a single service. Each of the containers here run in isolation but can interact with each other when required. Docker Compose files are written in  YAML. Another great thing about Docker Compose is that users can activate all the services (containers) using a single command.

2. On docker it's not possible to route access to docker container by it's name which means, container name resolution is not possible on default docker network. Hence we need to create a new network via composer file. 

3. Also we have added a new volume so that the volume will be used by the container but get stored on the host machine. 


Let's see the docker compose file
```sh
version: "3"
services:
  myweb:
    image: myweb:1
    container_name: myweb
    networks:
      - myweb
    volumes:
      - myweb:/var/myweb

  nginx:
    image: nginx
    container_name: mynginx
    ports:
      - "80:80"
      - "443:443"
    networks:
      - myweb
    volumes:
      - myweb:/var/www/html/
      - ./default.conf:/etc/nginx/conf.d/default.conf
      - ./gigingeorge.online.crt:/root/myweb/gigingeorge.online.crt
      - ./gigingeorge.online.key:/root/myweb/gigingeorge.online.key
  phpfpm:
    image: php:7.4.20-fpm-alpine3.13
    container_name: myphp
    networks:
      - myweb
    volumes:
      - myweb:/var/www/html

volumes:
  myweb:

networks:
  myweb:
 ```
 
Here I used a custom docker image that's used to download the contents from my github and is mounted to the volume. Hence the volume attached with the PHP-FPM will have the website contents. 

```sh
FROM httpd:2.4.48-alpine
RUN apk update
RUN apk add  git
WORKDIR /var/myweb/
RUN git clone https://github.com/gigingeorge/aws-elb-site.git website
RUN mv /var/myweb/website/* /var/myweb/
CMD ["httpd-foreground"]
```

The nginx conf file is

```sh
server {
       listen 80;
       listen [::]:80;

       server_name gigingeorge.online;

        listen 443 default ssl;

    ssl_certificate      /root/myweb/gigingeorge.online.crt;
    ssl_certificate_key  /root/myweb/gigingeorge.online.key;


        root /var/www/html;
        index index.php;


   location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ /index.php?$args;
  }

  # pass the PHP scripts to FastCGI server listening on wordpress:9000
  location ~ \.php$ {
    fastcgi_split_path_info ^(.+\.php)(/.+)$;
    fastcgi_pass myphp:9000;
    include fastcgi_params;
    fastcgi_index index.php ;
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    fastcgi_param SCRIPT_NAME $fastcgi_script_name;
  }
}
```

To execute the docker compose, use the command
```sh
docker-compose up -d 
```

![alt text](https://i.ibb.co/Qc7FGn6/Screenshot.png)


