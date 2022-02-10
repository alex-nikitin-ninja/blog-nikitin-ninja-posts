# nginx + letsencrypt + docker = multistack web app

**Difficulty Level**: entry

Imagine that we have a few different web services which work on different
languages, environment with its own configuration set.

What would be the approach of running applications on staging or live
environment easily without overengineering and easy deployment process with CI
pipelines.

Obvious first response is to use **docker** for wrapping our applications into
different environmemts.

But how would we route traffic from incoming requests into various docker
containers?

The answer is to use **nginx** as a reverse proxy and route traffic based on domain
name or path name.

And for taste of being encrypted and have all those services and microservices
behind https we use free **Let's encrypt** certificates and their bot to keep
certificates and configuration up-to-date automatically.

-more-

## Input data

We have:

- a FQDN: `example.com`
- a VPS or bare metal `server` (let's say debian/ubuntu)
- example.com domain has a `DNS record` pointing to our server interface
- an api backend web application wrapped into a container image (let's call it
`api-container`  and let's assume it needs to respond to `api.example.com`)
- a SPA web application wrapped into another container image (let's call it
`spa-container` and let's assume it needs to respond to `example.com` directly)

## Action

### Installation steps
1. **ssh** into our server
2. Install **nginx**:
```
$ apt-get install nginx
```
3. Install **docker**:  
([https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/))
4. Install **certbot**:
```
$ apt-get install certbot
```
5. Install certbot **plugin** for nginx (`python3-certbot-nginx`):
```
$ apt-get install python3-certbot-nginx
```

### Configuration steps

1. Configure **nginx** to work as a reverse proxy for `spa-container` and
`api-container`, so open conf file (use editor of your choise nano/vi/vim/etc):
```
$ nano /etc/nginx/conf.d/default.conf
```
Add entries for two new server sections:
```
...
server {
    listen 80;
    server_name example.com;
    location / {
         proxy_pass  http://127.0.0.1:8001/;
         proxy_set_header Host $host;
         proxy_set_header X-Forwarded-Proto $scheme;
         proxy_pass_request_headers on;
    }
}

server {
    listen 80;
    server_name api.example.com;
    location / {
        proxy_pass  http://127.0.0.1:8002/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass_request_headers on;
    }
}
...
```
2. Save the file and run a new certificate retrieval:
one cert for each domain:
```
$ certbot --nginx -d example.com
$ certbot --nginx -d api.example.com
```
or a single certificate covering two domains:
```
$ certbot --nginx -d example.com -d api.example.com
```
Follow instructions from console and follow to the next step

3. Verify certificates being added to nginx configuration (lines commented as `# managed by Certbot`):
```
...
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_pass  http://127.0.0.1:8002/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass_request_headers on;
    }

    listen 443 ssl http2; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    listen 80;
    server_name api.example.com;
    location / {
        proxy_pass  http://127.0.0.1:8002/;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass_request_headers on;
    }

    listen 443 ssl http2; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}
...
```






