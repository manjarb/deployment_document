#Install and Use PostgreSQL on Ubuntu

**Installation**
```sh
local $ ssh user@your_server_ip
remote $ sudo apt-get update
remote $ sudo apt-get install postgresql postgresql-contrib
```

**Switching Over to the postgres Account**
```sh
remote $ sudo -i -u postgres
remote $ psql
```

Exit out of the PostgreSQL prompt by typing:
```sh
postgres=# \q
```

**Accessing a Postgres Prompt Without Switching Accounts**
```sh
remote $ sudo -u postgres psql
```
exit the interactive Postgres session by typing:
```sh
postgres=# \q
```

**Create a New Role**
If you are logged in as the postgres account, you can create a new user by typing:
```sh
postgres@server:~$ createuser --interactive
```

If, instead, you prefer to use `sudo` for each command without switching from your normal account, you can type:
```sh
remote $ sudo -u postgres createuser --interactive
```

The script will prompt you with some choices and, based on your responses, execute the correct Postgres commands to create a user to your specifications.
```sh
Enter name of role to add: role_name
Shall the new role be a superuser? (y/n) y
```

**Create a New Database**

If you are logged in as the postgres account, you would type something like:
```sh
postgres@server:~$ createdb db_name
```

If, instead, you prefer to use sudo for each command without switching from your normal account, you would type:
```sh
remote $ sudo -u postgres createdb db_name
```

**Open a Postgres Prompt with the New Role**

If you don't have a matching Linux user available, you can create one with the adduser command. You will have to do this from an account with sudo privileges (not logged in as the postgres user):
```sh
remote $ sudo adduser user_name
```

Once you have the appropriate account available, you can either switch over and connect to the database by typing:
```sh
remote $ sudo -i -u user_name
remote $ psql
```
Or, you can do this inline:
```sh
remote $ sudo -u user_name psql
```

If you want your user to connect to a different database, you can do so by specifying the database like this:
```sh
remote $ psql -d postgres
```

Once logged in, you can get check your current connection information by typing:
```sh
sammy=# \conninfo
```


