---
title: How to do insert or update a large number of records
status: true
layout: post
categories: [Rails5, Postgresql]
tags: [Upreset, SQL]
published: True
---

### Problem:

We have a large number of records (~100,000+). We need to save these records in our database.

The first time these records are fetched we will be `inserting` them. Next time they are fetched we will be `updating` previously inserted records and `inserting` only the newly fetched records.

### Solution:

#### v0.1

```
data = [
 {
   name: "Sandeep",
   email: "abc@xyz.com"
 },
 {
   name: "Rodrigo",
   email: "def@xyz.com"
 }
]

data.each do |item|
  user = User.find_or_initialize(:email => item.email)
  user.udpate_attributes item
end
```

Here we are looping through all the records, and call `find_or_initialize` on each records by its unique identifier of `email`. `find_or_intialize` is a cool rails helper which returns a record either an existing one, or a newly initialized, not yet persisted, record. In the next step we are updating this record with all the params available.

Although cool, this strategy fails miserably when processing a high number of recorders.
- It is O(N).
- It opens and closes a transaction every time it goes through the loop.
- It attempts to updating record where nothing changed
- It fires all of the model callbacks.


#### v0.2

```
ActiveRecord::Base.transaction do
  data.each do |item|
    user = User.find_or_initialize(:email => item.email)
    user.udpate_attributes item
  end
end

Here by wrapping all the requests in 1 transaction, we are saving on opening and closing N transactions. This is certainly an improvement over the previous solution, but still wont cut it when processing a large number of records. This is primarily because its making 2N transactions.

```

#### v0.3

```
add_index :users, :email, :unique => true

inserts = data.each_with_object([]) do |item, arr|
  name = ActiveRecord::Base.sanitize(item.name)
  email = ActiveRecord::Base.sanitize(item.email)
  arr <<  "insert into users(email ,name) VALUES(#{email}, #{name}) " +
          "ON CONFLICT(email) DO UPDATE SET name=#{name}"
end

ActiveRecord::Base.transaction do
  inserts.each do |insert|
    ActiveRecord::Base.execute(insert)
  end
end
```

First i am adding a `unique` index on `email`. I need this because I am going to be inserting all the records fetched. By having a unique index, the database will do the work of rejecting a record being inserted which already exists.

Then I am using the `On Conflict` option of the `Insert`, specifying the kind of conflict and then specfying it do an update when there is a conflict.

Also I am using a raw sql to do the actual insert.

This approach is now a lot better than our earlier attempts.


#### v0.4

```
inserts = data.each_with_object([]) do |item, arr|
  name = ActiveRecord::Base.sanitize(item.name)
  email = ActiveRecord::Base.sanitize(item.email)
  arr <<  "(#{email}, #{name}) "
end

sql_string = "insert into users(email ,name)"+
          " VALUES #{inserts.join(',')}"+
          " ON CONFLICT
            (email)
          DO UPDATE SET
            name = excluded.name"

ActiveRecord::Base.connection.execute(sql_string)

```

In this approach we are making exactly 1 SQL call. Its O(1). We do this by doing providing multiple values to the `insert` statement and using the special keyword `excluded` to access the value being inserted with there is a conflict.