# Deploying Nodejs Angular Fullstack app with Nginx

**Step 1 — Installing Nginx**
Following this link
https://github.com/manjarb/deployment_document/blob/master/Nginx-setup.md

**Step 2 — Installing Nodejs**
Following this link
https://github.com/manjarb/deployment_document/blob/master/Nodejs-setup.md

**Step 3 — Setup Github Account and clone the repo**
Following this link
https://github.com/manjarb/deployment_document/blob/master/Setup-Github-and-cloning-Repository.md

**Step 4 — Installing PM2**
Now we will install PM2, which is a process manager for Node.js applications. PM2 provides an easy way to manage and daemonize applications (run them in the background as a service).
```sh
remote $ sudo npm install -g pm2
```

**Step 5 — Pull the latest code and Build the production Front end**
```sh
remote $ cd repo_folder
remote $ npm run pull-and-build
```

**Step 6 — Run PM2 on Production Background**
```sh
remote $ npm run prod-background
```

**Step 7 — Set Up Nginx as a Reverse Proxy Server**
```sh
remote $ sudo nano /etc/nginx/sites-available/default
```

Update nginx.conf due to the setting on the current repo Ex.
```sh
server {
    listen 80;
    listen [::]:80;
    server_name esanyoung.amusexd.com;
    return 301 https://esanyoung.amusexd.com$request_uri;
}

server {
    # listen 80;
    server_name esanyoung.amusexd.com;
    ssl on;

    client_max_body_size 75m;

    location / {
        root themall_esan_young;
        index index.html;
        try_files $uri $uri/ @api; # instead of 404, proxy back to express using a named location block;
        expires max;
        access_log off;
        # source: https://stackoverflow.com/a/15467555/8436941
        client_max_body_size 75m;
    }

    location @api {
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:5000;
        proxy_redirect off;

        client_max_body_size 75m;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/esanyoung.amusexd.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/esanyoung.amusexd.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
```

Make sure you didn't introduce any syntax errors by typing:
```sh
remote $ sudo nginx -t
remote $ sudo systemctl restart nginx
```

### Reference

| Title | Url |
| ------ | ------ |
| How To Set Up a Node.js Application for Production on Ubuntu 16.04 | [https://www.digitalocean.com/community/tutorials/how-to-set-up-a-node-js-application-for-production-on-ubuntu-16-04] [PlDb] |
