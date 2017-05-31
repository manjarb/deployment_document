# Initial Server Setup with Ubuntu

**Root Login**
```sh
local $ ssh root@your_server_ip
```

**Create a New User**
```sh
remote $ adduser deploy
```

**Root Privilegesr**
```sh
remote $ usermod -aG sudo deploy
```

**Allow Password authentication**
```sh
remote $ sudo nano /etc/ssh/sshd_config
```

**Change this line (temporary)**
inside /etc/ssh/sshd_config
```sh
PasswordAuthentication yes
```

**Reload ssh**
```sh
remote $ sudo systemctl reload sshd
```

**Copy Local Public Key to remote server**
```sh
local $ ssh-copy-id deploy@your_server_ip
```

**Disable Password authentication**
```sh
remote $ sudo nano /etc/ssh/sshd_config
```

**Change this line**
```sh
inside /etc/ssh/sshd_config
PasswordAuthentication no
```

**Reload ssh**
```sh
remote $ sudo systemctl reload sshd
```

**Test Log In**
```sh
local $ ssh deploy@your_server_ip
```

**Disbale root login**
```sh
local$ ssh root@your_server_ip
remote $ sudo nano /etc/ssh/sshd_config
PermitRootLogin no
remote $ sudo systemctl reload sshd
remote $ exit
```

**Set Up a Basic Firewall**
```sh
local$ ssh deploy@your_server_ip
remote $ sudo ufw app list
remote $ sudo ufw allow OpenSSH
remote $ sudo ufw enable
remote $ sudo ufw allow ssh
remote $ sudo ufw allow 80 (for HTTP server)
remote $ sudo ufw allow 443 (for SSL server)
remote $ sudo ufw enable
```

**Configure Timezones**
```sh
remote $ sudo dpkg-reconfigure tzdata
You will be presented with a menu system that allows you to select the geographic region of your server:
After selecting an area, you will have the ability to choose the specific time zone that is appropriate for your server:
```

**Configure NTP(Network Time Protocol) Synchronization**
```sh
remote $ sudo apt-get update
remote $ sudo apt-get install ntp
```

### Reference

| Title | Url |
| ------ | ------ |
| Initial Server Setup with Ubuntu 16.04 | [https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04] [PlDb] |
| Common Firewall Rules and Commands | [https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands] [PlDb] |
