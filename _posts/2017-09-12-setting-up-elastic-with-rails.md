---
title: Getting started Elasticsearch guide, written for an experienced Developer
layout: post
categories: [rails, elasticsearch]
tags: [rails, elasticsearch]
published: True
---

## Fair warning

If you are thinking about using Elasticsearch (ES), thing again. **Do you really need it?** 
If all you want to do is Full-Text search, then there are other 
- simpler, 
- cheaper, 
- easier to setup, 
- easier to use & 
- easier to maintain solutions.
 
Using ES just to do a text search on a table with a few million records is an **over-kill** solution. The overhead it adds **is not worth it**, especially if you are 
- using ES just for this purpose only and 
- you are new to ES.

It may serve you well to look at e.g. Postgres supports Full Text Search https://www.postgresql.org/docs/9.5/static/textsearch.html

With that said, lets get our hands dirty with ES.

## Install (ES)

```
sudo chown -R $(whoami) /usr/local
brew update
brew cask install java
brew install elasticsearch
brew services start elasticsearch
brew services list
```

Test that elastic is up and responding - http://localhost:9200/

Installation is done here:

```
/usr/local/etc/elasticsearch
/usr/local/opt/elasticsearch/libexec

/usr/local/opt/elasticsearch/libexec/config -> /usr/local/etc/elasticsearch
/usr/local/etc/elasticsearch/elasticsearch.yml
```

## Setup Rails App to use ES

#### Install gems

```
gem 'elasticsearch-rails'
gem 'elasticsearch-model'
```

#### Add initializer

Add `elasticsearch.rb` to `config\initiazlers\`. In it the most import thing is to configure Rails to find ES server

```
# ENV['ELASTICSEARCH_ADDRESS_INT'] is the environment variable

config = {
  host: "http://localhost:9200",
  transport_options: {
    request: { timeout: 5 }
  },
  log: true,
}

Elasticsearch::Model.client = Elasticsearch::Client.new(config)

Elasticsearch::Model.client.transport.logger.formatter = proc { |s, d, p, m| "\e[2m#{m}\n\e[0m" }
```

#### Add rake task

Add `elasticsearch.rake` to `lib\tasks`. To this file add:

```
require 'elasticsearch/rails/tasks/import'
```

Later in the document you will see bulk imports which rely on this task

#### Add instrumentation

Add `require 'elasticsearch/rails/instrumentation'` to `application.rb` and logs will display ES search data.

This will add `import` rake tasks.

## Add ES to your Rails model

#### include the module

```
include Elasticsearch::Model
```

This will add proxy methods to the model. The most important one being the `search`. All proxy methods are available under namespace `__elasticsearch__`. These can be accessed on the model `tweet` like `Tweet.__elasticsearch__.search`.

#### Setup index and document name

This is optional but i prefer to define them.

```
index_name Rails.application.class.parent_name.underscore # twitter
document_type self.name.downcase # tweet
```

#### Comparison drawn between RDBS & ES

```
Relational DB  ⇒ Databases ⇒ Tables ⇒ Rows      ⇒ Columns
Elasticsearch  ⇒ Indices   ⇒ Types  ⇒ Documents ⇒ Fields
```

A type represents a class of similar documents. A type has a `name` like `tweet` and it has `mapping` which describes the fields its documents will have like `integer`, `date` etc.

#### Setting up index & mappings

Two settings we care about the most - `number_of_shards` and `number_of_replicas`.

More about settings can be read here:
- https://www.elastic.co/guide/en/elasticsearch/guide/current/_index_settings.html

For mappings setting `dynamic:true` allows adding more fields as we go along and ES will look at the first value and determined what kind of data it and set the field type. For more the fields I know upfront I will setup the field types. The most important things to define are the `type` and whether the field is `analyzed` i.e. inverted-index is setup or if its `not_analyzed` which enables exact searches.

```
settings index: { number_of_shards: 5, number_of_replicas: 1 } do
  mappings dynamic: 'true' do
    indexes :id, type: 'long'
    indexes :message, type: 'text', index: 'analyzed', analyzer: 'english'
    indexes :message_with_name, type: 'text', index: 'analyzed', analyzer: 'english'
    indexes :username, type: 'string', index: 'not_analyzed'
    indexes :the_date, type: 'date', format: 'strict_date_optional_time||epoch_millis'
    indexes :my_phone_numbers, type: 'object' do
      indexes :number, type: 'long'
    end
  end
end
```


More on mappings can be found here:
- https://www.elastic.co/guide/en/elasticsearch/guide/current/dynamic-mapping.html
- https://www.elastic.co/guide/en/elasticsearch/guide/current/custom-dynamic-mapping.html
- https://www.elastic.co/guide/en/elasticsearch/guide/current/complex-core-fields.html

From the rails console you can always check how the settings & mappings are looking like:

```
ap Tweet.settings
ap Tweet.mappings
```

You can also see how the index mappings and settings are in ES

```
http://localhost:9200/_cat/indices?v
http://localhost:9200/tweets/tweet/_mapping
http://localhost:9200/tweets/_mapping/tweet/
http://localhost:9200/_mapping

http://localhost:9200/_settings
http://localhost:9200/tweets/_settings
```

#### Create the index

Once we have setup how the index should be mapped and defined its settings, its time create the index.

```
Tweet.__elasticsearch__.delete_index!
Tweet.__elasticsearch__.create_index!
```

#### Defining the flat data

In ES, data is stored as a flat document and for each record we have in rails we need to produce a corresponsing flat json object which will be stored in ES for that row.

In the model add `as_index_json` and have it return json represenation of what you want to store in ES for that row.

```
def as_indexed_json(options={})
    as_json(
      only: [:id, :message, :username, :message_with_name, :my_phone_numbers, :the_date],
      include: [:phone_numbers],
      methods: [:message_with_name, :my_phone_numbers]
    )
  end
```

The method above when called for a record of the model returns a `json` object. That flat object is stored in ES. We have explicitly defined which columns and method results are stored along with an array of `phone_numbers` which comes from an association.

#### How to manage Relational association relationships in ES.

There are four choices:
1. use an inner object. This is what i did above when i defined my `mappings`.
2. use a nested object.
3. use parent-child relationship
4. denormalize everything

Inner object is the default and what i got by mapping `phone_numbers` to an an `object`. By doing so ES flatens out everything and stores it in the same document. All properties of various `phone_numbers` are grouped together and put under the parent item in the case `tweet`.
Nested query: is what i would get if i had mapped `phone_numbers` to `nested`. This would store the nested object in entirety wihout merging their properties. But this would also require using the `nested` query when searching.
Denormalized: we can copy the parent data in to the child data and store all child data while repeating parent data.

More can be read here:
- https://qbox.io/blog/handling-relationships-using-nested-objects-elasticsearch
- https://qbox.io/blog/modeling-and-managing-relationships-in-elasticsearch
- https://stackoverflow.com/questions/23403149/elasticsearch-relationship-mappings-one-to-one-and-one-to-many
- https://www.elastic.co/blog/managing-relations-inside-elasticsearch
- https://qbox.io/blog/handling-relationships-using-nested-objects-elasticsearch

```
query: {
  nested: {
    path: "phone_numbers",
    query: {
      bool: [
        must: {
          match: {
            phone_numbers.number: "1231232134"
          }
        }
        must: {
          match: {
            phone_numbers.name: "SA"
          }
        }
      ]
    }
  }
}
```

#### Sync data

In our model we need to specify how to sync data as its added in our tables to ES.
The easiest way is to just add `include Elasticsearch::Model::Callbacks` to the model and let rails use callbacks to call ES and store data.

A better approach would be to not include the `callbacks` module and instead use background jobs to call ES as data changes. To accomplish this I am using Resque with Redis and `after_comit` callbacks on my Rails Model.

In my model I added:

```
after_commit :index_on_es, on: [:create, :update]
after_commit :delete_on_es, on: :destroy

def index_on_es
  EsIndexerJob.perform_later("index", self.id)
end

def delete_on_es
  EsIndexerJob.perform_later("delete", self.id)
end
```

And I added new job:

```
require 'resque/errors'
class EsIndexerJob < ApplicationJob
  queue_as :default

  CLIENT = Elasticsearch::Model.client

  def perform(operation, record_id)
    tweet = Tweet.find(record_id)

    case operation.to_s
      when /index/
        CLIENT.index  index: 'tweets', type: 'tweet', id: tweet.id,
                      body: tweet.__elasticsearch__.as_indexed_json
      when /delete/
        CLIENT.delete index: 'tweets', type: 'tweet', id: record_id
      else
        raise ArgumentError, "Unknown operation '#{operation}'"
    end
  rescue Resque::TermException
    Rails.logger.error "Asked to terminate #{self.class}"
    retry_job
  end
end
```

Now with this setup eveyrtime a `tweet` is `created`, `updated` or `destroyed`, we will call the job.
In the job we will get the `tweet_id` and call `index` or `delete` on ES.

But we still have `phone_numbers` as a separate table/model and we are now updating ES when a `phone_number` activity happens while we included `phone_numbers` in the `tweet`.

So in my `phone_number` model i have added:

```
after_commit :index_on_es, on: [:create, :update]
after_commit :delete_on_es, on: :destroy

def index_on_es
  Rails.logger.info "will do EsIndexerJob later for index on #{self.id}"
  EsIndexerJob.perform_later("index", self.tweet.id)
end

def delete_on_es
  Rails.logger.info "will do EsIndexerJob later for delete on #{self.id}"
  EsIndexerJob.perform_later("delete", self.tweet.id)
end
```

Note with the code above, i am re-indexing the entire `tweet` object as there is phone number activity.


## Import Data

Easiest way is to  call `.import` on the model where we have `ES` setup.
In my case I did `Tweet.import`. I noticed that its doing N+1 queries to fetch phone numbers for each tweet and wasn't batching the import and making too many calls to ES. This would meant trouble if I had too many tweets.
To solve the N+1 issue I have add a scope to pre-fetch all phone numbers

```
scope :with_phone_numbers, -> {includes(:phone_numbers)}
```

Now to import I can do any of these

```
Tweet.import scope: 'with_phone_numbers'
Tweet.import query: -> { includes(:phone_numbers) }
```

To do a bulk import we can use the rake task provided by ES.

```
rake environment elasticsearch:import CLASS='Tweet' SCOPE='with_phone_numbers'
```

Note: make sure to have `environment` in the rake command or the rails env wont be loaded when the rake task runs and your Model wont be present

Learn more here:
- https://github.com/elastic/elasticsearch-rails/blob/master/elasticsearch-model/lib/elasticsearch/model/importing.rb

## Searching

Below is an outline of how a search request is structured in rails.

```
  Tweet.__elasticsearch__.search(
    :query => {
    },
    :highlight => {
    },
    :suggest => {
    },
    :from => 0,
    :size => 1000
  )
```

Key things to note: 
- We are calling the `search` method. 
- We are passing in a hash with 5 keys.
- The `query` key is the required one to specify what to search. The other 4 keys are optional.

There are two kind of queries - analyzed & not_analyzed. 
- analyzed queries take search term break it up and search accordingly
- not_analyzed queries are searching for the exact thing.

We have two kind of contexts - query & filter. 
- Things in query context effect the score. 
- Things in filter query do not effect the score and can be picked by ES for caching

Common and easy to use queries:
- match 
- match_all
- match_phrase
- multi_match
- term
- terms
- range ( gte, lte, format)
- exist
- exists_not
- bool ( must, should, filter, must_not)
- ids

See more here: 
- https://www.elastic.co/guide/en/elasticsearch/guide/current/query-dsl-intro.html
- https://www.elastic.co/guide/en/elasticsearch/guide/current/_queries_and_filters.html
- https://www.elastic.co/guide/en/elasticsearch/guide/current/_most_important_queries.html
- https://www.elastic.co/guide/en/elasticsearch/guide/current/combining-queries-together.html


#### Full Text Queries:

Here each fields analyzer is first applied to the query string and then the search is done. 
These queries are useful for `email body` or `tweet message`
- match 
- match_phrase
- multi_match

https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html 
 
#### Term level Queries:

Here the analyzer is not applied on the query string. So its searching for the **exact** term stored in the inverted index. Key word here is that its searching for the exact term for an exact match in the inverted index. If your index was setup to use an analyzer then its very possible that the term you are searching for doesn't exist.
These queries are usually used for structured data like numbers, dates, and enums, rather than full text fields.
e.g. if you are looking for an username "sandeep arneja" you dont want to match on documents like "sandeep x" & "y arneja".
- term
- terms
- range
- exists

https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html

#### `bool` query

My Favorite is a `bool` query which allows combining multiple smaller queries. Its short for boolean query. 

Here is an example outline of a `boolean` query:

```
Tweet.__elasticsearch__.search(
  :query => {
    :bool => [
      :must => {
      },
      :should => [
        {
        
        },
        {
        
        }
      ]
      :filter => {
      },
      :must_not => {
      }
    ]
  }
)
```

Here our query specifys 1 `must` query, 2 `should` queries, 1 `filter` and 1 `must_not`.
- the thing in the `must` query must be present in the results. It effects the score.
- the thing in the `should` is not mandatory to be in the results, when its used with a `must`. It effects the score.
- the thing in `filter` must be present in the results. It runs in the filter context and thus does not effect the score.
- the thing in `must_not` must not be present in the results. It runs in the filter context and thus does not effect the score.   
 
More on `bool` query here: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html

#### `multi_match` query

For individual smaller queries my favorite is `multi_match`.

```
Tweet.__elasticsearch__.search(
  :query => {
    :multi_match => {
      :query => "*hello*",
      :fields => [:message, :username]
    }
  }
)
```

#### `match` with `boolean`

The same be accomplished by using `match` but since it matches on only 1 column, we will use `bool` to help us.

```
Tweet.__elasticsearch__.search(
  :query => {
    :bool => [
      :should => [
        :match => {
          :message => "*hello*" 
        },
        :match => {
          :username => "*hello*" 
        },
      ]
    ]
  }
)
```

#### See all records

use `match_all`. Use to debug or for real use with a `filter` or some other clauses in a `bool`.

```
Tweet.__elasticsearch__.search(
  :query => {
    :match_all => {}
  }
)
```
 
#### A full example query

```
Tweet.__elasticsearch__.search(
  :query => {
    :bool => [
      :must => {
        match_phrase: {
          username: {
            query: "sandeep arneja"
          }
        }
      },
      :should => [
        {
          multi_match: {
            query: "*hello*",
            fields: ["username", "message"]
          }
        }
      ]
      :filter => {
        range: {
          the_date: {
            gte: "2017-01-01"
            lte: "2019-01-01",
            format": "yyyy-MM-dd"
          }
        }
      },
      :must_not => {
        term: {
          message: "Hello World"
        }
      }
    ]
  }
)
```
  
## Working with the results

To make life easier intall `awesome_print` gem and then prefix each command when in the rails console with an `ap`. 
This will do 2 impotant things:
1. it will call `to_hash` on the response and then print it nicely on the screen
2. it will cause the query to execute


#### `.response`

To see the entire result access the `response` object.

```
ap Tweet.__elasticsearch__.search(
  :query => {
    :match_phrase => {
      :username => "sandeep arneja"
    }
  }
).response
```

And you will see instrumentation print:

```
GET http://localhost:9200/tweets/tweet/_search [status:200, request:0.008s, query:0.000s]
> {"query":{"match_phrase":{"username":"sandeep arneja"}}}
< {"took":0,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":0.6791366,"hits":[{"_index":"tweets","_type":"tweet","_id":"4","_score":0.6791366,"_source":{"id":4,"username":"sandeep arneja","message":"hi there - i have a date","the_date":"2017-12-09T00:00:00.000Z","message_with_name":"Answer is sandeep arneja : -\u003e hi there - i have a date","my_phone_numbers":[],"phone_numbers":[]}}]}}
  Tweet Search (8.6ms) {index: "tweets", type: "tweet", body: {query: {match_phrase: {username: "sandeep arneja"}}}}
``` 

And the output is:

```
{
         "took" => 0,
    "timed_out" => false,
      "_shards" => {
             "total" => 5,
        "successful" => 5,
           "skipped" => 0,
            "failed" => 0
    },
         "hits" => {
            "total" => 1,
        "max_score" => 0.6791366,
             "hits" => [
            [0] {
                 "_index" => "tweets",
                  "_type" => "tweet",
                    "_id" => "4",
                 "_score" => 0.6791366,
                "_source" => {
                                   "id" => 4,
                             "username" => "sandeep arneja",
                              "message" => "hi there - i have a date",
                             "the_date" => "2017-12-09T00:00:00.000Z",
                    "message_with_name" => "Answer is sandeep arneja : -> hi there - i have a date",
                     "my_phone_numbers" => [],
                        "phone_numbers" => []
                }
            }
        ]
    }
}
```

The `response` object gave us everything ES returned. To just get the meat we could call `.hits.hits` on the response.

#### `.records` 

We can also call `records` to get Rails `ActiveRelation` object. This is the stand rails result we get when doing a `where` query.

```
recs = Tweet.__elasticsearch__.search(
  :query => {
    :match_phrase => {
      :username => "sandeep arneja"
    }
  }
).records

recs.each { |r| ap r }
```

#### `.results`

This returns all the results as stored in ES

```
res = Tweet.__elasticsearch__.search(
  :query => {
    :match_phrase => {
      :username => "sandeep arneja"
    }
  }
).results

res.each { |r| ap JSON.parse(r.to_json) }
```
## highligting

Add the keyword `:highlight` to the hash passed to the `search` method

```
:highlight => {
  pre_tags: ['<mark>'],
  post_tags: ['</mark>'],
  fields: {
    message: {},
    username: {},
  }
},
```

```
x = Tweet.search("hi")
x.results[0].message
x.results[0].highlight.message.join(" ")
```

## suggesting

Add the keyword `:suggest` to the hash passed to the `search` method

```
suggest: {
  text: query,
  username: {
    term: {
      size: 1,
      field: :username
    }
  },
  message: {
    term: {
      size: 1,
      field: :message
    }
  }
},
```

```
x = Tweet.search("hell")
x.response.suggest.message[0].options # []
x.response.suggest.username[0].options # []
```


## montoring tool

checkout elasticsearch-head
https://github.com/mobz/elasticsearch-head

### Other getting start guides

- http://www.codinginthecrease.com/news_article/show/409843
- https://www.pluralsight.com/guides/ruby-ruby-on-rails/elasticsearch-with-ruby-on-rails
- https://aaronvb.com/articles/intro-to-elasticsearch-ruby-on-rails-part-1.html
- http://www.rubydoc.info/gems/elasticsearch-model
- https://www.slideshare.net/tomzeng/using-elasticsearch-with-rails