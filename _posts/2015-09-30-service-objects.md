---
layout: post
title: service objects
categories: [service objects, rails]
tags: [poro, rails, service objects]
published: True

---

There are many good reasons to use Service Objects. For example to encapsulate tasks which work on multiple models, does one specific task and doesn't really fit in as a class method on any one model which its works on. There are many articles talking about how/when to use them and there are so many ways of building a class and calling it a service object.

How do we decide what is a `well formed` service object and what should go in it? I mean anything form a simple PORO, to a subclass of some other complex class which has a `perform` method could be argued to be a service object. So what is the `convention` when using service objects.


### My opinion

1. Placement & Name
Put them in `/app/services/`
Update `application.rb` to load them `config.autoload_paths += ["#{config.root}/app/services}"]`
Name them like a verb/action, with a `*_service.rb` in the end. e.g. `analyze_user_points_service.rb`. It should be named like a method (`analyze_user_points_service.rb` and not like a Class (`UserPointsAnalyzerService`).

````
  class AnalayzeUserPointsService

  end
````

2. Return Value
Tells success & failure and is also a Data Transfer Object. I like to have one class named `ServiceResult`. The result here is immutable as the instances only have `attr_reader`.

````
class ServiceResult

  attr_reader :status, :message, :data, :errors

  def initialize(status:, message: nil, data: nil, errors: [])
    @status = status
    @message = message
    @data = data
    @errors = errors
  end

  def success?
    status == true
  end

  def failure?
    !success?
  end

  def has_data?
    data.present?
  end

  def has_errors?
    errors.present? && errors.length > 0
  end

  def to_s
    "#{success? ? 'Success!' : 'Failure!'} - #{message} - #{data}"
  end

end
````

3. Error
Errors should be caught and failure result should be returned.
In some rare cases like background jobs, we need to raise an error so the job can be retried or the error can be recorded.

````
def call
  # do whatever first
  return result
rescue => e
  Bugsnag.notify
  puts "got error #{e}"
  return result
end
````

4. Public Action Method
This is the one and only publically exposed method. I like calling it `call` as that's what Lambda's and proc's also take. I think `perform` is also an accetable name. My resque jobs have that name so it sits well with me.

````
def call
  # do all stuff here and call private methods from here
end
````

5. Factory Method
Having a `build` method makes a lot of things better. With `build` we are doing DI. This gives more flexibility and less coupling. The class'es new method is getting all dependecies built in the `build` and passed. The `new` just sets the instances and moves on. Testing is easier coz we can setup different dependecies and pass it in. The service is now immutable. Everytime build is called it creates a new instance.

````
def self.build(u_id)
  # here User is a dependecy managed here
  # also user is a dependency build and then injected
  user = User.find u_id
  self.new(user
end

def initialize(user)
  # see how user the dependency is being injected
  @user = user
end

````

Refernces

- http://multithreaded.stitchfix.com/blog/2015/06/02/anatomy-of-service-objects-in-rails/

- http://adamniedzielski.github.io/blog/2014/11/25/my-take-on-services-in-rails/

- http://solnic.eu/2013/12/17/the-world-needs-another-post-about-dependency-injection-in-ruby.html

- http://code.tutsplus.com/tutorials/service-objects-with-rails-using-aldous--cms-23689

- https://github.com/envato/aldous

- http://blog.rubybestpractices.com/posts/rklemme/017-Struct.html

- https://netguru.co/blog/service-objects-in-rails-will-helpTasks

- http://brewhouse.io/blog/2014/04/30/gourmet-service-objects.html





