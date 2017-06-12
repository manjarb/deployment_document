# Deploying a Rails App on Ubuntu with Capistrano, Nginx, and Puma

**Step 1 — Installing Nginx**
Following this link
https://github.com/manjarb/deployment_document/blob/master/Nginx-setup.md

**Step 2 — Installing Databases**
PostgreSQL
https://github.com/manjarb/deployment_document/blob/master/Install_and_Use_PostgreSQL_on_Ubuntu.md
https://github.com/manjarb/deployment_document/blob/master/Use%20PostgreSQL%20with%20Your%20Ruby%20on%20Rails%20Application%20on%20Ubuntu.md

**Step 3 — Installing RVM and Ruby**
Before installing RVM, you need to import the RVM GPG Key:
```sh
remote $ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
```

Then install RVM to manage our Rubies:
```sh
remote $ curl -sSL https://get.rvm.io | bash -s stable
remote $ source ~/.rvm/scripts/rvm
remote $ rvm requirements
remote $ rvm install 2.3.0(or current version)
remote $ rvm use 2.3.0(or current version) --default
```

**Step 4 — Installing Rails and Bundler**
```sh
remote $ gem install rails -V --no-ri --no-rdoc
remote $ gem install bundler -V --no-ri --no-rdoc
```

**Step 5 — Setting up SSH Keys**
```sh
remote $ ssh -T git@github.com
remote $ ssh -T git@bitbucket.org
```

Don't worry if you get a Permission denied (publickey) message. Now, generate a SSH key (a Public/Private Key Pair) for your server:
```sh
remote $ ssh-keygen -t rsa
```

Open a terminal on your local machine. If you don't have a SSH Key for your local computer, create one for it as well. In your local terminal session:
```sh
local $ ssh-keygen -t rsa
```

Add your local SSH Key to your Droplet's Authorized Keys file (remember to replace the port number with your customized port number):
```sh
local $ cat ~/.ssh/id_rsa.pub | ssh -p your_port_num deploy@your_server_ip 'cat >> ~/.ssh/authorized_keys'
```

**Step 6 — Adding Deployment Configurations in the Rails App**
On your local machine, create configuration files for Nginx and Capistrano in your Rails application. Start by adding these lines to the Gemfile in the Rails App:

```sh
Gemfile

group :development do
  ...

  gem 'capistrano',         require: false
  gem 'capistrano-rvm',     require: false
  gem 'capistrano-rails',   require: false
  gem 'capistrano-bundler', require: false
  gem 'capistrano3-puma',   require: false
end
```

Use bundler to install the gems you just specified in your Gemfile. Enter the following command to bundle your Rails app:
```sh
local $ bundle
local $ cap install
```

This will create:
- `Capfile` in the root directory of your Rails app
- `deploy.rb` file in the `config` directory
- `deploy` directory in the `config` directory

Replace the contents of your Capfile with the following:
```sh
# Load DSL and Setup Up Stages
require 'capistrano/setup'
require 'capistrano/deploy'

require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rvm'
require 'capistrano/puma'
install_plugin Capistrano::Puma, load_hooks: false  # Default puma tasks without hooks

require 'capistrano/scm/git'
install_plugin Capistrano::SCM::Git

# Loads custom tasks from `lib/capistrano/tasks' if you have any defined.
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
```


Replace the contents of config/deploy.rb with the following, updating fields marked in red with your app and Droplet parameters:
```sh
config/deploy.rb

# Change these
server 'your_server_ip', port: your_port_num, roles: [:web, :app, :db], primary: true

set :repo_url,        'git@example.com:username/appname.git'
set :application,     'appname'
set :user,            'deploy'
set :puma_threads,    [4, 16]
set :puma_workers,    0

# Don't change these unless you know what you're doing
set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/apps/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.error.log"
set :puma_error_log,  "#{release_path}/log/puma.access.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w(~/.ssh/id_rsa) }
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true  # Change to false when not using ActiveRecord

## set shared folder for upload images
set :linked_dirs, %w(public/uploads)

## Defaults:
# set :scm,           :git
# set :branch,        :master
# set :format,        :pretty
# set :log_level,     :debug
# set :keep_releases, 5

## Linked Files & Directories (Default None):
# set :linked_files, %w{config/database.yml}
# set :linked_dirs,  %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

namespace :puma do
  desc 'Create Directories for Puma Pids and Socket'
  task :make_dirs do
    on roles(:app) do
      execute "mkdir #{shared_path}/tmp/sockets -p"
      execute "mkdir #{shared_path}/tmp/pids -p"
    end
  end

  before :start, :make_dirs
end

namespace :deploy do
  desc "Make sure local git is in sync with remote."
  task :check_revision do
    on roles(:app) do
      unless `git rev-parse HEAD` == `git rev-parse origin/master`
        puts "WARNING: HEAD is not the same as origin/master"
        puts "Run `git push` to sync changes."
        exit
      end
    end
  end

  desc 'Initial Deploy'
  task :initial do
    on roles(:app) do
      before 'deploy:restart', 'puma:start'
      invoke 'deploy'
    end
  end

  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      invoke 'puma:restart'
    end
  end

  before :starting,     :check_revision
  after  :finishing,    :compile_assets
  after  :finishing,    :cleanup
  after  :finishing,    :restart
end

# ps aux | grep puma    # Get puma pid
# kill -s SIGUSR2 pid   # Restart puma
# kill -s SIGTERM pid   # Stop puma
```

Update `config/deploy/production.rb`
```sh
config/deploy/production.rb
server 'your_server_ip', port: 22, roles: [:web, :app, :db], primary: true
```

You can change all options depending on your requirements. Now, Nginx needs to be configured. Create config/nginx.conf in your Rails project directory, and add the following to it (again, replacing with your parameters):
```sh
config/nginx.conf

upstream puma {
  server unix:///home/deploy/apps/appname/shared/tmp/sockets/appname-puma.sock;
}

server {
  listen 80 default_server deferred;
  # server_name example.com;

  root /home/deploy/apps/appname/current/public;
  access_log /home/deploy/apps/appname/current/log/nginx.access.log;
  error_log /home/deploy/apps/appname/current/log/nginx.error.log info;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @puma;
  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}
```sh

**Step 7 — Deploying your Rails Application**
Again, from your local machine, make your first deployment:
```sh
local $ cap production deploy:initial
```

If everything goes smoothly, we're now ready to connect your Puma web server to the Nginx reverse proxy.

On the Droplet, Symlink the nginx.conf to the sites-enabled directory:

```sh
remote $ sudo rm /etc/nginx/sites-enabled/default
remote $ sudo ln -nfs "/home/deploy/apps/appname/current/config/nginx.conf" "/etc/nginx/sites-enabled/appname"
```

Restart the Nginx service:
```sh
remote $ sudo service nginx restart
```

**Normal Deployments**
Whenever you make changes to your app and want to deploy a new release to the server, commit the changes, push to your git remote like usual, and run the `deploy` command:
```sh
remote $ git add -A
remote $ git commit -m "Deploy Message"
remote $ git push origin master
remote $ cap production deploy
```sh

