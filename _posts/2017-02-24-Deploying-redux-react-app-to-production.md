---
title: Deploying Redux React App to Production
status: 
layout: post
categories: [Redux, React, React-router]
tags: [Redux, React, React-router]
published: True
---

I have a keen desire to host my React/Redux app with my rails app. Currently I have react app seperated hosts on a static server using github.io and I have my rails app seperate on amazon ec2. 

if i can copy the resources to the public folder, then nginx will serve my assets and it is setup to specify cache of 60 seconds

```
location ~*  \.(jpg|jpeg|png|gif|ico|css|js|ttf|ttc|otf|eot|woff|woff2|svg|map|html)$ {
          add_header "Access-Control-Allow-Credentials" "true";
          add_header "Access-Control-Allow-Origin" "*";
          add_header "Access-Control-Allow-Methods" "GET, POST, OPTIONS, PUT, DELETE";
          add_header "Access-Control-Allow-Headers" "x-requested-with";
          add_header Cache-Control "public";
          expires 60s;
        }
```

http://www.delight.consulting/blog/monolithic-rails-react-redux-application/

nginx is now rednering your files. this can be verified in your logs at /var/logs/nginx/access.log

Theis happens because your nginx is saying to do so in its conf at /etc/nginx/nginx.conf

root                    /var/deploy/abm_rails/web_head/current/public;

ad_unit.bundle.js
compiled webpack
require(ad_unit.css)
require(img/abc.png)
compiled abc.png -> dist/adsadsadsa.png
__webpack_public_path__ = `${CLOUDFRONT_URL}/`

add utf8 charset to your page
<meta charset="utf-8">

github.io was adding:
Content-Type:text/html; charset=utf-8

nginx gave:
Content-Type:text/html

so in my html i added
<meta charset="utf-8">

On my local machine I have nginx here: /usr/local/etc/nginx

