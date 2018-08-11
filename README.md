# Docker Gogs MySQL [![Build Status](https://travis-ci.org/nanoninja/docker-gogs-mysql.svg?branch=master)](https://travis-ci.org/nanoninja/docker-gogs-mysql) [![GitHub version](https://badge.fury.io/gh/nanoninja%2Fdocker-gogs-mysql.svg)](https://badge.fury.io/gh/nanoninja%2Fdocker-gogs-mysql)

## [Gogs](https://gogs.io/) is a painless self-hosted Git service

*NOTE/Disclaimer:* This is a fork of the awesome repo (that made things much easier for me!) however, I found that as I'd setup nginx on my server with subdomain redirect (gogs.mydomain.com) that the install scripts on this repo didn't quite set things up correctly for me, as it was designed to work with a domain:port address.

Checkout the bottom of this readme for an example nginx config.

### Getting started

1. Clone project :

    ```sh
    git clone https://github.com/nanoninja/docker-gogs-mysql.git
    ```

2. You could customize your settings before installation :

    Edit `.env` file and `etc/app.ini.dist`

3. Install :

    use [Makefile](https://en.wikipedia.org/wiki/Makefile)

    ```sh
    # show commands
    make help

    sudo make install
    ```

    or by entering the following commands

    ```sh
    # Copy the configuration file from the dist file
    cp etc/app.ini.dist etc/app.ini

    # Start services
    sudo docker-compose up -d

    # Generate self-signed certificates
    source .env && sudo docker-compose exec -T gogsapp bash -c "cd /app/gogs; exec /app/gogs/gogs cert -ca=true -duration=$GOGS_CERT_DURATION -host=$GOGS_HTTP_DOMAIN"

    # Copy the configuration file to the container
    sudo docker cp $(pwd)/etc/app.ini $(sudo docker-compose ps -q gogsapp):/data/gogs/conf/app.ini

    # Restart the server to reload the configuration
    sudo docker-compose restart gogsapp

    # Automatic form filling with cURL
    sudo docker run --env-file $(pwd)/.env --rm -v $(pwd)/bin/install.sh:/install.sh --net=host appropriate/curl /bin/sh /install.sh
    ```

3. Open your favorite browser :

    - [https://localhost:10080/](https://localhost:10080)

---

## Using Git with SSH

### Configure Git to trust a self-signed certificate

#### Local

```sh
git -c http.sslVerify=false push origin master
```

#### Global 

```sh
git config --global http.sslVerify false
```

#### Unset Global

```sh
git config --global --unset http.sslVerify
```

---

### Example Nginx Config

[This is using SSL setup using certbot](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)

```
server {
    listen 443 ssl;
    server_name gogs.mydomain.com;
    location / {
            proxy_pass https://mydomain.com:10080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_connect_timeout 150;
            proxy_send_timeout 100;
            proxy_read_timeout 100;
            proxy_buffers 4 32k;
            client_max_body_size 8m;
            client_body_buffer_size 128k;

    }

    ssl_certificate /etc/letsencrypt/live/gogs.mydomain.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/gogs.mydomain.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
```

---

## Images to use

- [Gogs](https://hub.docker.com/r/gogs/gogs/)
- [MySQL](https://hub.docker.com/_/mysql/)
- [cURL](https://hub.docker.com/r/appropriate/curl/)
