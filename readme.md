# Rails deploy

Ansible provisioning to setup a pre-configured Ruby/Rails server that contains the following:

- Installs ruby via rbenv
- Installs node.js for asset pipeline
- Nginx + phusion passenger for serving your app
- PostgreSQL + Redis databases
- Setup some cron jobs to run rake tasks
- Sets up ufw and fail2ban for basic security
- Sets up swapon for when you run out of memory
- Checkout the [changelog](CHANGELOG.md) for recent additions

## Prerequisites

- [Install ansible](http://docs.ansible.com/intro_installation.html) on your system
- Setup a Ubuntu 14.04 VPS server. Try a [digital ocean](https://digitalocean.com) 1gb droplet (would recommend at least 1GB memory).
- SSH into the server as root and ensure that is working from your local machine.

## Usage

For each server you want to setup clone this repo. This allows you to keep the configuration around and deploy again at any time, or deploy multiple servers for some horizontal scailing (in this instance you will need a shared database server).

```sh
# Copy the example vars file over to vars/all.yml
$ cp vars/example.yml vars/all.yml

# Copy the example hosts file
$ cp hosts.example hosts
```

Edit `./vars/all.yml` to meet your requirements. Replace the 127.0.0.1 IP address in `./hosts` with your servers IP

```sh
# Run the initial boostrap for your server (sets up deploy user etc)
$ ansible-playbook -i hosts bootstrap.yml --user root --ask-pass

# Install all the things
$ ansible-playbook -i hosts rails_deploy.yml --user deploy
```

### Allow server to access source code

First up you’ll need to allow your server access to wherever your source code is hosted, get the public SSH key generated on your serve by running a command like:

```sh
# Remember to substitue your servers IP (and the deploy username if you altered it)
$ ssh deploy@127.0.0.1 -t cat /home/deploy/.ssh/id_rsa.pub
```

Next you’ll want to add this key to your ssh keys on github (or bitbucket or whatever git host you use)

### Deploying your code

This is designed to work out of the box with capistrano for deploys. You’ll want to setup capistrano like so:

#### Gemfile

Add the following gems to your Gemfile and run `bundle`.

```ruby
gem 'dotenv-rails', '~> 1.0.2'

group :development do
  gem 'capistrano',  '~> 3.1'
  gem 'capistrano-rails', '~> 1.1'
  gem 'capistrano-rbenv', '~> 2.0'
end
```

Once installed run `cap install` to create capistrano configuration files and then edit each file below.

#### ./Capfile

Next up make sure the following lines are uncommented from your `Capfile`.

```ruby
require 'capistrano/rbenv'
require 'capistrano/bundler'
require 'capistrano/rails/assets'
require 'capistrano/rails/migrations'
```

#### ./config/deploy.rb

```ruby
# config valid only for Capistrano 3.1
lock '3.2.1'

set :application, 'APP_NAME'
set :deploy_to, '/var/www/app' # Change this to match your deploy_dir in vars/app.yml
set :repo_url, 'YOUR_GITHUB_URL'
set :keep_releases, 3
set :log_level, :info
set :linked_files, %w{.env}
set :migration_role, :app
set :rbenv_type, :user
set :rbenv_ruby, '2.1.5'

namespace :deploy do
  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 3 do
      execute :touch, release_path.join('tmp/restart.txt')
    end
  end

  after :publishing, :restart
end

namespace :rails do
  desc 'Open the rails console on the primary remote server'
  task :console do
    on roles(:app), primary: true do |host|
      command = "cd #{deploy_to}/current && /home/#{host.user}/.rbenv/shims/bundle exec rails console #{fetch(:stage)}"
      exec "ssh -l #{host.user} #{host.hostname} -p #{host.port || 22} -t 'cd #{deploy_to}/current && #{command}'"
    end
  end
end
```

#### ./config/deploy/production.rb

Replace `127.0.0.1` with your servers IP address and update the user if you changed it from the defaults.

```ruby
server '127.0.0.1', user: 'deploy', roles: %w{web app}
```

## TODO

- SSL Support
- Common exports like EDITOR and RAILS_ENV for running rake tasks etc
