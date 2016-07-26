---
title: Token Based Authentication
layout: post
categories: [Rails, Authentication]
tags: [Rails, Authentication]
published: True
---

## My Stack

I am using rails 3.2, Devise, simple_token_authentication. Although my implementation is specific to this stack, the concept is generic and I attempt to explain token based authentication regardless of stack.

## Little Background

In a traditional app upon login username & password go to the server. There they are checked. If things pass a session is created and a cookie with the session identifier is returned.

Now when a person visits a page which requires authentication the cookie is automatically sent up by the browser and therefore the person is authenticated and is able to see the page. When the user clicks logout which makes a request to the server, the session is deleted and the cookie can no longer get the user access. After this point the person is no longer able to see any page which requires authentication.

This system of authentication depends upon cookies which it's prone to CSRF attacks. Also management of the cookies is done by the browser and not necessarily the client app. It's the browser which is sending the cooking up and down & maintaining the session with the domain which issued the cookie.

## Why token based authentication

In a client side app we do not have access to the authenticity token which means we have to disable CSRF protection and therefore if we use the cookie based authentication our system is insecure and vulnerable to CSRF attacks.

## what is token based authentication

In token based authentication on Login the client sends the username and password to the server and in return receives a token instead of a cookie. now the client site app Will send token with every request it makes to authenticate it self. No cookie was ever received and no cookies are being sent up-and-down with requests. No cookies mean no CSRF attacks.

## Setting up token based authentication

### The first step is to disable sending of cookies when a user logs in. I am doing this by using the rack middleware which simply deletes the session cookie from being sent down.

Write the middlware file

````
# lib/cookie_filter.rb
class CookieFilter
  def initialize(app)
    @app = app
  end

  def call(env)
    status, headers, body = @app.call(env)

    # use only one of the next two lines

    # this will remove ALL cookies from the response
    # headers.delete 'Set-Cookie'

    # this will remove just your session cookie
    Rack::Utils.delete_cookie_header!(headers, '_meme_backend_session')

    [status, headers, body]
  end
end
````

Update the application.rb to load the `/lib` directory

````
    config.autoload_paths += %W(#{config.root}/lib)
````

Set up the app using middleware and silent it in stack trace by adding the following lines to any initializer.

````
Rails.application.config.middleware.insert_before ::ActionDispatch::Cookies, ::CookieFilter
Rails.backtrace_cleaner.add_silencer { |line| line.start_with? "lib/cookie_filter.rb" }
````

You may also need to load `lib` directory as part of your rails app if you are not automatically already loading it.

````
    # application.rb
    config.autoload_paths += %W(#{config.root}/lib)
````

I also disable CSRF token protection in my application controller as it's no longer needed since I have already disabled cookies. Do this by commenting to protect from forgery line in application_controller.rb

````
class ApplicationController < ActionController::Base
  # protect_from_forgery
end
````

There are other ways also to delete / disable this cookie. For reference look at the stack overflow question - http://stackoverflow.com/questions/5435494/rails-3-disabling-session-cookies


### The second step is to enable you're authentication System to accept json requests. I did this by adding `respond_to :json`
to the Sessions controller and registrations controller.

````
class RegistrationsController < Devise::RegistrationsController
    respond_to :json
end
````

````
class SessionsController < Devise::SessionsController
  respond_to :json
end
````

Then I updated my routes to use these new controllers when doing registration and session handling.

````
  devise_for :users, :controllers => {:registrations => "registrations", sessions: "sessions"}
````

We can now check our set up by making a request to the login page and ensuring that we get back JSON with users record in it.

Endpoint: http://localhost:3000/users/sign_in

Request Header:

````
Accept: application/json
````

Response Body:
````
{
    "authentication_token": "WAGzPbE49sh9cSD21mea",
    "created_at": "2016-07-23T18:41:43Z",
    "email": "sandeep45@gmail.com",
    "id": 1,
    "updated_at": "2016-07-24T20:08:05Z"
}
````

To learn more about logging in viq Ajax requests you can refer to Andrew's blog - http://blog.andrewray.me/how-to-set-up-devise-ajax-authentication-with-rails-4-0/. This is where I learned how to set up devise for ajax requests.

### the third step is to require token on pages which require authentication.

I am doing this by using a gem called `simple_token_authentication`. https://github.com/gonzalo-bulnes/simple_token_authentication

In my application controller I add `acts_as_token_authentication_handler_for User`. this makes all controllers which extends from application controller to require user token whenever any action of the controllers is accessed.

Also in my User model I add `acts_as_token_authenticatable` and create a `auth_token` field. This gem Will automatically populate token field with a securely generated token. This is the token which will be used for the user to get access.

We can now test this by visiting a controller action. We will get a 401 error because all actions require authentication. Then we can visit the same page again but this time with the token and we should get the page's content back.

Endpoint: http://localhost:3000/phone_numbers

Request Header:

````
Accept: application/json
````

Response Body:
````
{
    "error": "You need to sign in or sign up before continuing."
}
````

Endpoint: http://localhost:3000/phone_numbers

Request Header:

````
Accept: application/json
X-User-Email: sandeep45@gmail.com
X-User-Token: WAGzPbE49sh9cSD21mea
````

Response Body:
````
[
  {
    id: 1, number: 1
  }
]
````

### the fourth step is save that token upon login or user creation and use it on all API requests.

I make my ajax call with axios and then add update default headers.

````
export const doUserSignIn = (email, password) => {
  return axios.post(`/users/sign_in`, {
      user:{
        email,
        password
      }
    }, {
      headers: {
          Accept: "application/json"
        }
    }
  );
}
````

````
export const addAuthTokenInAjax = (email, authentication_token) => {
  axios.defaults.headers.common['X-User-Email'] = email;
  axios.defaults.headers.common['X-User-Token'] = authentication_token;
}
````

### Thoughts

- change token upon logout
- expire token after some time
