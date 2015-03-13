---
layout: post
title: '[OP] Deploy Production Server Using Travis-CI, Figaro and Capistrano'
---

Recently I've successfully made a try to deploy production server of [MAGI Systems](https://magi.systems) with [Travis CI](http://travis-ci.org).

I would like to share this adventure.

As I mentioned before, I'm using `Figaro` to manage variables, `Capistrano` to deploy, and `Travis CI` for testing. All I have to do is combine them together.

### Make `Capfile` loading `Figaro` variables

`Figaro` can read `config/application.yml`, and load proper variables to `ENV`.

`Figaro` will be automatically loaded during booting `Rails`, and `Figaro` relies on `Rails` environment.

So, We have to load `Rails` in `Capfile`.

Added a line in `Capfile`.

```ruby
require 'capistrano/setup'
require 'capistrano/deploy'
require 'capistrano/rails'
require 'capistrano/rvm'
require 'capistrano/thin'
require 'capistrano/sidekiq'

# This line load Rails environment, so we have Figaro
require File.expand_path('../config/environment',  __FILE__)

```

Some how, load `Rails` environment before `require 'capistrano/setup'` will cause error.

### Write production server informations in `application.yml`

This step is optional, but is necessary if you want to deploy server locally using `Capistrao`.

Write informations of production server in `application.yml` (git ignored), so `Figaro` will load them, and we can deploy production server locally.

But you should **NOT** write them in `application.yml.sample` (git included), not even a empty string.

We will use `Travis CI`'s env variables instead of `application.yml` later.

```yaml
# Production Server (Used by Capistrano, commented out for Travis-CI variables)
# PROD_SERVER: ''
# PROD_SERVER_USER: ''
# PROD_SERVER_PASSWORD: ''
```

### Make `Capistrano` link `application.yml`

This step should not be included in this post, but It's necessary if you want to use `Figaro` with `Capistrano`.

Add a rake file in `lib/capistrano/tasks`

```ruby
# lib/capistrano/tasks/figaro.rb
namespace :figaro do
  desc "SCP transfer figaro configuration to the shared folder"
  task :setup do
    on roles(:app) do
      upload! "config/application.yml", "#{shared_path}/application.yml", via: :scp
    end
  end

  desc "Symlink application.yml to the release path"
  task :symlink do
    on roles(:app) do
      execute "ln -sf #{shared_path}/application.yml #{release_path}/config/application.yml"
    end
  end
end
after 'deploy:updating', 'figaro:symlink'
```

This will add 'figaro:setup' to `Capistrano`, which upload local `application.yml` to shared folder of remote server, and makes `Capistrano` link `application.yml` during each release.

### Make `config/deploy/production.rb` register server using `ENV`

Edit `config/deploy/production.rb` like this

```ruby
if ENV['PROD_SERVER'].present?
  server ENV['PROD_SERVER'], user: ENV['PROD_SERVER_USER'], roles: %w{web worker db app}, auth_methods: %w{ password }, password: ENV['PROD_SERVER_PASSWORD']
end
```

This makes `Capistrano` register server from `ENV`.

### Configure `Travis-CI` with server informations

Like many other platforms, `Travis-CI` can specify env virables manually.

So we can store production server informations in env, not in `application.yml`.

![Config Travis-CI](/assets/images/config-travis-ci1.png)
![Config Travis-CI](/assets/images/config-travis-ci2.png)

### Invoke `cap production deploy` in `.travis.yml`

Last thing we need to do is to invoke `cap production deploy` if our build succeed.

This is my `.travis.yml`.

Before script, `application.yml.sample` will be copied to `application.yml`.

After success, invoke 'cap production deploy', `Sidekiq` will be automatically restarted during this process.

I'm using `capistrao-thin`, so I have to invoke `deploy:restart`

```yaml
language: ruby
services:
  - redis-server
rvm:
  - 2.1.0
branches:
  only:
    - master
before_script:
  - cp config/application.yml.sample config/application.yml
  - RAILS_ENV=test bundle exec rake db:drop
  - RAILS_ENV=test bundle exec rake db:create
  - RAILS_ENV=test bundle exec rake db:schema:load
  - RAILS_ENV=test bundle exec rake db:seed
script:
  - RAILS_ENV=test bundle exec rspec spec
after_success:
  - bundle exec cap production deploy
  - bundle exec cap production deploy:restart
```

### Done

The hornor belongs to Travis-CI.

Source can be found at [https://github.com/yanke-guo/magi-systems](https://github.com/yanke-guo/magi-systems)
