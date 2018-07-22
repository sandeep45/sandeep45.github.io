---
title: 10 easy steps Guide for an experienced developer to create a Rails 5.0 API only app
status:
layout: post
categories: [Redux, React, React-router]
tags: [Redux, React, React-router]
published: True
---

#### Step 1 - Housekeeping

- Install all updates which are pending on the Mac
- Update your xcode and agree to all the TOS things - `xcode-select --install`
- Update Brew `brew update`

#### Step 3

Create your project directory - `mkdir tweets` and cd to it - `cd tweets`

#### Step 4

Add files to lock-down which version of ruby and which gemset you will be using.

Get latest version here:

- https://rubygems.org/search?query=rails
- https://www.ruby-lang.org/en/downloads/

##### File 1 - `.ruby-gemset`

 ```
 echo "tweet_rails514" > .ruby-gemset
 ```

##### File 2 - `.ruby-version`
```
echo "ruby-2.4.1" > .ruby-version
```

#### Step 5

install ruby - `rvm install ruby 2.4.1`

#### Step 6

go in and out the directory so rvm `ruby-version` and `ruby-gemset` lock can kick in

```
cd .. && cd tweets
rvm current
```

#### Step 7 - install pre-requisites

install the following gems. Once we have these gems then we can use them to get the rest of the gems and manage all gems from bundler & gemfile.

```
gem install bundler
gem install nokogiri
gem install rails  --version=5.1.4
```

**Note**:
- here I am installing the latest version of rails.
- If installation of nokogiri fails then refer to this guide: http://www.nokogiri.org/tutorials/installing_nokogiri.html#mac_os_x

#### Step 8

create the rails app, follow it up by installation of all the dependent gems

```
cd ~/code/tweets
rails _5.1.4_ new . --database=postgresql --api -f
bundle install
```

**Note**: here since I am in the `tweets` directory I am using a `.` as where I want the Rails App to be built.

#### Step 9

create the databases, scaffold and migrations.

```
rake db:setup
rails g scaffold Tweet username:string message:text the_date:datetime
rake db:migrate
```

**Note**: use an app like `Postico` to verify that the databases were created.

#### Step 10

run the rails server and make a hit to get the tweets
```
rails s
http://localhost:3000
http://localhost:3000/tweets
```

#### Step 11

Add `Hirb` - https://github.com/cldwalker/hirb

**coming soon**

#### Step 12

Setting up Git

**coming soon**

#### Step 13 - Enable Cross Site Requests

##### Option 1
```
gem 'rack-cors', :require => 'rack/cors'
```

Then update `application.rb` to allow cross site requests

```
config.middleware.insert_before 0, Rack::Cors do
      allow do
        origins '*'
        resource '*', :headers => :any, :methods => :any
      end
    end
 ```
 
Check that `rack-cors` is on the top of the middleware stack - `bundle exec rake middleware`

do not send `credentials` header as part if the request from the front end as it prevents doing `*` for `origins` in the response.

```
// axios.defaults.withCredentials = true; // do not add this line.
```

Read the gotchas on the [github readme](https://github.com/cyu/rack-cors)

##### Option 2

add the CORS headers in nginx.

```
location / {
  add_header "Access-Control-Allow-Credentials" "true";
  add_header "Access-Control-Allow-Origin" "*";
  add_header "Access-Control-Allow-Methods" "GET, POST, OPTIONS, PUT, DELETE";
  add_header "Access-Control-Allow-Headers" "x-requested-with";
  add_header Cache-Control "public";
  expires 1200s;
} 
```
 



Using postman to start off the API's

Adding Devise
https://www.valentinog.com/blog/devise-token-auth-rails-api/

add database columns to model comments

adding testing framework

Jbuilder or AMS

rack-cors

Rack::Attack

email

sms

mailcatcher

gem 'annotate_models'
rails g annotate:install
skip_on_db_migrate=true|false
gem 'awesome_print'

nvm
https://github.com/creationix/nvm


#### Reference

- http://railsapps.github.io/installrubyonrails-ubuntu.html
- http://guides.rubyonrails.org/getting_started.html
