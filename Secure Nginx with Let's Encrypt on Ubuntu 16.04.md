# How To Secure Nginx with Let's Encrypt on Ubuntu 16.04

**Step 1: Install Certbot**

```sh
remote $ sudo add-apt-repository ppa:certbot/certbot
remote $ sudo apt-get update
remote $ sudo apt-get install certbot
```

**Step 2: Obtain an SSL Certificate**

We'll show you how to use the Webroot plugin to obtain an SSL certificate.

How To Use the Webroot Plugin

```sh
remote $ sudo nano /etc/nginx/sites-available/default
```

Inside the server block, add this location block:
```sh
location ~ /.well-known {
        allow all;
}
```

You will also want look up what your document root is set to by searching for the root directive, as the path is required to use the Webroot plugin. 
If you're using the default configuration file, 
the root will be `/var/www/html`.

Save and exit.

Check your configuration for syntax errors:
```sh
remote $ sudo nginx -t
```

If no errors are found, restart Nginx with this command:
```sh
remote $ sudo systemctl restart nginx
```

Now that we know our webroot-path, we can use the Webroot plugin to request an SSL certificate with these commands. Here, we are also specifying our domain names with the -d option. If you want a single cert to work with multiple domain names (e.g. example.com and www.example.com), be sure to include all of them. Also, make sure that you replace the highlighted parts with the appropriate webroot path and domain name(s):
```sh
sudo certbot certonly --webroot --webroot-path=/var/www/html -d example.com -d www.example.com
```

If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, you should see a message telling you the process was successful and where your certificates are stored:
```sh
Output
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at
   /etc/letsencrypt/live/example.com/fullchain.pem. Your cert
   will expire on 2017-07-26. To obtain a new or tweaked version of
   this certificate in the future, simply run certbot again. To
   non-interactively renew *all* of your certificates, run "certbot
   renew"
 - If you lose your account credentials, you can recover through
   e-mails sent to sammy@example.com.
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

You will want to note the path and expiration date of your certificate, highlighted in the example above.
```sh
Firewall Note: If you receive an error like Failed to connect to host for DVSNI challenge, your server's firewall may need to be configured to allow TCP traffic on port 80 and 443.

Note: If your domain is routing through a DNS service like CloudFlare, you will need to temporarily disable it until you have obtained the certificate.
```

**Certificate Files**

After obtaining the cert, you will have the following PEM-encoded files:

- cert.pem: Your domain's certificate
- chain.pem: The Let's Encrypt chain certificate
- fullchain.pem: cert.pem and chain.pem combined
- privkey.pem: Your certificate's private key

It's important that you are aware of the location of the certificate files that were just created, so you can use them in your web server configuration. The files themselves are placed in a subdirectory in /etc/letsencrypt/archive. However, Certbot creates symbolic links to the most recent certificate files in the /etc/letsencrypt/live/your_domain_name directory. Because the links will always point to the most recent certificate files, this is the path that you should use to refer to your certificate files.

You can check that the files exist by running this command (substituting in your domain name):
```sh
remote $ sudo ls -l /etc/letsencrypt/live/your_domain_name
```

**Generate Strong Diffie-Hellman Group**

To further increase security, you should also generate a strong Diffie-Hellman group. To generate a 2048-bit group, use this command:
```sh
remote $ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
This may take a few minutes but when it's done you will have a strong DH group at /etc/ssl/certs/dhparam.pem.


**Step 3: Configure TLS/SSL on Web Server (Nginx)**

First, let's create a new Nginx configuration snippet in the /etc/nginx/snippets directory.

To properly distinguish the purpose of this file, we will name it ssl- followed by our domain name, followed by .conf on the end:
```sh
remote $ sudo nano /etc/nginx/snippets/ssl-example.com.conf
```

Within this file, we just need to set the ssl_certificate directive to our certificate file and the ssl_certificate_key to the associated key. In our case, this will look like this:
```sh
/etc/nginx/snippets/ssl-example.com.conf

ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
```

Next, we will create another snippet that will define some SSL settings. This will set Nginx up with a strong SSL cipher suite and enable some advanced features that will help keep our server secure.

The parameters we will set can be reused in future Nginx configurations, so we will give the file a generic name:
```sh
remote $ sudo nano /etc/nginx/snippets/ssl-params.conf
```

To set up Nginx SSL securely, we will be using the recommendations by Remy van Elst on the Cipherli.st site. This site is designed to provide easy-to-consume encryption settings for popular software. You can read more about his decisions regarding the Nginx choices here.
```sh
/etc/nginx/snippets/ssl-params.conf

# from https://cipherli.st/
# and https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_ecdh_curve secp384r1;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
# disable HSTS header for now
#add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;

ssl_dhparam /etc/ssl/certs/dhparam.pem;
```

**Adjust the Nginx Configuration to Use SSL**
We will assume in this guide that you are using the default server block file in the /etc/nginx/sites-available directory. If you are using a different server block file, substitute it's name in the below commands.

Before we go any further, let's back up our current server block file:
```sh
remote $ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak
```

Now, open the server block file to make adjustments:
```sh
sudo nano /etc/nginx/sites-available/default
```

Inside, your server block probably begins like this:
```sh
/etc/nginx/sites-available/default

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # SSL configuration

    # listen 443 ssl default_server;
    # listen [::]:443 ssl default_server;

    . . .
```

We will be splitting the configuration into two separate blocks. After the two first listen directives, we will add a server_name directive, set to your server's domain name. We will then set up a redirect to the second server block we will be creating. Afterwards, we will close this short block:
```sh
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

    # SSL configuration

    # listen 443 ssl default_server;
    # listen [::]:443 ssl default_server;

    . . .
```

Next, we need to start a new server block directly below to contain the remaining configuration. We can uncomment the two listen directives that use port 443. We can add http2 to these lines in order to enable HTTP/2 within this block. Afterwards, we just need to include the two snippet files we set up:
```sh
/etc/nginx/sites-available/default

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;
}

server {

    # SSL configuration

    listen 443 ssl http2 default_server;
    listen [::]:443 ssl http2 default_server;
    include snippets/ssl-example.com.conf;
    include snippets/ssl-params.conf;

    . . .
```

Save and close the file when you are finished.

**Step 4: Adjust the Firewall**
If you have the ufw firewall enabled, as recommended by the prerequisite guides, you'll need to adjust the settings to allow for SSL traffic. Luckily, Nginx registers a few profiles with ufw upon installation.

You can see the current setting by typing:
```sh
remote $ sudo ufw status
```

To additionally let in HTTPS traffic, we can allow the "Nginx Full" profile and then delete the redundant "Nginx HTTP" profile allowance:
```sh
remote $ sudo ufw allow 'Nginx Full'
remote $ sudo ufw delete allow 'Nginx HTTP'
```

**Step 5: Enabling the Changes in Nginx**
Now that we've made our changes and adjusted our firewall, we can restart Nginx to implement our new changes.

First, we should check to make sure that there are no syntax errors in our files. We can do this by typing:
```sh
remote $ sudo nginx -t
```

If everything is successful, you will get a result that looks like this:
```sh
Output
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If your output matches the above, your configuration file has no syntax errors. We can safely restart Nginx to implement our changes:
```sh
remote $ sudo systemctl restart nginx
```

The Let's Encrypt TLS/SSL certificate is now in place and the firewall now allows traffic to port 80 and 443. At this point, you should test that the TLS/SSL certificate works by visiting your domain via HTTPS in a web browser.

You can use the Qualys SSL Labs Report to see how your server configuration scores:

In a web browser:
https://www.ssllabs.com/ssltest/analyze.html?d=example.com

**Step 6: Set Up Auto Renewal**
Let's Encrypt's certificates are only valid for ninety days. This is to encourage users to automate their certificate renewal process. We'll need to set up a regularly run command to check for expiring certificates and renew them automatically.

To run the renewal check daily, we will use cron, a standard system service for running periodic jobs. We tell cron what to do by opening and editing a file called a crontab.
```sh
remote $ sudo crontab -e
```

Your text editor will open the default crontab which is a text file with some help text in it. Paste in the following line at the end of the file, then save and close it:
```sh
. . .
15 3 * * * /usr/bin/certbot renew --quiet --renew-hook "/bin/systemctl reload nginx"
```

The 15 3 * * * part of this line means "run the following command at 3:15 am, every day". You may choose any time.

The renew command for Certbot will check all certificates installed on the system and update any that are set to expire in less than thirty days. --quiet tells Certbot not to output information nor wait for user input. --renew-hook "/bin/systemctl reload nginx" will reload Nginx to pick up the new certificate files, but only if a renewal has actually happened.

cron will now run this command daily. All installed certificates will be automatically renewed and reloaded when they have thirty days or less before they expire.


Reference
https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04#prerequisites



