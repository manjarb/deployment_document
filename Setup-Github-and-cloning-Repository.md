# Setup Github and cloning Repository

Login to the server
```sh
local $ ssh user@your_server_ip
```

**Install Git with Apt**
```sh
remote $ sudo apt-get update
remote $ sudo apt-get install git
```

**Set Up Git**
```sh
remote $ git config --global user.name "Your Name"
remote $ git config --global user.email "youremail@domain.com"
```

**Add Public key to the repo**
```sh
remote $ ssh-keygen -t rsa
remote $ cat ~/.ssh/id_rsa.pub
```

Then copy public key to deployment key on the repo

Reload ssh
```sh
remote $ sudo systemctl reload sshd
```

**Clone The repo**
```sh
remote $ sudo mkdir /www
remote $ sudo chown -R deploy:deploy /www
remote $ cd /www
remote $ git clone git@github.com:username/repo_name.git
```

### Reference

| Title | Url |
| ------ | ------ |
| How To Install Git on Ubuntu 16.04 | [https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-16-04] [PlDb] |
