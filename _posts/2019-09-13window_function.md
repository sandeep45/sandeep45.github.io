https://www.postgresql.org/docs/9.1/static/tutorial-window.html

there is one output row for each row in the table

OVER clause causes it to be treated as a window function and computed across an appropriate set of rows.

A window function call always contains an OVER clause directly following the window function's name and argument(s). This is what syntactically distinguishes it from a regular function or aggregate function.

The PARTITION BY list within OVER specifies dividing the rows into groups, or partitions, that share the same values of the PARTITION BY expression(s).

For each row, the window function is computed across the rows that fall into the same partition as the current row.

You can also control the order in which rows are processed by window functions using ORDER BY within OVER. (The window ORDER BY does not even have to match the order in which the rows are output.)

e.g.
sum
avg
count
rank

the rank function produces a numerical rank within the current row's partition for each distinct ORDER BY value, in the order defined by the ORDER

window function sees the subset of rows produced after filtering with things like where, group, having etc.

you can have multiple window functions, they all work over the orignal same filtered rows

order by can be omitted if order is not important to the window function
parition by can also be omitted to get just one partition containing all rows.

concept called - window frame
normally, i.e when no order is present
window frame has all the rows in the partiion
sometimes, i.e. when a order by is present
window frame is all the rows above the current row

There is another important concept associated with window functions: for each row, there is a set of rows within its partition called its window frame. Many (but not all) window functions act only on the rows of the window frame, rather than of the whole partition. By default, if ORDER BY is supplied then the frame consists of all rows from the start of the partition up through the current row, plus any following rows that are equal to the current row according to the ORDER BY clause. When ORDER BY is omitted the default frame consists of all rows in the partition

```
select * from impressions limit 10;

select count(*) from(select * from impressions limit 10) as first_10_impressions;

select date(created_at) from impressions limit 10;

select date(impressions.created_at), count(*) from impressions group by date(impressions.created_at);

select date(impressions.created_at), count(*) over(PARTITION BY date(impressions.created_at)) as x from impressions;

select distinct domain, sum(bid_price_micros_usd/1000) over(PARTITION BY domain) from impressions;
select distinct domain, avg(bid_price_micros_usd) over(PARTITION BY domain) from impressions;
select distinct domain, count(*) over(PARTITION BY domain) from impressions;
select distinct domain, rank() over(PARTITION BY domain) from impressions;

select domain, id, rank() over(partition by domain ORDER BY id ASC) from impressions;
select domain, bid_price_micros_usd, rank() over(partition by domain ORDER BY bid_price_micros_usd ASC) from impressions;

select * from accounts;

select distinct date(created_at), domain, count(*) over (PARTITION by date(created_at) ORDER BY date(created_at) ASC) as date_count, count(*) over (partition by domain) as domain_count from impressions;

select domain, rank() over(order by id ASC) from impressions limit 10;

select distinct domain,
sum(1) over() as running_imp_count
from impressions;


select distinct domain,
count(*) over(partition by domain) as domain_imp_count,
count(*) over(order by domain ASC) as running_imp_count,
count(*) over() as total_imp_count
from impressions order by domain ASC;
```

```SQL
select distinct date(rx_timestamp),
count(*) over(partition by date(rx_timestamp)) as count_per_day,
count(*) over(order by date(rx_timestamp) ASC) as running_count,
count(*) over() as total_count
from impressions order by date(rx_timestamp) ASC;
```

select * from(
	select distinct date(rx_timestamp),
					count(*) over(partition by date(rx_timestamp)) as count_per_day,
					count(*) over(order by date(rx_timestamp) ASC) as running_count,
					count(*) over() as total_count
	from impressions order by date(rx_timestamp) ASC
	) as stats
where stats.count_per_day > 2800;



http://www.postgresqltutorial.com/postgresql-window-function/
Given impressions give me delta_impressions of that day and total impressions count per day

window functions like `lag` and `lead` can be used to get the value before and after respectively

window functions like `first_value`, `last_value` and `nth_value` can be used to get value at a specific number

window functions like `rank` will give the position of the current row when compared to the window data. two items with same value get same number and then next number is skipped. `dense_rank` does the same but doesn't skip a number. `row_number` does the same thing of giving each row a number, but doesn't compare the rows with each other so if values are same, they wont get same number, instead consecutive numbers.




select date(created_at),
       count(*) over() as total_impressions,
       count(*) over(partition by date(created_at)) as delta_impressions,
       count(*) over(order by date(created_at) ASC) as total_impressions
from impressions;


## Process of getting account basics domain along with an account

```SQL
select accounts.id as account_id,
account_basics.id as account_basics_id,
account_basics.domain as account_basics_domain
from accounts
inner join account_basics on account_basics.account_id = accounts.id
order by accounts.id;

select distinct accounts.id as account_id,
account_basics.id as account_basics_id,
account_basics.domain as account_basics_domain
from accounts
inner join account_basics on account_basics.account_id = accounts.id
order by accounts.id;

select distinct on(accounts.id) accounts.id as account_id,
account_basics.id as account_basics_id,
account_basics.domain as account_basics_domain
from accounts
inner join account_basics on account_basics.account_id = accounts.id
order by accounts.id;

select accounts.id, first_value(account_basics.domain) over(partition by account_basics.account_id)
from accounts
inner join account_basics on account_basics.account_id = accounts.id
order by accounts.id;

select distinct accounts.id, first_value(account_basics.domain) over(partition by account_basics.account_id)
from accounts
inner join account_basics on account_basics.account_id = accounts.id
order by accounts.id;

select distinct accounts.id,
first_value(account_basics.id) over(partition by account_basics.account_id) as account_basics_id,
first_value(account_basics.domain) over(partition by account_basics.account_id) as account_basics_domain
from accounts
inner join account_basics on account_basics.account_id = accounts.id
order by accounts.id;

select distinct accounts.id,
first_value(account_basics.id) over(partition by account_basics.account_id order by account_basics.id ASC) as account_basics_id,
first_value(account_basics.domain) over(partition by account_basics.account_id order by account_basics.id ASC) as account_basics_domain
from accounts
inner join account_basics on account_basics.account_id = accounts.id
order by accounts.id;

select distinct accounts.id, first_value(account_basics.id) over w, first_value(account_basics.domain) over w
from accounts
inner join account_basics on account_basics.account_id = accounts.id
window w as (partition by account_basics.account_id order by account_basics.id ASC)
order by accounts.id;
```

## Deleting 1 at a time

select * from impressions order by id limit 10;
delete from impressions where id in (select id from impressions order by id limit 1)


## temp tables
name them different, otherwise they will shadow the main table

select * into temp sandeep from impressions limit 20;
select count(*) from sandeep; // 20
select * into sandeep from impressions limit 10;
select count(*) from sandeep; // 20
// restart connection
select count(*) from sandeep; // 10
drop table sandeep;

select into will make the table in HEAP.
select * into x from accounts;
to get indexes instead create table and then do insert into
insert into x
select * from accounts;

## proof that temp tables created via `select into` are slow and dont use index

explain select * from impressions where auction_id = '68797aa8-b9eb-47a0-a163-f097faf2a037'
select * into temp sandeep from impressions;
select count(*) from sandeep;
explain select * from sandeep where auction_id = '68797aa8-b9eb-47a0-a163-f097faf2a037'

to solve it we can add index on the temp table and then do the query

CREATE UNIQUE INDEX index_sandeep_on_auction_id ON sandeep USING btree (auction_id)
explain select * from sandeep where auction_id = '68797aa8-b9eb-47a0-a163-f097faf2a037'

from my anecdotal research doesn't seem like analyze was needed to increase performance and to leverage the added index.
analyze verbose sandeep

SET enable_seqscan = OFF;
-- Run our query with EXPLAIN
SET enable_seqscan = ON;

