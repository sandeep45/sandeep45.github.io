---
title: Distinct vs Distinct on
status: True
layout: post
categories: [Postgresql, sql, rails, distinct, distinct_on]
tags: [Postgresql, sql, rails, distinct, distinct_on]
published: True
---

Let start the basic command - `distinct`. Regardless of your belief it will:
- Make each row unique
- When checking for uniqueness it will look at all columns selected.
- It does not care for whats in parenthesis around it.
- It does not send any column to display.
- You dont do distinct on any columns
- It is just a keyword added at the beginning of the select statement
- It is not proceeded by parenthesis or a comma.

### Scenario 1

Lets print all `organizations` with just the `user_defined_status` column. With the help of `distinct` we should get only 1 row, as all organizations have the same value of `nil` for `user_defined_status` column

```ruby
Organization.
  select('distinct user_defined_status')
```

```sql
SELECT distinct user_defined_status
FROM "organizations"
```

```
+----+---------------------+
| id | user_defined_status |
+----+---------------------+
|    |                     |
+----+---------------------+
```

Cool, so distinct works as expected since we see only 1 row printed. But this example kind of sucks as the value is blank. Lets just do another example:

```ruby
 Campaign.
  select("name")
```

```sql
SELECT "campaigns"."name"
FROM "campaigns"
```

```
+----+------+
| id | name |
+----+------+
|    | soup |
|    |      |
|    | foo  |
|    | foo  |
|    | foo  |
|    | foo  |
+----+------+
```

```ruby
Campaign.
  select("distinct name")
```

```sql
SELECT distinct name 
FROM "campaigns"
```

```
+----+------+
| id | name |
+----+------+
|    | foo  |
|    | soup |
|    |      |
+----+------+
```

Cool again. `distinct` has narrowed down the results so each row is has a unique `campaigns.name`

### Scenario 2

Lets add more columns to the output:

```ruby
Organization.
  select('distinct user_defined_status, credits')
```

```sql
SELECT distinct user_defined_status, credits 
FROM "organizations"
```

```
+----+---------------------+---------+
| id | user_defined_status | credits |
+----+---------------------+---------+
|    |                     | 101.0   |
|    |                     | 0.0     |
+----+---------------------+---------+
```

In the output above you can see that each row is unique. Lets add parenthesis around user_defined_status and see if makes rows unique by just that column or does it still look at all columns.

```ruby
Organization.
  select('distinct(user_defined_status), credits')
```

```sql
SELECT distinct(user_defined_status), credits 
FROM "organizations"
```

```
+----+---------------------+---------+
| id | user_defined_status | credits |
+----+---------------------+---------+
|    |                     | 101.0   |
|    |                     | 0.0     |
+----+---------------------+---------+
```
So we can see adding parenthesis around a column made no difference to the `distinct` command. Both rows are printed even though they both have the same value of `nil` for `user_defined_status`. 

### Scenario 3

Lets add a join and see if distinct works as expected or does it go bonanza

```ruby
Organization.
  joins(:campaigns).
  select("organizations.id as org_id, campaigns.name as camp_name").
  order("organizations.id, campaigns.name")
``` 

```sql
 SELECT organizations.id as org_id, campaigns.name as camp_name 
 FROM "organizations" 
 INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
 ORDER BY organizations.id, campaigns.name
```

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | foo       |
|    | 1      | foo       |
|    | 1      | soup      |
|    | 2      | foo       |
|    | 2      | foo       |
|    | 2      |           |
+----+--------+-----------+
```

In the output above we can see that each organization has 3 campaigns, so we got a total of 6 rows. Lets add `distinct` to this query and see what we get. What we should get are distinct rows and not unique organizations or campaigns.

```ruby
Organization.
  joins(:campaigns).
  select("distinct organizations.id as org_id, campaigns.name as camp_name").
  order("organizations.id, campaigns.name")
```

```sql
 SELECT distinct organizations.id as org_id, campaigns.name as camp_name 
 FROM "organizations" 
 INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
 ORDER BY organizations.id, campaigns.name
```

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | foo       |
|    | 1      | soup      |
|    | 2      | foo       |
|    | 2      |           |
+----+--------+-----------+
```

So here we can see `distinct` working again. It removed 2 rows, as the values were producing duplicate rows. Note here, even though distinct worked and removed duplicate rows, we have 2 rows for organization 1, and 2 rows for organization 2. Before we had 3, but now have 2. 2 is better than 3, but what if we want only 1 row per organization. What if the task is to print 1 row per organization with any 1 campaign name it has?

This can be done by using `distinct on` or `group by`. 

### Scenario 4

Let's use `group by` to print 1 row per organization and 1 campaign name for each organization

```ruby
 Organization.
  joins(:campaigns).
  group("organizations.id").
  select("organizations.id as org_id, campaigns.name as camp_name").
  order("organizations.id, campaigns.name")
```

```sql
SELECT organizations.id as org_id, campaigns.name as camp_name 
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
GROUP BY organizations.id 
ORDER BY organizations.id, campaigns.name
```

Ehhhh... Breaks applied. This wont work and error out. Once a `group by` has been applied, then all other columns from the table on the right which have multiple values, must either also be in the `group_by` that is therefore being unique or must be in an aggregate function and thus again being unique. This makes sense because on the left side, sql is only printing 1 organization, but each organization has 3 campaigns, it needs to know which campaign out of the 3 to print on that row. We need to tell that by using an aggregate function.

The exact error SQL throws:

```sql
 PG::GroupingError: ERROR:  column "campaigns.name" must appear in the GROUP BY clause or be used in an aggregate function
LINE 1: SELECT organizations.id as org_id, campaigns.name as camp_na...
```

Before we proceed, to explore this error and solving it, let me just point out one fine detail. Only columns from the right side or the ones which have multiple possible values need to addressed to remove the error.

```ruby
Organization.
  joins(:campaigns).
  group("organizations.id").
  select("organizations.id as org_id, organizations.user_defined_status").
  order("organizations.id")
``` 

```sql
SELECT organizations.id as org_id, organizations.user_defined_status 
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
GROUP BY organizations.id 
ORDER BY organizations.id
```

```
+----+---------------------+--------+
| id | user_defined_status | org_id |
+----+---------------------+--------+
|    |                     | 1      |
|    |                     | 2      |
+----+---------------------+--------+
```

In the example above, i added `organizations.user_defined_status` column instead of `campaigns.name`. It didint complain because this column is on the `organization` and therefore we dont have multiples of these.

```ruby
Organization.
  joins(:campaigns).
  group("organizations.id, campaigns.name").
  select("organizations.id as org_id, campaigns.name as camp_name").
  order("organizations.id, campaigns.name")
```

```sql
SELECT organizations.id as org_id, campaigns.name as camp_name 
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
GROUP BY organizations.id, campaigns.name 
ORDER BY organizations.id, campaigns.name
```

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | foo       |
|    | 1      | soup      |
|    | 2      | foo       |
|    | 2      |           |
+----+--------+-----------+
```

In this example above I am using `campaigns.name` and it no longer complains because I have added it to the `group_by` clause. Now `organizations.id` & `campaigns.name` put together defined the rows being printed, therefore we dont have multiple of `campaigns.name` and thus no error.

Okay now lets go back to the orignal problem of us wanting to print 1 row per organization with any campaign_name.

```ruby
 Organization.
  joins(:campaigns).
  group("organizations.id").
  select("organizations.id as org_id, (array_agg(campaigns.name))[2] as camp_name").
  order("organizations.id")
```  

```sql
 SELECT organizations.id as org_id, (array_agg(campaigns.name))[2] as camp_name 
 FROM "organizations" 
 INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
 GROUP BY organizations.id 
 ORDER BY organizations.id
```

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | foo       |
|    | 2      | foo       |
+----+--------+-----------+
```

Ahhha, finally we got what we wanted. We were able to do this by using `array_agg` which basically joined all the multiple campaign names together and then we just said to print us the one at index 2. We said 2, but could have said any 0, 1 or 2. 

To better understand this, lets print them all.

```ruby
Organization.
  joins(:campaigns).
  group("organizations.id").
  select("organizations.id as org_id, array_agg(campaigns.name) as camp_name").
  order("organizations.id")
```

```sql
SELECT organizations.id as org_id, array_agg(campaigns.name) as camp_name 
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
GROUP BY organizations.id 
ORDER BY organizations.id
```

```
+----+--------+----------------+
| id | org_id | camp_name      |
+----+--------+----------------+
|    | 1      | soup, foo, foo |
|    | 2      | , foo, foo     |
+----+--------+----------------+
```

### Scenario 5

Okay, so we got one row per organization with campaign name of our choice on the right. But its slow!. `array_agg` is not performant and kind feels hacky!. Lets explore and use `distinct on`. 

A distinct on:

- takes a comma separated list of columns on which the row is to be made unique
- it doesn't print anything to the output
- it doesn't expect a comma to be printed after it, this is just like distinct
- when there are multiple results, it will just pick the first one and skip all other rows till it gets to the next unique row
- it looks and cares about whats passed to it in parenthesis as that's what the rows will be unique by
- it should be used with `order by` to get consistent results.

Lets first once again just print all organizations with all of their campaign names on multiple lines.

```ruby
Organization.
  joins(:campaigns).
  select("organizations.id as org_id, campaigns.name as camp_name").
  order("organizations.id, camp_name")
```

```sql
SELECT organizations.id as org_id, campaigns.name as camp_name 
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
ORDER BY organizations.id, camp_name
``` 

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | foo       |
|    | 1      | foo       |
|    | 1      | soup      |
|    | 2      | foo       |
|    | 2      | foo       |
|    | 2      |           |
+----+--------+-----------+
```

Look at the output above very carefully. What we really want is to print row number 1 and row number 4. This is what `distinct on` can exactly do for us. All we have to do is, tell it to make each row distinct based on the `org_id`. This will reduce the result to 2 rows as we have 2 distinct values of 1 & 2 for `org_id`. Then we need it to pick up any `camp_name`. This will happen by default with `distinct on` as it just picks the first.

```ruby
Organization.
joins(:campaigns).
select("distinct on(organizations.id) organizations.id as org_id, campaigns.name as camp_name").
order("organizations.id, camp_name")
```  

```sql
SELECT distinct on(organizations.id) organizations.id as org_id, campaigns.name as camp_name
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
ORDER BY organizations.id, camp_name
```

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | foo       |
|    | 2      | foo       |
+----+--------+-----------+
```

### Scenario 6

Lets say we want the last `campaigns.name` for each organization. This can be easily accomplished by changing the order of things. `disntinct on` will still return the first row which is distinct based on the columns passed to it. What we will do is use order to just change the way things appear in front of `distinct on`.

Lets first, once again print all the organizations with their campaign names on multiple rows, but this time we will order it so campaign names reversed, i.e. the last campaign name, this time will be the first campaign name

```ruby
Organization.
joins(:campaigns).
select("organizations.id as org_id, campaigns.name as camp_name").
order("organizations.id, camp_name DESC")
```

```sql
SELECT organizations.id as org_id, campaigns.name as camp_name 
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
ORDER BY organizations.id, camp_name DESC
```

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | soup      |
|    | 1      | foo       |
|    | 1      | foo       |
|    | 2      |           |
|    | 2      | foo       |
|    | 2      | foo       |
+----+--------+-----------+
```

Cool, now that we have campaign names flipped, lets add the `distinct on`.

```ruby
Organization.
  joins(:campaigns).
  select("distinct on(organizations.id) organizations.id as org_id, campaigns.name as camp_name").
  order("organizations.id, camp_name DESC")
```

```sql
SELECT distinct on(organizations.id) organizations.id as org_id, campaigns.name as camp_name 
FROM "organizations" 
INNER JOIN "campaigns" ON "campaigns"."organization_id" = "organizations"."id" 
ORDER BY organizations.id, camp_name DESC
```

```
+----+--------+-----------+
| id | org_id | camp_name |
+----+--------+-----------+
|    | 1      | soup      |
|    | 2      |           |
+----+--------+-----------+
```

And it works, yay!


### Side Notes:
- You can get the same result with `distinct` as you would get with `distinct on` if your reduce the columns you are going to select to the same columns you were going to do `distinct on ()`
- `SELECT DISTINCT ON ( expression [, …] )` **keeps** only the first row of each set of rows where the given expressions evaluate to be unique
- `distinct on` is kinda working after the fact. It is filtering out rows which match the same clause and keeping the first one, after all the joins and stuff have happened.
- Best way to think of `distinct on` is to run your query without, let it print duplicate rows the data on left being same, and data on right getting repeated, and then ask yourself would you like to see just the first row of each group. if so add the distinct on. if you want to change how the groupings are defined, then control that by the expressions in distinct on. If you want to change which row should be the first one in each group, then do that by adding orders.
- `distinct` will make the whole row unique, every column of it. It can not take comma seperated list of columns to make unique by. Intead you will have to reduce the number of columns in output.
- `group by` can take list of columns to make output unique on but will need aggregate function to handle the columns which have multiple possible values.

 

