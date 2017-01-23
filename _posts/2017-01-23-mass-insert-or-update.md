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
find_or_initialize(:id)
update_attributes
```

#### v0.2

```
transaction do
end
```

#### v0.3

- raw sql multiple insert statements
- raw sql update all records
- sanatize input values

#### v0.4

- single insert statement with repeated values

#### v0.5

- Upsert





