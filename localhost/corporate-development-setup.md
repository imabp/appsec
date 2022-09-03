- Assumptions
    - Linux/ MacOS

## Create Certificates

- Corporate Engineers should never do development on localhost.
- Every corporate level api-endpoint, has a `API-Origin` and set of trusted domains for security reasons. Its recommended to map your locahost port(`127.0.0.1`) to the corporate development environment.

- First step is to setup certificates

```sh
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
-keyout DEVELOPMEN_SITE_KEY.key -out DEVELOPMENT_SITE_CERT.crt -extensions san -config \
<(echo "[req]";
echo distinguished_name=req;
echo "[san]";
echo subjectAltName=DNS:'*.DEVELOPMENT_SITE.com',IP:127.0.0.1
) \
-subj /CN='*.DEVELOPMENT_SITE.com'

```

`DEVELOPMENT_SITE` -> Should be replaced with corporate development environment.


Are you on windows? Read this [blog](https://kaushikghosh12.blogspot.com/2016/08/self-signed-certificates-with-microsoft.html)

## Setup NGINX

- First download [nginx](https://nginx.org/en/download.html). Read [this](https://dev.to/arjavdave/installing-nginx-on-mac-46ac) for MacOS
- To view NGINX configuration file location, `nginx -t` 
- Open the directory of NGINX configuration, and paste your certs generated in above step into `certs` folder
- Open `nginx.conf` file in your vscode, and paste the following.
- Replace `DEVELOPMENT_SITE` with your corporate development environment.

```sh
worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    #gzip  on;

    server {
      listen 443 ssl;
      server_name  *.DEVELOPMENT_SITE.com;
	    client_max_body_size 100M;
	
	    ssl_certificate /usr/local/etc/nginx/certs/DEVELOPMENT_SITE_CERT.crt;
	    ssl_certificate_key /usr/local/etc/nginx/certs/DEVELOPMEN_SITE_KEY.key;

	    underscores_in_headers on;

        location / {
	        proxy_pass http://localhost:3000/;
	        proxy_set_header Host $host;

          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection $http_connection;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

    include servers/*;
}
```

## Finally Setup hosts

Find out host file, based on your operating system

- for mac/linux: `/etc/hosts`
- for win: `c:\Windows\System32\Drivers\etc\hosts`

Add the following to the file.
*(Do not delete anything from hosts file, these are used for booting up your OS, so be careful, just add to the next line)*

```sh
127.0.0.1     DEVELOPMENT_SITE.com
```
