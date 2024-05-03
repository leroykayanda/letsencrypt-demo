
This repo is a tutorial on how to set up letsencrypt certificates on an nginx container.

docker-compose.yaml

    version: "3"
    services:
      nginx:
        image: nginx
        ports:
          - 80:80
          - 443:443
        volumes:
          - /home/azure/tests/site-files:/usr/share/nginx/html
          - /home/azure/tests/nginx/default.conf:/etc/nginx/conf.d/default.conf
          - /home/azure/tests/nginx/ssl-params.conf:/etc/ssl/ssl-params.conf
          - /home/azure/tests/nginx/dhparam.pem:/etc/ssl/certs/dhparam.pem
          - /home/azure/tests/letsencrypt/etc:/etc/letsencrypt

**volumes**

`- /home/azure/tests/site-files:/usr/share/nginx/html`

Website files are stored here

`- /home/azure/tests/nginx/default.conf:/etc/nginx/conf.d/default.conf`

This is the nginx configuration file

`- /home/azure/tests/nginx/ssl-params.conf:/etc/ssl/ssl-params.conf`

These are the SSL settings

`- /home/azure/tests/nginx/dhparam.pem:/etc/ssl/certs/dhparam.pem`

This is a file containing Diffie-Hellman parameters used in the SSL/TLS handshake. The contents have been redacted
 
`- /home/azure/tests/letsencrypt/etc:/etc/letsencrypt`

Letsencrypt will store certificates and logs here.

All instances of example.com need to be replaced with your domain name.

Use the command below to create a certificate in letsencrypt's staging environment to avoid rate limits while testing.

    docker run -it --rm \
    -v "/home/azure/tests/letsencrypt/etc:/etc/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/lib:/var/lib/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/log:/var/log/letsencrypt" \
    -v "/home/azure/tests/site-files:/data/letsencrypt" \
    certbot/certbot \
    certonly --webroot \
    --register-unsafely-without-email --agree-tos --no-eff-email \
    --webroot-path=/data/letsencrypt \
    --staging \
    -d example.com -d www.example.com

Once you are done testing, use this for prod.

    docker run -it --rm \
    -v "/home/azure/tests/letsencrypt/etc:/etc/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/lib:/var/lib/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/log:/var/log/letsencrypt" \
    -v "/home/azure/tests/site-files:/data/letsencrypt" \
    certbot/certbot \
    certonly --webroot \
    --email test@gmail.com --agree-tos --no-eff-email \
    -d example.com -d www.example.com

Set up a cron to run this to automate renewals.

    docker run --rm  \
    --name certbot \
    -v "/home/azure/tests/letsencrypt/etc:/etc/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/lib:/var/lib/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/log:/var/log/letsencrypt" \
    -v "/home/azure/tests/site-files:/data/letsencrypt" \
    certbot/certbot renew --webroot -w /data/letsencrypt  

To set up a wildcard cert, we have to use [dns challenge](https://letsencrypt.org/docs/challenge-types/). Note that we specify that we shall create a TXT manually using the --manual flag. This certificate thus cannot be renewed automatically. We have to run the cmd below again when we want to renew.

staging

    docker run -it --rm \
    -v "/home/azure/tests/letsencrypt/etc:/etc/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/lib:/var/lib/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/log:/var/log/letsencrypt" \
    certbot/certbot \
    certonly \
    --manual \
    --preferred-challenges=dns \
    --register-unsafely-without-email --agree-tos --no-eff-email \
    --staging \
    -d *.example.com


prod

    docker run -it --rm \
    -v "/home/azure/tests/letsencrypt/etc:/etc/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/lib:/var/lib/letsencrypt" \
    -v "/home/azure/tests/letsencrypt/log:/var/log/letsencrypt" \
    certbot/certbot \
    certonly \
    --manual \
    --preferred-challenges=dns \
    --email test@gmail.com --agree-tos --no-eff-email \
    -d *.example.com 
