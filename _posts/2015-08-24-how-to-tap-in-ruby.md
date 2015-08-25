---
layout: post
title: how to tap in ruby
categories: [ruby]
tags: [ruby, tap]
published: True
---

Ruby has an interesting and useful method called tap. Its defined in `object.c` so available wherever you desire for it it be.

Its function is simple. When called upon an object with a proc as a paramter, it calls the passed in proc with the object as a parameter and then it returns the objcet.

````
VALUE
rb_obj_tap(VALUE obj)
{
    rb_yield(obj);
    return obj;
}
````

Here is one use case

````
User.first.tap { |u| puts u.id}.update_attributes :name => "x"
````

This will print the user id and then pass on the user object to the nex chained thing.

Here is another more useful example

````
User.first.tap { |u| u.name = "Sample Person" }.email_sample_report
````

Here we changed the name of the user for the purpose of sending out a sample report. the name gets changed on the object which was tapped, then that object gets passed on to the next chained method which emails a report which uses the name.

#### From the DOC

Yields x to the block, and then returns x.
The primary purpose of this method is to "tap into" a method chain,
in order to perform operations on intermediate results within the chain.




