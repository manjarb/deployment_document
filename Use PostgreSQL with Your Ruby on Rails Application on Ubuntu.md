# Use PostgreSQL with Your Ruby on Rails Application on Ubunt

**Create Database User**
```sh
remote $ sudo -u postgres createuser -s user_name
```

If you want to set a password for the database user, enter the PostgreSQL console with this command:
```sh
remote $ sudo -u postgres psql
```

The PostgreSQL console is indicated by the postgres=# prompt. At the PostgreSQL prompt, enter this command to set the password for the database user that you created:
```sh
postgres=# \password user_name
exit
postgres=# \q
```

**Configure Database Connection**

Open your application's database configuration file in your favorite text editor.

config/database.yml excerpt
```sh
username: pguser
password: pguser_password
```

**Create Application Databases**
```sh
remote $ rake db:create
```


