FROM alpine:latest

VOLUME ["/etc/haproxy","/etc/letsencrypt/archive"]
EXPOSE 80 443

RUN apk add --no-cache haproxy certbot openssl supervisor bash vim\
 && mkdir -p /etc/haproxy\
 && mkdir -p /etc/letsencrypt/certs.d\
 && openssl req -x509 -out /etc/letsencrypt/certs.d/default.crt -keyout /etc/letsencrypt/certs.d/default.key \
  -newkey rsa:2048 -nodes -sha256 \
  -subj '/CN=localhost'\
 && cat /etc/letsencrypt/certs.d/default.crt /etc/letsencrypt/certs.d/default.key | tee /etc/letsencrypt/certs.d/default.pem\
 && echo -e '#!/bin/sh\n\
 if [ ! -d "/etc/letsencrypt/live/$LETSENCRYPT_DOMAIN" ]; then\n\
     certbot certonly --standalone --domain $LETSENCRYPT_DOMAIN --email $LETSENCRYPT_EMAIL --http-01-port 8080 --agree-tos --no-eff-email\n\
     if [ -f "/etc/letsencrypt/live/$LETSENCRYPT_DOMAIN/fullchain.pem" ]; then\n\
         cp -f /etc/letsencrypt/live/$LETSENCRYPT_DOMAIN/fullchain.pem /etc/letsencrypt/certs.d/default.crt\n\
         cp -f /etc/letsencrypt/live/$LETSENCRYPT_DOMAIN/privkey.pem /etc/letsencrypt/certs.d/default.key\n\
         cat /etc/letsencrypt/certs.d/default.crt /etc/letsencrypt/certs.d/default.key | tee /etc/letsencrypt/certs.d/default.pem\n\
         sed -i "s/\@reboot.*//g" /etc/crontabs/root\n\
         echo -e -n "0\t0\t1\t*\t*\tsh /usr/bin/update-certs\n" >> /etc/crontabs/root\n\
         killall haproxy\n\
     fi\n\
 fi\n\
 ' > /usr/bin/certbot-certonly\
 && chmod +x /usr/bin/certbot-certonly\
 && echo -e '#!/bin/sh\n\
 certbot renew --force-renewal\n\
 if [ -f "/etc/letsencrypt/live/$LETSENCRYPT_DOMAIN/fullchain.pem" ]; then\n\
     cp -f /etc/letsencrypt/live/$LETSENCRYPT_DOMAIN/fullchain.pem /etc/letsencrypt/certs.d/default.crt\n\
     cp -f /etc/letsencrypt/live/$LETSENCRYPT_DOMAIN/privkey.pem /etc/letsencrypt/certs.d/default.key\n\
     cat /etc/letsencrypt/certs.d/default.crt /etc/letsencrypt/certs.d/default.key | tee /etc/letsencrypt/certs.d/default.pem\n\
     killall haproxy\n\
 fi\n\
 ' > /usr/bin/update-certs\
 && chmod +x /usr/bin/update-certs\
 && echo -e -n "@reboot\t\t\t\t\tsh /usr/bin/certbot-certonly\n" >> /etc/crontabs/root


 ADD supervisord.conf /etc/
 ENTRYPOINT ["supervisord", "--configuration", "/etc/supervisord.conf"]
