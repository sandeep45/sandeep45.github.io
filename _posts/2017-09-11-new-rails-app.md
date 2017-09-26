---
title: 10 easy steps Guide for an experienced developer to create a Rails 5.0 API only app
status: 
layout: post
categories: [Redux, React, React-router]
tags: [Redux, React, React-router]
published: True
---

#### Step 1

Install all updates which are pending on the Mac

#### Step 2

Update your xcode and agree to all the TOS things - `xcode-select --install`

#### Step 3

Create your project directory - `mkdir tweets` and cd to it - `cd tweets`

#### Step 4

Add files to lockdown which version of ruby and which gemset you will be using.
 
##### File 1`: `.ruby-gemset`
 
 ```
 tweet_rails514
 ```
 
##### File 2: `.ruby-version`
```
ruby-2.4.1
```

#### Step 5

install ruby - `rvm install ruby 2.4.1`

#### Step 6

go in and out the directory to rvm ruby version and gemset lock can kick in
 
```
cd .. && cd tweets
rvm current
```

#### Step 7

install the pre-requisites

```
gem install bundler
gem install nokogiri
gem install rails  --version=5.1.4
```

**Note**: here I am installing the latest version of ruby. If installation of nokogiri fails then refer to this guide: http://www.nokogiri.org/tutorials/installing_nokogiri.html#mac_os_x 

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

#### Step 10

run the rails server and make a hit to get the tweets
```
rails s
http://localhost:3000/tweets
```

## hirb

**coming soon**


#### Reference

- http://railsapps.github.io/installrubyonrails-ubuntu.html
- http://guides.rubyonrails.org/getting_started.html
