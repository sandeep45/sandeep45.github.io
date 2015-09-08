---
layout: post
title: Mocking in rspec
categories: [testing, rspec, rails]
tags: [mocking, stubbing, spies]
published: True
---

With code, mocking is quite straight forward in Rails.

for e.g. We have a `@user` object and we want some method `foo` on it to always return `5`

````
  expect(@user).to receive(:foo).and_return(5)
````

This is a partial double. We have a `user` object and its partially mocked. every other method, attribute etc. in it works like normal. This means:

````
  puts @user.id # returns the id of the user
````

Partial Doubles are different from instance doubles, class doubles and object doubles. These 3 methods create an empty double object. They do no have any original functionality and can not call original. What they do have is that they do not allow mocking of things which do not exist on the item whose template they were created from. For e.g.

````
  user = instance_double("User")
  expect(user).to receive(:non_existant_method).and_return(5)
  # throws RSpec::Mocks::MockExpectationError: User does not implement: non_existant_method
````

This is because user does not have the method called `non_existant_method`

Difference between `instance_double` and `class_double` is that one uses the instance as the template and the other uses the class as the template. They are both built by passing in the name of the Class like:

````
u1 = instance_double("User")
u2 = class_double("User")
````

The `object_double` on the other hand takes in an object as parameter and uses that as the template. For e.g.

````
u = User.first
o1 = object_double(u)
o2 = object_double(User)
````

Object doubles have a unique advantage over instance_doubles because they have access to method_missing and dynamically created methods from it. So those are part of the skelton and can therefore be stubbed.

We also have just `double`. These create a strict object which does not follow any template and allows anything expecified and rejects the rest. They can be made loose too by specigying `as_null_object` on them. This makes them return self on anything not specified. For e.g.

````
 dbl = double
 allow(dbl).to receive(:foo).and_return "5"
 dbl.foo #5
 dbl.sandeep #error

 dbl1 = double("yo yo double", :say => "yo")
 dbl1.say #yo
 dbl1.sandeep #error


 dbl2 = double("foo" :name => "foo").as_null_object
 dbl2.name #foo
 dbl2.sandeep # dbl2
````



So far `partial_double` is my favorite. It allows to keep the sanity of the object and at the same time stub out things.
My second favorite is just a `loose double`. Its very verstile. I can built it with no effort, it doesn't check or complain on anything. I can use it pass in wherever an object is expected and then check for things happening in on that object. I can also use it completly remove dependecies.

For e.g. Lets say we need a user object which has 10 visitors with an average height of 6' 0''. It has a method called `average_visitor_height` and that should return 6.

````
 u = double("A cool usser", :average_height => 6).as_null_object
````

A common pattern used when writing tests is: Given-when-then AKA Act-Arrange-Assert. For E.g.

````
it "returns number of children when asked for size" do

  # Given
  u = FactoryGirl.create :user
  child1 = FactoryGirl.create :child, :user_id => u.id
  child2 = FactoryGirl.create :child, :user_id => u.id

  # When
  num_of_children = u.children.size

  # Then
  expect(num_of_children).to eq(2)
end
````

This pattern breaks when checking for something to have been called. For e.g.

````
it "calls send_email after_building csv file" do

  # Given
  u = FactoryGirl.create :user

  # Expect
  expect(u).to receive(:send_email).at_least(:once)

  # Then
  u.build_csv

end
````

Note that in the above example we could not use the Given-When-Then pattern. We are setting up an expectation of something before we have even did the thing which make that something happen.

To assist in this we have what we call `spies`. Spies allow us to monitor an object and then later check for message by using methods like `have_received`. For e.g.

````
it "calls send_email after_building csv file" do

  # Given
  u = FactoryGirl.create :user
  u1 = spy(u)

  # Then
  u1.build_csv

  # Expect
  expect(u1).to have_received(:send_email).at_least(:once)

end
````

Spying also works on partial_doubles. Here is an example:

````
it "calls send_email after_building csv file" do

  # Given
  u = FactoryGirl.create :user
  allow(u).to receive(:send_email)

  # Then
  u1.build_csv

  # Expect
  expect(u1).to have_received(:send_email).at_least(:once)

end
````







