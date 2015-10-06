---
layout: post
title: json properties in postgres
categories: [json, activerecord, postgres, rails]
tags: [json, activerecord, postgres, rails]
published: True

---

Getting distinct records like `select distinct * from visitors` or like this in Rails 3.2 `Visitor.select("distinct *")` is not going to work if you have a `json` property on the table in postgress (9.3.3)

````
Visitor.select("distinct *")
Visitor Load (4.6ms)  SELECT distinct * FROM "visitors"
PG::UndefinedFunction: ERROR:  could not identify an equality operator for type json
LINE 1: SELECT distinct * FROM "visitors"
                        ^
: SELECT distinct * FROM "visitors"
Hirb Error: PG::UndefinedFunction: ERROR:  could not identify an equality operator for type json
LINE 1: SELECT distinct * FROM "visitors"
                        ^
: SELECT distinct * FROM "visitors"
    /Users/sandeeparneja/.rvm/gems/ruby-2.1.3@fk/gems/activerecord-3.2.21/lib/active_record/connection_adapters/postgresql_adapter.rb:1163:in `async_exec'
    /Users/sandeeparneja/.rvm/gems/ruby-2.1.3@fk/gems/activerecord-3.2.21/lib/active_record/connection_adapters/postgresql_adapter.rb:1163:in `exec_no_cache'
    /Users/sandeeparneja/.rvm/gems/ruby-2.1.3@fk/gems/activerecord-3.2.21/lib/active_record/connection_adapters/postgresql_adapter.rb:660:in `block in exec_query'
    /Users/sandeeparneja/.rvm/gems/ruby-2.1.3@fk/gems/activerecord-3.2.21/lib/active_record/connection_adapters/abstract_adapter.rb:280:in `block in log'
    /Users/sandeeparneja/.rvm/gems/ruby-2.1.3@fk/gems/activesupport-3.2.21/lib/active_support/notifications/instrumenter.rb:20:in `instrument'
````

In Postgress 9.3.3 the database can not tell if two json values are the same or not. Makes me wonder maybe I should have used a `text` attribute.

The work around I have employed is to use `group by` to get distinct records. This works becuase when getting a distinct record I am doing a distinct by checking just the visitors.id column

````
Visitor.scoped.group("visitors.id")
Visitor Load (17.1ms)  SELECT "visitors".* FROM "visitors" GROUP BY visitors.id
````

Regarding performance, the `group by` is also using the index on the visitor.id

````
Visitor.group("visitors.id").explain
Visitor Load (9465.2ms)  SELECT "visitors".* FROM "visitors" GROUP BY visitors.id
EXPLAIN (64.8ms)  EXPLAIN SELECT "visitors".* FROM "visitors" GROUP BY visitors.id
=> "EXPLAIN for: SELECT \"visitors\".* FROM \"visitors\"  GROUP BY visitors.id
QUERY PLAN
------------------------------------------------------------------------------------------------
Group  (cost=0.42..57090.60 rows=499318 width=1398)
Group Key: id
->  Index Scan using visitors_pkey on visitors  (cost=0.42..55842.31 rows=499318 width=1398)(3 rows)"
````
