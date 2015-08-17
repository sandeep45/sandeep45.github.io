---
layout: post
title: Testing Webhooks
categories: [Rails, Testing]
tags: [webhook, teting]
published: True
---

In our app, we receive a bunch of data from other Marketing Automation(MA) systems. They send us data by calling our webhook. As we add more and more MA systems, we need a good way to tests these webhooks. A webhook is similar to an API call, with the difference that in a webhook somebody else is calling us and giving us data whereas in an API we call someone else and get data from them.

I want to test that my webhook's are capable of:
1. Parsing the data sent by the external system
2. Is able to use that data and update the database

I started by doing a functional test in rspec. I stored the paramters normally sent by a webhook in a hash called `valid_attrbutes`. T I also created another hash called `invalid_attributes` where i stored the parameters incorrectly.

Then i called my webhook with these parameters and tested that my code was handling the data correctly. Below is an example

````
    context "#update_visitor" do
      let(:valid_attributes) do
        {
          :user_id => @u1.id,
          :visitor_id => @v1.id,
          :name => "YO YO Honey Singh"
        }
      let(:invalid_attributes) do
        {
          :jiberish => "more jiberish"
        }
      end

      it "updates visitor's name" do
        get :update_visitor, valid_attributes
        expect(@v1.name).to eq("YO YO Honey Singh")
      end

      it "gives 406 with invalid attributes" do
        get :update_visitor, valid_attributes
        expect(response.code).to eq("406")
      end
````

This works but quickly goes out of hand as you start making a bunch of hashes each to represent a different scenerio. Also the example above was an extremly simplified request. In real world the parameters are much bigger and are also accompanied with header values. The straw which broke the camel's back in our case was salesforce. It posts a gigantic XML file with batched data. The file is just huge and has hundred's of attributes. Converting that XML document to a hash and storing just didn't seem like something we wanted to do. So we move on to find a better solution.

Before, I share my hack/solution to manage testing webhooks, I would like to mention some handy tools

1. requestbin: gives you a url which external systems can hit. it records all the parameters, headers etc. which then you can inspect and learn from.

2. runscope: its a cloud testing suite. it allows you to specify your webhook as an api, then enter the request data you want to send it and then setup assertions. Not to forget, it also does sequnces and more complex sets of tests with dynamic variables. I didn't explore this much as I am more intersted in doing my tests using rspec.

3. ngrok: gives you a publically accesible url for your local server. Yes it sounds to good to be true, but it works. And yes your local server could be burried behind firwalls and routers and whatever. This thing will make it accessible on the ouside so other services can call your public url.

4. Postbin Gem: Allows to setup a local post explorer. Gives you a url and all posts made to that url are stored and diplayed in the UI. https://github.com/lantins/postbin

5. Postman: Its a chrome extension to make API calls easily and accurately and then explore the response.

#### Using fixtures to test webhooks

First, I updated the webhook address at my external MA to my mahcine: `http://listenloop.ngrok.com/users/test`

In the `test` method I just print it all the headers and parameters as seen by the rails controller. Here is how I did that:

````
  def test
    result = {}

    headers = request.env.select { |k,v| k.start_with? 'HTTP_' }.
      reject { |k,v| k == "HTTP_COOKIE" }
    result["headers"] = headers

    parameters = params.reject { |param| ["action", "controller"].include? param  }
    parameters_in_json_string = parameters.to_json
    parameters_in_json = JSON.parse(parameters_in_json_string)
    result["parameters"] = parameters_in_json

    puts JSON.pretty_generate(result)

    render :json => result
  end
````

The output of calling this `test` method looks like this:

````
{
  "headers": {
    "HTTP_HOST": "localhost:3000",
    "HTTP_CONNECTION": "keep-alive",
    "HTTP_ACCEPT": "application/json",
    "HTTP_CACHE_CONTROL": "no-cache",
    "HTTP_SANDEEP": "arneja12345",
    "HTTP_USER_AGENT": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.155 Safari/537.36",
    "HTTP_ACCEPT_ENCODING": "gzip, deflate",
    "HTTP_ACCEPT_LANGUAGE": "en-US,en;q=0.8",
    "HTTP_VERSION": "HTTP/1.1"
  },
  "parameters": {
    "user_id" => 123,
    "visitor_id" => 1,
    "name" => "YO YO Honey Singh"
  }
}
````

At this point we have the headers and parameters as seen by rails for an incoming webhook pritned in our console. I will then copy paste it into a `.json` file under my fixtures folder.

#### Using Rspec to test webhooks with fixtures

Now I am going to re-write my initial rspec test. This time instead of usign params from hashes, I will be loading them from the fixture file.

````
    context "#update_visitor" do
      it "updates visitor's name" do
        fixture_path = "#{Rails.root}/spec/fixtures/salesforce_webhooks/lead.json"
        lead_json = JSON.parse(File.read(fixture_path))
        request.env.merge! lead_json["headers"]
        get :update_visitor, lead_json["parameters"]
        expect(@v1.name).to eq("YO YO Honey Singh")
      end

      it "gives 406 with invalid attributes" do
        get :update_visitor, "#{Rails.root}/spec/fixtures/salesforce_webhooks/jiberish.json"
        expect(response.code).to eq("406")
      end
````

I find this way of storing entire webhook params in a file much more manageable, than keeping them in a hash inside the spec file.