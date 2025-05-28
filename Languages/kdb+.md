#kdp #quant #q 
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

## Select
If you have used SQL, you will find the syntax of qSQL queries very similar.

### Select all
- `qSQL` is like `SQL` but does not require a `*` to select all columns
```q
select from tableName
```

### Select specific
- Table is partitioned using the <font color=#fd6206><strong>1st column named.</strong></font>
```q
select col1, col3, col2 from tableName
```

### custom named column/s:
- Assign a name using `column_name:value`
```q
select duration, total: fare + tip, fare, tip from trips
```

### Indexing
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


# Aggregations
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
