# Install Nginx on Ubuntu 16.04

**User Login**
```sh
local $ ssh user@your_server_ip
```

**Install Nginx**
```sh
remote $ sudo apt-get update
remote $ sudo apt-get install nginx
```

**Adjust the Firewall**
```sh
remote $ sudo ufw allow 'Nginx HTTP'
remote $ sudo ufw status
```

**Check your Web Server**
```sh
remote $ systemctl status nginx
```
output
```sh
nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2016-04-18 16:14:00 EDT; 4min 2s ago
 Main PID: 12857 (nginx)
   CGroup: /system.slice/nginx.service
           ├─12857 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
           └─12858 nginx: worker process
```

**Manage the Nginx Process**
To stop your web server, you can type:
```sh
remote $ sudo systemctl stop nginx
```
To start the web server when it is stopped, type:
```sh
remote $ sudo systemctl start nginx
```
To stop and then start the service again, type:
```sh
remote $ sudo systemctl restart nginx
```
If you are simply making configuration changes, Nginx can often reload without dropping connections. To do this, this command can be used:
```sh
remote $ sudo systemctl reload nginx
```

**Get Familiar with Important Nginx Files and Directories**
Content
- /var/www/html: The actual web content, which by default only consists of the default Nginx page you saw earlier, is served out of the /var/www/html directory. This can be changed by altering Nginx configuration files.

Server Configuration
- /etc/nginx: The nginx configuration directory. All of the Nginx configuration files reside here.
- /etc/nginx/nginx.conf: The main Nginx configuration file. This can be modified to make changes to the Nginx global configuraiton.
- /etc/nginx/sites-available: The directory where per-site "server blocks" can be stored. Nginx will not use the configuration files found in this directory unless they are linked to the sites-enabled directory (see below). Typically, all server block configuration is done in this directory, and then enabled by linking to the other directory.
- /etc/nginx/sites-enabled/: The directory where enabled per-site "server blocks" are stored. Typically, these are created by linking to configuration files found in the sites-available directory.
- /etc/nginx/snippets: This directory contains configuration fragments that can be included elsewhere in the Nginx configuration. Potentially repeatable configuration segments are good candidates for refactoring into snippets.

Server Logs
- /var/log/nginx/access.log: Every request to your web server is recorded in this log file unless Nginx is configured to do otherwise.
- /var/log/nginx/error.log: Any Nginx errors will be recorded in this log.

**Update /etc/nginx/sites-enabled/ for handle React router**
```sh
remote $ sudo nano /etc/nginx/sites-enabled/default

server {
    listen 80;

    server_name your_server_ip;
    root /www/app_name/build;
    index index.html index.htm;
    rewrite ^/(.*)/$ $1 permanent;

    # Any route containing a file extension (e.g. /devicesfile.js)
    location ~ ^.+\..+$ {
      try_files $uri =404;
    }
    
    location / {
        try_files $uri $uri/ /index.html;
    }

    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
        return 404;
    }

}

remote $ sudo systemctl restart nginx

```

### Reference

| Title | Url |
| ------ | ------ |
| How To Install Nginx on Ubuntu 16.04 | https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04] [PlDb] |
