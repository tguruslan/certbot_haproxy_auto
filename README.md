# haproxy-certbot

Auto-updatable container with haproxy and certbot
to use, enter your domain and email address in the variable, correct the haproxy configuration and run

<pre>
$ docker volume create cert
$ docker run -d\
 --name some-haproxy-certbot\
 -p 80:80\
 -p 443:443\
 --restart always\
 -v $(pwd)/config:/etc/haproxy\
 -v cert:/etc/letsencrypt/archive\
 -e LETSENCRYPT_EMAIL=mail@example.com\
 -e LETSENCRYPT_DOMAIN=example.com\
 tguruslan/certbot_haproxy_auto:latest
</pre>
example docker-compose.yml
<pre>
version: '3'

services:
  haproxy:
    image: tguruslan/certbot_haproxy_auto:latest
    environment:
      LETSENCRYPT_EMAIL: mail@example.com
      LETSENCRYPT_DOMAIN: example.com
    ports:
      - 80:80
      - 443:443
    restart: always
    volumes:
      - ./config:/etc/haproxy
      - cert:/etc/letsencrypt/archive
    networks:
      haproxy:

networks:
  haproxy:
    driver: bridge

volumes:
  cert:
</pre>
example ./config/haproxy.cfg
<pre>
global
    maxconn 4096

    ca-base /etc/ssl/certs
    crt-base /etc/ssl/private

    ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
    ssl-default-bind-options no-sslv3

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http-in
    bind *:80

    acl host_1 path_beg /path
    use_backend host_1 if host_1

    acl letsencrypt_http_acl path_beg /.well-known/acme-challenge/
    use_backend letsencrypt_http if letsencrypt_http_acl

    default_backend host_1

frontend https-in
    bind *:443 ssl crt /etc/letsencrypt/certs.d/default.pem ciphers ECDHE-RSA-AES256-SHA:RC4-SHA:RC4:HIGH:!MD5:!aNULL:!EDH:!AESGCM
    mode http

    acl host_1 path_beg /path
    use_backend host_1 if host_1

    default_backend host_1

backend letsencrypt_http
    mode http
    server letsencrypt_http_srv 127.0.0.1:8080

backend host_1
    mode http
    http-request replace-path /path/(.*) /\1
    http-request add-header X-Forwarded-Proto https
    http-request add-header X-Forwarded-For %[src]
    server h1 127.0.0.1:80 maxconn 32 # replace 127.0.0.1 by your domain or ip
</pre>
