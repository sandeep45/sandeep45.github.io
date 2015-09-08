---
layout: post
title: testing a model method
categories: [rails, rspec, testing, active_record]
tags: [rspec, mocks, stubs, dobule]
published: True
---

This is what we have:

````
  class Visitor < ActiveRecord::Base

    def process_for_events_called_agent_assigned
      visitor = self
      events = user.events.where(:action => Event::ACTION[:agent_assigned],:status => true)
      events.each { |event| event.process_on_visitor(self) }
      return true
    end

  end
````

We want to test this method: `process_for_events_called_agent_assigned`

What should be tested?

First, lets see what its doing. This is what comes to mind right away:

1. It always returns true
2. It iterates over every event which it gets from the user of the visitor
3. It calls process_on_visitor over every event which it gets from the user of the visitor
4. It only gets the events which have status ON and agent_assigned as action when it fetches these events

So how should we setup our test? We start of by following the `Given-When-Then` pattern but then go off it.

Lets first do the `Given` part:

````
  before(:each) do
    @u = FactoryGirl.create :user
    @v = FactoryGirl.create :visitor, :user_id => @u.user_id
    @e1 = FactoryGirl.create :event, :user_id => @u.user_id, :action => Event::ACTION[:agent_assigned]
    @e2 = FactoryGirl.create :event, :user_id => @u.user_id,:action => Event::ACTION[:agent_assigned]
    @e3 = FactoryGirl.create :event, :user_id => @u.user_id,:action => nil
  end
````

Now we have a user, who has a visitor and that user also has events of the kind we want.

Next lets do the `When` part:

````
  after(:each) do
    @result = @v.process_for_events_called_agent_assigned
  end
````


Now lets the do the `Then` part.

````
  it "always return true" do
    expect(@v.process_for_events_called_agent_assigned).to eq(true)
  end

  it "iterates over every event it gets from the user" do
    array = double.as_null_object
    allow(@v.user).to receive(:events).and_return(array)
    expect(array).to receive(:each)
  end

  it " calls process_on_visitor on every event it receives" do
    allow(@v.user).to receive(:events).and_return(double(:where => [@e1,@e2]))

    expect(@e1).to receive(:process_on_visitor)
    expect(@e2).to receive(:process_on_visitor)
  end

  it "filters on user events by status and action of agent assigned" do
    ar_double = double("active relation object").as_null_object

    expect(ar_double).to receive(:where).with(
      hash_including(
        :status => true,
        :action => Event::ACTION[:agent_assigned]
      )
    )

    allow(@v.user).to receive(:events).and_return(ar_double)
  end
````
