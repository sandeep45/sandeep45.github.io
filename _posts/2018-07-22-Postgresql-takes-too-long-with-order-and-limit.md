---
title: Order & Limit 1 do not play well together when there is no data
status: True
layout: post
categories: [Postgresql, Rails, Sql, Database]
tags: [Postgresql, Rails, Sql, Database]
published: True
---

Lets explore this scenario in which Postgresql is taking upwards of 10 seconds where I expect it to take a few milliseconds.

Lets make a query below which has a 0 results and returns an answer in milliseconds.

```ruby
Impression.
  joins(:prospect => :account).
  where("accounts.id = 1234567890")
``` 

```sql
SELECT "impressions".* FROM "impressions" 
INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" 
INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" 
WHERE (accounts.id = 1234567890)
```

```
Impression Load (1.5ms)
#<ActiveRecord::Relation []>
```

Here we got 0 results in 1.5ms, simply because I used an account id of `1234567890` which doesn't exist.

Now, lets add an order to this query.

```ruby
Impression.
  joins(:prospect => :account).
  where("accounts.id = 1234567890").
  order("impressions.id")
``` 

```sql
SELECT "impressions".* FROM "impressions" 
INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" 
INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" 
WHERE (accounts.id = 1234567890)
ORDER BY impressions.id
```

```
Impression Load (1.5ms)
#<ActiveRecord::Relation []>
```

Adding an order cause no damage, still 0 results in 1.5ms. Now lets remove the order and instead add a limit of 1.

```ruby
Impression.
  joins(:prospect => :account).
  where("accounts.id = 1234567890").
  limit(1)
``` 

```sql
SELECT "impressions".* FROM "impressions" 
INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" 
INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" 
WHERE (accounts.id = 1234567890)
limit 1
```

```
Impression Load (1.5ms)
#<ActiveRecord::Relation []>
```

Again, no damage done. Now lets add them both - `order` and the `limit 1`

```ruby
Impression.
  joins(:prospect => :account).
  where("accounts.id = 1234567890").
  order("impressions.id").
  limit(1)
``` 

```sql
SELECT "impressions".* FROM "impressions" 
INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" 
INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" 
WHERE (accounts.id = 1234567890)
ORDER BY impressions.id
limit 1
```

```
Impression Load (10439.2ms)
#<ActiveRecord::Relation []>
```

Whoh!! Now it is taking 10 seconds. This is crazy. Lets do an explain here to dive in.

```
EXPLAIN for: SELECT  "impressions".* FROM "impressions" INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" WHERE (accounts.id = 1234567890) ORDER BY impressions.id LIMIT $1 [["LIMIT", 1]]
                                                     QUERY PLAN
---------------------------------------------------------------------------------------------------------------------
 Limit  (cost=1.27..7202.37 rows=1 width=1733)
   ->  Nested Loop  (cost=1.27..3629355.16 rows=504 width=1733)
         ->  Nested Loop  (cost=0.85..3629340.43 rows=504 width=1737)
               ->  Index Scan using impressions_pkey on impressions  (cost=0.43..1565854.01 rows=4477707 width=1733)
               ->  Index Scan using prospects_pkey on prospects  (cost=0.42..0.45 rows=1 width=8)
                     Index Cond: (id = impressions.prospect_id)
                     Filter: (account_id = 1234567890)
         ->  Materialize  (cost=0.41..8.44 rows=1 width=4)
               ->  Index Only Scan using accounts_pkey on accounts  (cost=0.41..8.43 rows=1 width=4)
                     Index Cond: (id = 1234567890)
(10 rows)
```

It seems that having a limit of 1, throws off the engine. I dont fully understand the reason but these two articles seems to know what they are talking about:
- https://dba.stackexchange.com/questions/110636/postgres-poor-performance-on-order-by-id-desc-limit-1/110919
- https://stackoverflow.com/questions/21385555/postgresql-query-very-slow-with-limit-1

Interestingly enough, if we increase the number we want to limit by, the query time drops

```ruby
Impression.
  joins(:prospect => :account).
  where("accounts.id = 1234567890").
  order("impressions.id").
  limit(5)
``` 

```sql
SELECT "impressions".* FROM "impressions" 
INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" 
INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" 
WHERE (accounts.id = 1234567890)
ORDER BY impressions.id
limit 5
```

```
Impression Load (1.6ms)
#<ActiveRecord::Relation [{Object}]>
```

Also if data is found, then the query is fast again

```ruby
Impression.
  joins(:prospect => :account).
  where("accounts.id = 1").
  order("impressions.id").
  limit(1)
``` 

```sql
SELECT "impressions".* FROM "impressions" 
INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" 
INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" 
WHERE (accounts.id = 1)
ORDER BY impressions.id
limit 1
```

```
Impression Load (1.6ms)
#<ActiveRecord::Relation [{Object}]>
```

## A Solution

We know the query is fast without the limit, so lets make the query do just that. Then we can put a query around that and in the outside query just order and limit the result of the first query.

```ruby
Impression.select("*").from(
  Impression.joins(:prospect => :account).
    where("accounts.id = 1234567890").
    order("impressions.id")
).
order("id").
limit(1)
```

```sql
 SELECT  * FROM (
  SELECT "impressions".* FROM "impressions" 
  INNER JOIN "prospects" ON "prospects"."id" = "impressions"."prospect_id" 
  INNER JOIN "accounts" ON "accounts"."id" = "prospects"."account_id" 
  WHERE (accounts.id = 1234567890) 
  ORDER BY impressions.id
) subquery 
ORDER BY id 
LIMIT 1
```

```
Impression Load (2.1ms)
#<ActiveRecord::Relation [{Object}]>
```

Something to note here:
- it is slow only when there is no data. if data was found its fast
- it is no longer slow if we add a limit of a number higher than 1
- all the joins I had, have nothing to do with the issue as shown below

```
irb(main):065:0> Impression.where("creative_id = 1234567890")
  Impression Load (0.8ms)  SELECT "impressions".* FROM "impressions" WHERE (creative_id = 1234567890)
=> #<ActiveRecord::Relation []>
irb(main):066:0> Impression.where("creative_id = 1234567890").order("impressions.id")
  Impression Load (1.1ms)  SELECT "impressions".* FROM "impressions" WHERE (creative_id = 1234567890) ORDER BY impressions.id
=> #<ActiveRecord::Relation []>
irb(main):067:0> Impression.where("creative_id = 1234567890").limit(1)
  Impression Load (0.9ms)  SELECT  "impressions".* FROM "impressions" WHERE (creative_id = 1234567890) LIMIT $1  [["LIMIT", 1]]
=> #<ActiveRecord::Relation []>
irb(main):068:0> Impression.where("creative_id = 1234567890").order("impressions.id").limit(1)
  Impression Load (3840.6ms)  SELECT  "impressions".* FROM "impressions" WHERE (creative_id = 1234567890) ORDER BY impressions.id LIMIT $1  [["LIMIT", 1]]
=> #<ActiveRecord::Relation []>
irb(main):069:0>
```