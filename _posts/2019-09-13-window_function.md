---
title: Getting started with Window Functions
status: 
layout: post
categories: [SQL, Postgresql, Window]
tags: [SQL, Postgresql, Window]
published: True
---


In the article, we will be discussing window functions, but before we begin we need to go over the pre-requisites. Also, you will see some window functions in the pre-reqs. If you understand them great, if not, dont worry and just keep reading. Using real window functions helps keep examples real and you will eventually undertand them all when you reach the section where those window functions are discussed.

## Refernces:

Breif Intro on Window Functions - https://www.postgresql.org/docs/9.1/tutorial-window.html

Various General Purpose Window Functions - https://www.postgresql.org/docs/9.1/functions-window.html

Various General Purpose Agregate functions - https://www.postgresql.org/docs/9.5/functions-aggregate.html

Window Frame Definition - https://www.postgresql.org/docs/10/sql-expressions.html#SYNTAX-WINDOW-FUNCTIONS

Select Statement - https://www.postgresql.org/docs/9.1/sql-select.html

Tutorial on Window Queries - https://www.postgresql.org/docs/9.4/queries-table-expressions.html#QUERIES-WINDOW

Other Window Function Blog Posts worthy of attention:
- https://www.compose.com/articles/metrics-maven-window-frames-in-postgresql/
- https://blog.matters.tech/sql-window-functions-basics-e9a9fa17ce7e
- https://www.depesz.com/2012/11/20/window-window-on-the-wall/
- https://blog.matters.tech/sql-window-functions-basics-e9a9fa17ce7e

## Sample Table for our examples

items

```
"id","name","amount"
1,"food",10
2,"car",20
3,"food",50
4,"car",100
```

## Order of Computation:

A SQL select statement runs in the order mentioned below. The first thing to get computed is the `from` statement and the last being `offset`. Everything else, happens in the middle, in the order mentioned below:

- from
- where
- group
- aggregate
- having
- window functions
- select
- distinct
- union/intersect/except
- order by
- offset

Lets go over a few intersting things which happen due to the select order of computation

#### Use group-by inside window functions

The group and its aggregate functions are computed before the window functions. This means that we can use results of a group by's aggregate function inside a window function.

e.g.

```
"id","name","amount"
1,"food",10
2,"car",20
3,"food",50
4,"car",100
```

```
select name, sum(amount) from items group by name;
```

```
name,sum
food, 60
car, 120
```

```
select name, sum(amount), sum(sum(amount)) over () from items group by name;
```

```
name,sum,sum
food, 60, 180
car, 120, 180
```

```
select name, sum(amount), rank() over (order by sum(amount) DESC) from items group by name;
```

```
name,sum,rank
car, 120, 1
food, 60, 2
```

Here you can see that we are using the results of the aggregate `sum` function inside `sum` & `rank` window functions.

I know we haven't learned what window functions are yet, but here i want to bring your attention to the fact the window functions come after group-by functions in the order of computation and therefore you can use results of group-by aggregate function in window functions.

#### Can-not use friendly names in window functions

Another interesting thing here is that the `select` comes after `window functions` but before `order by`. This means that if we rename a column to a friendlier name, then we can't reference that column with its friendlier name in the window functions.

e.g.

```
id,name,amount
1, food, 10
2, car, 20
3, food, 50
4, car, 100
```

```
select id, name, amount, amount+100 as larger_amount from items;
```

```
id,name,amount, larger_amount
1, food, 10, 110
2, car, 20, 120
3, food, 50, 150
4, car, 100, 200
```

```
select id, name, amount, amount+100 as larger_amount,
sum(larger_amount) over(order by larger_amount ASC)
from items;
```

```
ERROR:  column "larger_amount" does not exist
```

The window function can not see the friendly name, as its computed in select which happens after the window function.

To solve it we may use CTE's like this:

```
with foo as (
  select id, name, amount, amount+100 as larger_amount from items
)
select *, sum(larger_amount) over(order by larger_amount ASC) from foo;
```

```
id, name,amount, larger_amount, sum
1, food, 10, 110, 110
2, car, 20, 120, 230
3, food, 50, 150, 380
4, car, 100, 200, 580
```

#### You can use friendly names in order-by

Now on the other hand, we can use the friendly name in the `order by` as its computed after the `select`

```
select id, name, amount, amount+100 as larger_amount
from items
order by larger_amount DESC;
```

```
id,name,amount, larger_amount
4, car, 100, 200
3, food, 50, 150
2, car, 20, 120
1, food, 10, 110
```

#### You can not use window functions in where caluse

This is obvious as based on the order of computation because `where` clause is computed first and `window` happens later

## CTE - Custom Table Expression

I prefer to use CTE over sub-queries as it reads better. You can read them top to bottom rather than inside to outside in case of sub queries.

A common use case for CTE and window functions is when we want to fight the Nth highest record. With CTE first we can apply record and then in the second step of CTE just get the Nth record.

```
with foo as (
select *,
rank() over (order by amount DESC)
from items
)
select * from foo where rank = 2
```

```
"id","name","amount","rank"
3,"food",50,2
```

#### Using Window functions over window functions

e.g. Lets say we want to:
- first calculate a running sum of our items per the name category
- and then rank each row based on its running sum
- and then only see the Number 1 ranked item of each category

```
select * from items order by id;
```

```
id,name,amount
1, food, 10
2, car, 20
3, food, 50
4, car, 100
```

```
with foo as (
select *, sum(amount) over (partition by name order by id) from items
),
moo as (
select *, rank() over (partition by name order by sum ASC) from foo
)
select * from moo where rank = 1
```

```
id, name, amount, sum, rank
2, car, 20, 20, 1
1, food, 10, 10, 1
```

#### Adding results horizontally

Another interesting use case of CTE is to add results horizontally.

e.g.

```
with foo as (
select * from items
),
moo as (
select * from items
)
select * from moo inner join foo on foo.id = moo.id;
```

Above will just horizontally double the table. This trick can sometimes come handy.

#### Using friendly names in Window Functions

Another interesting use case of CTE is when you want to add clarity by naming things in the select statement and then use the friendly names in the following commands.

e.g.

Lets calculate running sum via a window function of larger_amount column which is basically sum+100

```
with foo as (
  select id, name, amount, amount+100 as larger_amount from items
)
select *, sum(larger_amount) over(order by larger_amount ASC) from foo;
```

```
id, name,amount, larger_amount, sum
1, food, 10, 110, 110
2, car, 20, 120, 230
3, food, 50, 150, 380
4, car, 100, 200, 580
```

## Aggregate Function

These are the general purpose regular aggregate functions. Let's start with an example first. We want to know the total amount. Aggregate function comes to our rescue.

```
id,name,amount
1, food, 10
2, car, 20
3, food, 50
4, car, 100
```

```
select sum(amount) from items;
```

```
sum
180
```

#### So lets analyze what just happened

We used the `sum` aggregate function. We passed in a column name to it and ran it on a bunch of rows, which in our case were just all the rows in the `items` table. The sql ran and it returned us just 1 row back. The result has only 1 column called sum with the value we want.

Key thing to note here:

- The input is N rows and the output is 1 row.
- In the output none of the individual data of the `items` is available. We no longer know the `ids` and `names` of the 4 items we have

So, even though aggregate function gave us something, in this case the `sum` of the amount across all `items`, it also took away something, in this case that is the details of each item like their `id`, `name` etc.

Let this official definition sink in:
> An aggregate function reduces multiple inputs to a single output value, such as the sum or average of the inputs.

Here are more examples:

```
selet count(*) from items;
```

```
count
4
```

```
select array_agg(name order by amount DESC);
```

```
array_agg
{car,food,car,food}
```

```
select string_agg(name, ';' order by amount DESC);
```

```
array_agg
car;food;car;food
```

Again note, in all these examples, we  went from N input rows to 1 output row.

## Group By Aggregate Functions

Now lets say that we want the sum of the amount of all items, but this time we want an individual sum for each category.

Here is our `items` table:

```
id,name,amount
1, food, 10
2, car, 20
3, food, 50
4, car, 100
```

And I want to know the sum for each category defined by its `name` - `food` and `car`. So we will be going from N input rows to Y output rows, where Y is the number of unique categories which is no longer fixed at 1 necessarily.

Lets first do this without group by aggregate functions. Let's just use the aggregate function we learned earlier:

```
select sum(amount) from items where name = 'food'
select sum(amount) from items where name = 'car'
```
```
sum
60

sum
120
```

Now this works and its because earlier as mentioned in the order of computation section we learned that the `where` clause fires prior to the `aggregate` function. So each time the `sum` command ran, it only saw a limited number of rows.

But this sucks too as it cost us 2 queries and we somehow knew the 2 category name's. This is unlikely to be the case in production.

Okay enough of beating around the bush now lets dive in to `group-by`.

```
select sum(amount) from items group by name
```

```
sum
60
120
```

Group-by went over all the N rows and divided them in to M smaller groups of rows, where each group has a unique value for the column group by ran on. Once the division in to groups is done, then the `sum`  aggregate function just like before ran across just the rows in that group.

Cool. We got the result we were looking for. We ran only 1 query, we didn't need to know the category names.

But what about showing other columns in the result besides sum. Well `Group-by` allows usage of columns in the `select` part if the columns have the same value for all the rows in each group across which the aggregate function is running. So in our case since we grouped on `name`, all the rows its grouping and producing sum on have the same `name`. This means we can put `name` in the select, along with the `sum`.

e.g.

```
select name, sum(amount) from items group by name
```

```
name, sum
foo, 60
car, 120
```

But we can not put a column which doesnt' have the same value for each row in the group. An example of that would be `id`. Think on this for a second. The sum function is running over a group of rows, now each row has a different `id` and if you ask to print the `id` next to the `sum` then it wont know which `id` to print as there maybe many rows, all with their own `ids`.

So, we can not do the following:

```
select id, name, sum(amount)
from items
group by name
```

```
ERROR:  column "items.id" must appear in the GROUP BY clause or be used in an aggregate function
```

So what can we do about this. This seems like a serious limitation. Lets add description to our table

```
id,name,amount, description
1, food, 10, for eating
2, car, 20, for driving
3, food, 50, for eating
4, car, 100, for driving
```

now we want to see the `sum` accross each category and also want to see `description` of each category. In theory this should work as the description is the same for every row in the group. Lets try it.

```
select name, sum(amount), description
from items
group by name
```

```
ERROR:  column "items.description" must appear in the GROUP BY clause or be used in an aggregate function
```

So DB engine rejects it. Even though we know that description is unique, the DB engine doesn't and rejects. So lets also add the extra unique column to that group syntax:

```
select name, sum(amount), description from items group by name, description
```

```
name,amount, description
food, 60, for eating
car, 170, for driving
```

This works and the results are still right. This is because now we made groups of rows where both `name` and `description` were unqiue. this worked out for us because description and name both were consistent.

We still cant do this for `id`. If we add id to the group by its going to make 4 unqiue groups as each row has different `id's` and our results will be wrong, even though we will be able to show `id` in the `select`.

So if we must see id, and we can not add it to the group function, then next thing to see it in output would be to use a different aggregate function which can show it.

```
SELECT array_agg(id), name, sum(amount) FROM items group by name;
```

```
array_agg, name, sum
{1,3}, food, 60
{2,4}, car, 170
```

Now this works as the DB engine is just showing ids of all the rows in the group its aggregating on. So the point here is that as long as you can tell the DB to get a value from accross all the rows in the group, then it will do what you ask. More examples which would work are:

```
SELECT avg(id), name, sum(amount) FROM items group by name;
SELECT max(id), name, sum(amount) FROM items group by name;
SELECT min(id), name, sum(amount) FROM items group by name;
SELECT sum(id), name, sum(amount) FROM items group by name;
SELECT string_agg(id::varchar, ';'), name, sum(amount) FROM items group by name;
```

All these work as they go over a number of rows and you are telling the DB how to produce the results accross multuple rows.

An alternate way to get uncollapsed results while using `group by`

```
select items.*, foo.sum from items inner join (
select name, sum(amount) from items group by name) as foo on foo.name = items.name
```

```
"id","name","amount","sum"
2,"car",20,120
3,"food",50,60
4,"car",100,120
1,"food",10,60
```

So to sum it up, `group-by` gave us something in this case the result of an aggregate function accross multiple rows, and it built those multiple rows by dividing all the rows it got in to smaller groups where all rows in each smaller group share a common value for a particular colunn, in our case that column being `name`. It also took something away from us. It still doesnt show the granularity and detail of each row. We still cant see details of each row like `id`.

For the official defination:
> The GROUP BY Clause is used to group together those rows in a table that have the same values in all the columns listed.

## Intro to Window functions

Okay so finally now we done with pre-reqs and have setup a problem we were unable to easily solve with aggregate function or group by aggregate functions.

Here is our `items` table

```
"id","name","amount"
1,"food",10
2,"car",20
3,"food",50
4,"car",100
```

and we want to see:
- the total sum accross all rows
- and total sum accross each category
- and we want to see all existing item rows

So we still want to get the ids of each row along with these sums we asked for. In other words we have been provided with N rows, we want to have N rows in the output along with 2 extra columns. We dont want to give up any row. I know this sounds greedy, but thats what we want and thats what window functions will help us do.

```
select *,
sum(amount) over () as total_sum,
sum(amount) over (partition by name) as category_sum
from items;
```

```
"id","name","amount","total_sum","category_sum"
2,"car",20,180,120
4,"car",100,180,120
1,"food",10,180,60
3,"food",50,180,60
```

It looks like we finally got what we were looking for. We gave N rows and got back 4 rows. We got our sums and lost nothing. This is a win-win. This is why we want to learn window functions.

So how did this work?

The first thing, we did is we used the `sum` aggegate function. This does exactly what you expect. It produces the sum of the `amount` column, but in this case its followed by an `over`. This makes it an aggregate window function, rather than just a regular aggregate function. In the `over` clause we specify how many rows should be considered in the `window` accross which the aggregate function is to run. So its kinda like `group-by`, but we are defining the group for each function seperately and calling that the function's window.

Within the first `over()` function, we didnt write anything, so that means inlcude all rows in the window frame. This gives us our first column which is the total sum accross all rows.

In our second `over()` we specify a `partition` inside the `over()` function. so now it will create a smaller partition of the rows and the sum function will run over just those rows. The `partition-by` takes column names and then looks at the current row for values of those columns. Then it goes and finds all rows with the same value for those columns. This process is done for each row. So each row get its own set of partitioned rows bases on each rows value for column - `name`. Then finally the sum is calculated over these specific rows. This gives us the sum for each category by its name name.

This wasnt in our example above, but to spice things up a but, you should know that we can further tune the window function by using window frames and filter. Both of them will further limit which rows are used in the window. These may be used with the `partition-by` or without it. They will be discussed shortly.

**An intersting trick**: if for any reason we do window functions, but then say that meehh i dont want to see all rows, I would like to give up resolution, as I only want to see row 1 row per unique name, well then we can do that by adding `distinct on`.

```
select distinct on (name) *,
sum(amount) over () as total_sum,
sum(amount) over (partition by name) as category_sum
from items;
```

```
id, name, amount, total_sum, category_sum
2 car 20  180 120
1 food  10  180 60
```

`Distinct-on` is a topic of its own and I recommend you to read this article I wrote about it here: http://sandeep45.github.io/postgresql/sql/rails/distinct/distinct_on/2018/07/22/distinct-vs-distinct-on.html. Briefly though, `distinct-on` is computed all the way in the end, so our window functions runs like noraml and we get all the data with all 4 rows and then `distinct-on` computes and keeps the first row with a unqiue value for the column which was provided to it. In our case it gets the first row where `name` is `car`, then skips the remaining rows where `name` is `car`.

Offical Defination:
> Window functions provide the ability to perform calculations across sets of rows that are related to the current query row

The case to learn window functions has been made. Hopefully you feel the same. With the case made, lets dive in to the details.

### Syntax of Window Function

Lets break down a window function in to smaller sections and see what its made of.

function_name([expression]) [filter (where filter_clause)] over {window_defination|window_name}

#### `function_name` looks like

`sum(column_name)` or `count(*)`

Various window functions are discussed shortly.

#### `filter_clause` looks like:

```
filter (where column_name = 'x')
```

these limit the rows where are provided to the window function.
they also only work for aggregate functions like `sum` and `count`. It **won't work** for general purpose window functions like lead, nth_value etc.

e.g.

```
select *,
sum(amount) over () as total_sum,
sum(amount) filter(where name = 'food') over () as food_sum,
sum(amount) filter(where name != 'food') over () as non_food_sum,
sum(amount) over (partition by name) as name_category_sum
from items;
```

```
"id","name","amount","total_sum","food_sum","non_food_sum","name_category_sum"
2,"car",20,180,60,120,120
4,"car",100,180,60,120,120
3,"food",50,180,60,120,60
1,"food",10,180,60,120,60
```

#### `window_defination` looks like:

```
[partition by expression [, more_expressions] ] [order by expression] [frame_clause]
```

#### `parition by` looks like

`partition by column_name [order by column_name] [rows|range between frame_start and frame_end]`

`partition-by` further refines all the rows to just the rows whose `column_name` matches the `column_name` of the current row.

`parition-by` can also accept an `expression`. In this case it divides the rows in to 2 group, one for which expresion resolves to true and the other where its false. e.g. `over (partition by amount > 90)`

`partition-by` Unlike a group by, doesnt collpase the other rows.

`order-by` is opitional and it wont matter for aggregate functions like `sum` and `count`. As no matter what order the rows are provided the result for `sum` and `count` is the same. But on the other hand for general purpose window functions like `rank` and `nth_value`, the `order` matters.

**Additionally take note:** Once an `order-`by` is provided, a `frame` is also implicity in action.

`frame_clause` is an even further reduction of the number of rows within the partition which are being considered. It filters the rows by taking them from either:

- the begining of the result set
- or the current row

and it takes row up to:

- the end of the result set
- or the current row

The rows `order` **matter**. Its the order which decides what comes first and what comes last, therefore without an order we can't have frame.

And always remem ber as **this one is not obvious** and **is the source of many bugs**:

- **without** an order, there is **no** frame in effect
- **with** an order, there **always** is a frame in effect
- if a frame is not explicitly defined, the **default frame** kicks in **–>** `range between unbounded preceding and current row`
- functions like `rank`, `nth_value` care for `order`, but `frames` don't have any effect on them

Threatening Speak: Ordering a `window` without defining the `frame` is equivalent to **developing vicious and undetectable bugs**.

Take a look at the break down below:

- sum(amount) over() **–>** all rows
- sum(amount) over(parition by name) **–>** all rows whose name matches the current rows name
- sum(amount) over(parition by name order by id) **–>** all rows whose name matches the current rows name and rows are produced ordered order by id and refined by the default frame
- sum(amount) over(parition by name order by id rows 2 preceding and 2 following) **–>** all rows whose name matches the current rows name and are produced order by id and refined by the provided frame of 5 rows, which are 2 following, current row and 2 preceding

#### `frame_caluse` loos like:

- {range | rows} frame_start
- {range | rows} between frame_start and frame_end

#### rangs and rows

- if you write `rows`, then the meaning of `current row` is actually the current row
- if you write `range` then the meaning of current row is actually the peer rows, where peer rows are defined as all the rows who have the same value the frame window is being sorted on.
- if you write `range` and use `current row` for `frame_end` then it means the last row of the peer rows
- if you write `range` and use `current row` for `frame_start` then it means the first row of the peer rows

#### fram_start and frame_end

values for `frame_start` and `fram_end` can be:

- `unbounded preceding` **–>** says start from the first row
- `N preceding` **–>** says start from N rows before current row
- `current row` **–>** says start/end at current row
- `N following` **–>** says end at N rows after current row
- `unbounded following` **–>** says end at the last row

you have 3 choices with them:

- if you dont write either of them. that is don't define either a start or an end. Then it means **–>** start in the begning and end at current row. Remember meaning of current row here is subject to wether `rows` or `range` was used.
- if you write just start part, then it means **–>** start where you specified and end at current row. This is because when you dont define `frame_end`, it means `current row`
- if you write both start and end the it means **–>** start at part 1 and end at part 2

In all cases keep in mind that meaning of `current row` will change between `current` and `peer row` depending on usage `row` and `range` respectively

A __wierd__ case here is if you write - `rows current row`. Since only item is given, then the rule is start where specified and end at current row. So its the same as saying `rows between current row and current row`. This is basically just 1 row.

And its counter part which is a bit less __wierd__ is to set a range between current row and curren row. In this case it could be 1 row or multiple, depending on how peer rows exist

```
select *,
count(*) over (partition by name range between current row and current row)
from items;
```

```
"id","name","amount","count"
2,"car",20,2
4,"car",100,2
3,"food",50,2
1,"food",10,2
```

Lets do some more examples. Lets consider the case when no frame is defined

```
select *,
sum(amount) over (order by name),
sum(amount) over (order by name range between unbounded preceding and current row)
from items;
```

```
"id","name","amount","sum","sum"
2,"car",20,120,120
4,"car",100,120,120
1,"food",10,180,180
3,"food",50,180,180
```

keep in mind that if order by was not present, then default will be all rows as current row will be last row of peer and peer will all rows

e.g.

```
select *,
sum(amount) over (),
sum(amount) over (range between unbounded preceding and current row)
from items;
```

```
"id","name","amount","sum","sum"
1,"food",10,180,180
2,"car",20,180,180
3,"food",50,180,180
4,"car",100,180,180
```

saying just start

```
select *,
sum(amount) over (order by name rows unbounded preceding),
sum(amount) over (order by name rows between unbounded preceding and current row)
from items;
```

```
"id","name","amount","sum","sum"
2,"car",20,20,20
4,"car",100,120,120
1,"food",10,130,130
3,"food",50,180,180
```

saying both start and end

```
select *,
sum(amount) over (order by name rows between unbounded preceding and 1 following)
from items;
```

```
"id","name","amount","sum"
2,"car",20,120
4,"car",100,130
1,"food",10,180
3,"food",50,180
```

Usage of Range vs Row changes the meaning of current row:

- In Row mode, current row means literally the current row
- In Range mode, current row means all peer rows which share the same value as seen by the order by operator

e.g.

```
select *,
sum(amount) over (order by name range between current row and current row),
sum(amount) over (order by name rows between current row and current row)
from items
```

```
"id","name","amount","sum","sum"
2,"car",20,120,20
4,"car",100,120,100
1,"food",10,60,10
3,"food",50,60,50
```

A practical use case for `range`:

You want a running sum, of sales per day. so your order by day. but now you have multiple sales per day. so your running sum will increase with every sale within that day. but you just want the sum for that day.
solution would be to use `range` over `rows` and sort in such a way that the 2 rows of the same day get same value for sorting.

#### `window_name` looks like:

`window foo as (partition by name)`

e.g. lets get the minimum and maximum of each group

```
select *,
min(amount) over win_by_name,
max(amount) over win_by_name
from items
window win_by_name as (partition by name);
```

```
"id","name","amount","min","max"
2,"car",20,20,100
4,"car",100,20,100
3,"food",50,10,50
1,"food",10,10,50

```

This is a nice short-hand, especially usefult when using the same window clause accross multiple functions as it can reduce typos.

### Kinds of Window Functions

There are two kind of window functions:

- General Purpose Window Functions **–>** These are unique to window functions and wont work without a window specified. Some of them work over the entire window and some only work over a frame of the window.

    e.g. rank, row_number, lag, lead, first_value, last_value etc.

- Aggregate Window Functions **–>** These are the same General Purpose window functions we use with group-by/having. These same functions when written followed by `over (window_expression | window_name)` become aggregate window functions and work only over the provided rows in the window.

    e.g. sum, count etc.

#### General Purpose Window Functions

Interesting distinction of these functions is that:

- they rely on order function
- some of them (first_value, last_value & nth_value) **do work** on a frame
- rest of them (rank, lag, ntile etc.) **do not work** on a frame

e.g.

Lets add two columns where:

- first column shows a row number for each item based on their `amount` with largest `amount` getting row number 1 and row with lowest amount getting the highest row number
- second column shows the value of the `amount` column of the previous row when the rows are grouped by their name and previous row is taken based on each rows `amount`, where the row with the highest `amount` comes first.

```
select *,
row_number() over (order by amount DESC) as row_number,
lag(amount) over (partition by name order by amount DESC) as last_amount
from items;
```

```
id, name, row_number, last_amount
4	car	100	1
2	car	20	3	100
3	food	50	2
1	food	10	4	50
```

__Important:__ Among the general purpose window functions - `first_value`, `last_value` & `nth_value` look at the `frame` and produce the result accordingly.

Offical wording:
> Note that first_value, last_value, and nth_value consider only the rows within the "window frame", which by default contains the rows from the start of the partition through the last peer of the current row. This is likely to give unhelpful results for last_value and sometimes also nth_value. You can redefine the frame by adding a suitable frame specification (RANGE or ROWS) to the OVER clause.

e.g. lets print the last rows amount

```
select *,
last_value(amount) over (order by id)
from items
```

now by default the frame clause is - `range between unbounded preceding and current row`
so we can re-write the query with the default frame so its more clear

```
select *,
last_value(amount) over (order by id range between unbounded preceding and current row)
from items
```

now for every row, the range is all the rows before it, till the current row. and since `range` instead of `rows`, then `current row` when used for `frame_end` will mean the last row in the peer rows.

so we dont get what we want. last value is the current row. well thats not what we want.

```
"id","name","amount","last_value"
1,"food",10,10
2,"car",20,20
3,"food",50,50
4,"car",100,100
```

Therefore we can see here that frame is playing an important role here. Lets use right frame to get what we want

```
select *,
last_value(amount) over (order by id rows between unbounded preceding and unbounded following)
from items;
```

now the frame is all rows before and after, as we are not stopping at the current row.

ouput now is what we want

```
"id","name","amount","last_value"
1,"food",10,100
2,"car",20,100
3,"food",50,100
4,"car",100,100
```

On the other hand, the remaining general purpose window functions like `ranking`, `lead`, `ntile` etc. dont care of the frame inside the window. They work on the entire window and can not work on a specific frame.

e.g. all 4 of these sql commands produce the same result and in my opinion only the 1st one is right, the remaining 3 are not correct.

```
select *, row_number() over (order by id) from items;
select *, row_number() over (order by id rows between current row and unbounded following) from items;
select *, row_number() over (order by id rows between unbounded preceding and current row) from items;
select *, row_number() over (order by id rows between 1 preceding and 1 following) from items;
```

"id","name","amount","row_number"
1,"food",10,1
2,"car",20,2
3,"food",50,3
4,"car",100,4

Below are my favorite and most commonly used general purpose window functions:

- **row_number** - looks at all rows (frame doesn't matter) and gives each row a number starting from 1
- **rank** - looks at all rows (frame doesn't matter) and gives each row a number. will produce duplicate ranks and then appropriate gaps if multiple rows have same value on which they are being sorted.
- **dense_rank** - looks at all rows (frame doesn't matter) and gives each row a number. will produce duplicate ranks but no gaps if multiple rows have same value on which they are being sorted.
- **ntile(bucket_size)** -  looks at all rows **(frame doesn't matter)** and gives each row a number with an attempt to divide all rows equally in to the bucket size provided
- **lag(column_name)** -  looks at the row behind current row **(frame doesn't matter)** and gives the previous rows value for provided column
- **lead(column_name)** -  looks at the row after current row **(frame doesn't matter)** and gives the next rows value for provided column
- **first_value(column_name)** - **looks at the frame** and gives the value of provided column in the first row of the frame
- **last_value(column_name)** - **looks at the frame** and gives the value of provided column in the last row of the frame
- **nth_value(column_name, number)** - **looks at the frame** and then gives value  of provided column at the provided row number as long as its in the frame or you will get NULL.

Reminder: all of the General Purpose Window functions depend and use the order by to figure out whats before and after each row. Without an `order-by` they won't work.

All General Purpose window functions are documneted here - https://www.postgresql.org/docs/9.1/functions-window.html

#### Aggregate Window Functions

These are just like regular aggregate functions, but follwed by an `over` statement so they work over a window of rows rather all rows or grouped rows.

e.g. sum, count, min, max etc.

An interesting note about these is that:

- they dont dont care for order
- when no order is applied, they work fine accross entire window, with no frame coz frame doesnt exist without order
- when order is applied, they honor the frame
- its the frame, which can make function like `sum` return `total_sum` vs `running_sum`

**Very Important**: Let this point mentioned above about `frames` sink in. All Aggregate functions can work on a `frame` and also without a `frame`. Its all about whether a `frame` was provided. And this inlcudes an implicit `frame` which comes with an `order`. So be careful when you put an `order` to a funcion like `sum` and `average`. You may think, hey why would the result of summing a bunch of records, or finding their average change if we change the order we porcess them in. Well its because **order brings an implicit frame** and **frames decide how many rows are made available** to the window function.

e.g.

```
select *,
avg(amount) over () as avg,
avg(amount) over (order by id rows between unbounded preceding and current row) as running_avg
from items;
```

```
"id","name","amount","avg","running_avg"
1,"food",10,45,10
2,"car",20,45,15
3,"food",50,45,26.6666666666666667
4,"car",100,45,45
```

```
select *,
count(*) over () as avg,
count(*) over (order by name range between unbounded preceding and current row) as running_avg,
count(*) over (order by name) as running_avg
from items;
```

```
"id","name","amount","avg","running_avg","running_avg"
2,"car",20,4,2,2
4,"car",100,4,2,2
1,"food",10,4,4,4
3,"food",50,4,4,4
```

Some aggregate functions of interest are:

- array_agg(column_name) **–>** gives an array of the values of column_name accross all rows in the window
- string_agg(column_name, delimter) **–>** gives an string with values of column_name seperated by delimer
- count(*) **–>** number of rows
- count(expression) **–>** number of rows for which expression is non-null
- sum(column_name) **–>** sum of column acrross rows
- avg(column_name) **–>** avg of column acrross rows
- min(column_name) **–>** min of column acrross rows
- max(column_name) **–>** max of column acrross rows

You will find a list of aggregte funtions here: https://www.postgresql.org/docs/9.1/functions-aggregate.html

## Conclusion

Hopefully now you can appreciate window functions. Now you know you can get all the results and also get their analytical results (have the cake and eat it too!).

In my opinion to practice is to learn and this is where I practiced: https://academy.vertabelo.com/course/window-functions
