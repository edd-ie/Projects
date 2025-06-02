#kdp #quant #Q 
Table query language like #sql.

# Resources
- [Q for Mortals - book](https://code.kx.com/q4m3/)
- [QSQL query templates](https://code.kx.com/q/basics/qsql/)
- [Reference card](https://code.kx.com/q/ref/)
- [Datatypes](https://code.kx.com/q/ref/#datatypes)

# Tables
See available table in database:
```q
tables[]
```

Inspect/view table contents: 
```q
tableName
```

## Table data
Count number of records :
```q
count tableName
```

List column names :
```q
cols tableName
```

Get table meta data:
- `c`: column name
- `t`: column [type](https://code.kx.com/q/ref/#datatypes)
- `f`: [foreign keys](https://code.kx.com/q/wp/foreign-keys/)
- `a`: [attributes](https://code.kx.com/q/basics/syntax/#attributes): modifiers applied for performance characteristics (like `s` = sorted)
```q
meta tableName
```

# Select
If you have used SQL, you will find the syntax of qSQL queries very similar.

## Select all
- `qSQL` is like `SQL` but does not require a `*` to select all columns
```q
select from tableName
```

## Select specific
- Table is partitioned using the <font color=#fd6206><strong>1st column named.</strong></font>
```q
select col1, col3, col2 from tableName
```

## custom named column/s:
- Assign a name using `column_name:value`
```q
select duration, total: fare + tip, fare, tip from trips
```

# Count
Count number of records meeting a condition:
```q
count select from jan09 where passengers > 5
// 48821
```

## Indexing
A virtual column `i` maps to a record index in the table. 
- A simple aggregation can be obtained by taking the count of this virtual column.
```q
select count i from trips 
	where date = min date, passengers = 2

// On the earliest date, how many trips had fewer than two passengers?
select count i from trips
    where date = min date, passengers < 2
```


# Filter
Just as in SQL, table results can be filtered by expressions following a `where`. 
- Multiple filter criteria separated by `,` are evaluated starting from the left.
- Ordering is important for efficiency
```q
// Option1 
select date, month, vendor, passengers, fare, tip 
	from trips
	where date = min date, tip > 20

// Option2
select date, month, vendor, passengers, fare, tip 
	from trips 
	where tip > 20, date = min date
```

`Option1` runs faster. 
- The `trips` table is a partitioned on the `date` column, which is the first filter in the [Where phrase](https://code.kx.com/q/basics/qsql/#where-phrase). 
- Filtering on the partition column value first minimizes the number of directories to read.

## Columnar based filtering
<font color=#fd6206><strong>Querying a partitioned table</strong></font> 
When querying a partitioned table, the first `Where` subphrase should select from the value/s used to partition the table.
- Otherwise, `kdb+` will (attempt to) load into memory all partitions for the column/s in the first subphrase.

## Range
Use keyword  [within](https://code.kx.com/q/ref/within/) to filter data in a range.
- The filter column must be sortable.
```q
select from trips where date within 2009.01.10 2009.01.31
```

## Filter by
Nested queries are commonly required in SQL where filter criteria require aggregations in the context of some other column. 
- For example, getting all records where the ride duration is less than the average for that taxi’s vendor.
```q
// Get the average duration per vendor and save resulting table in a variable
resBy: select avgDuration:avg duration by vendor from jan09

// Using 'lj' to join the average duration column to our table
// Don't worry about this now, joins are discussed in a later section
select from jan09 lj resBy where duration < avgDuration
```
The syntax of `fby` is `(aggregation;data) fby group`, as query below. Compare the above statement to how it was done without `fby`.
```q
select max fare from jan09 where duration < (avg;duration) fby vendor
```

Examples:
```q
// Which payment type produces the highest average tip when only trips with a fare larger than the average for each vendor is considered?
ax:select avg tip by payment_type from jan09
	where (fare > (avg;fare) fby vendor);
show select payment_type from ax where tip = max tip;

// Which vendor has the largest number of trips when only considering trips shorter than the average duration for each vendor? 
ay:select trips:count i by vendor from jan09 
	where (duration < (avg;duration) fby vendor)
show select vendor from ay where trips = max trips
```

## Performance
To time performance of a query in terms of `time(t)` or `space(s)` or both
- Precede the query with `\ts`
```q
// Option1
\ts select date, month, vendor, passengers, fare, tip 
from trips
where date = min date, tip > 20

// Option2
\ts select date, month, vendor, passengers, fare, tip 
from trips 
where tip > 20, date = min date
```
Results :
```txt
t  s
34 794016
4027 1725040
```
_Option1 is fast and more space efficient than Option2_


# Assignment
Store the result of a query in a `variable`:
- Syntax `var:query`
```q
// Use keyword 'within' to filter the date
jan09:select from trips where date within 2009.01.10 2009.01.31

// Check how many records are in the filtered table
count jan09
```


# Grouping
Arrange the result using specific columns.
- keyword `by`
- When used without an aggregation, `by` returns a list of values from the selected column
```q
select fare by vendor from jan09
select fare by vendor,duration from jan09 
```
- grouping is best used with aggregation functions
```q
select sum fare, sum tip by vendor from jan09

// Get the number of records per day in the filtered table
select count i by date from jan09

// What is the highest tip and average tip per payment_type?
select Highest:max tip, Average:avg tip by payment_type from jan09
```


# Compare
Compare the result of 2 queries if equal using `~`
- `0b` = `false`
- `1b` = `true`
```q
query1~query2
// 0b
```


# Update

Use `update` to change the data.
```q
// Update maximum number of passengers to be 5 
// Save changes by reassigning back to jan09
jan09: update passengers: 5 from jan09 where passengers > 5
```

## Add column
```q
// add a column with the weighted average fare per passenger:
jan09:update wAvgfare:passengers wavg fare from jan09

meta jan09 //new column has been added to the end of the table
```


# Delete
Use `delete` with  condition for specific without for general
```q
count jan09  //number of records before deleting rows

// Always reassign to save changes
jan09:delete from jan09 where duration=00:00:00.000
count jan09  //number of records after deleteing rows 
```

<font color=#ff2800>nyi</font> - _not yet implemented_ error : query doesn't exits
```q
jan09:delete vendor from jan09 where month=2009.01
```


# Time
Time series analysis / temporal arithmetic
[Data types - [12, 19]](https://code.kx.com/q/basics/datatypes/)

_The `pickup_time` column in the data has a type of `timestamp`. convert the `pickup_time` values to their `minute` values (including hours and minutes), and group the data based on this time frame._
```q
select pickup_time, pickup_time.second, pickup_time.minute, pickup_time.hh from jan09
```

  
Aggregate further on time by using `xbar` to bucket the minutes into hours. Group the minutes into 60-unit buckets to produce hours:
```q
select count i by 60 xbar pickup_time.minute from jan09 
	where date = 2009.01.10
// Filter by hour

// Show the largest tip for each 15-minute timespan during the month of January.
select  max tip by 15 xbar pickup_time.minute from jan09 
	where date > 2008.12 and date < 2009.02

// Break this information down by vendor.
select  max tip by 15 xbar pickup_time.minute, vendor from jan09 
	where date > 2008.12 and date < 2009.02
```


# Math

## Order  of operation
#Q **evaluation is from right-to-left**.  This simple rule holds everywhere; there are <font color=#ffcba4><strong>no priorities</strong></font>.
```q
100*2+5
// 700
```

To impose and order use `()` 

## xbar
Rounds down to the nearest divisor 
```q
3 xbar 1 2 3 4 5 6 7 8 9 31
// 0 0 3 3 3 6 6 6 9 30
```

## Aggregations
Analytics can be run without performing the filter query again. 

[`sum`](https://code.kx.com/q/ref/sum/) is one of many built-in aggregations. Other built-in aggregations include, but are not limited to

- [`avg`](https://code.kx.com/q/ref/avg/#avg) - average (mean)
- [`med`](https://code.kx.com/q/ref/med/) - median
- [`min`](https://code.kx.com/q/ref/min/) - minimum value
- [`max`](https://code.kx.com/q/ref/max/) - maximum value
- [`count`](https://code.kx.com/q/ref/count/) - number of values

Reference: [Mathematics and statistics](https://code.kx.com/q/basics/math/)
```q
// sum of both the `fare` and `tip` columns. 
select sum fare, sum tip from jan09

// Calculate the minimum and maximum tip from the `jan09` table.
select Max:max tip, Min:min tip from jan09
```


# Keys
A table can be keyed in a number of ways, however the easiest is to use the [`xkey`](https://code.kx.com/q/ref/keys/#xkey) function
```q
`date xkey table //we are keying on date 
1!table          //we are keying on the first column 
3!table          //we can key on N number of columns
```

## Unkey
Remove key from table:
```q
0!table
```


# Joins
Reference  -  [Joins | Basics - kdb+ and q documentation](https://code.kx.com/q/basics/joins/)

## Left join
Left table takes precedence , keyword `lj` -  [left join](https://code.kx.com/q/ref/lj/)
For each record in `x`, the result has one record with the columns of `y` joined to columns of `y`:
- if there is a matching record in `y`, it is joined to the `x` record; common columns are replaced from `y`.
- if there is no matching record in `y`, common columns are left unchanged, and new columns are null
The `lj` operator requires that at least the right hand table argument be keyed.
- unkeyed left = unkeyed join.

```q
// select date and precipitation from the weather table
// key the result on date
// join to the unkeyed table jan09C (0! unkeys the table)
jan09W:jan09C lj `date xkey select date, precip from weather 
jan09W

jan09W:jan09C lj select avg precip by date from weather //using the by clause to key
```

<font color=#fd6206><strong>Note :</strong></font> both have same columns ,left table will be overridden by right table