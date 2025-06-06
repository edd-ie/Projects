#kdp #quant #Q 
Table query language like #sql.

# Resources
- [Q for Mortals - book](https://code.kx.com/q4m3/)
- [QSQL query templates](https://code.kx.com/q/basics/qsql/)
- [Reference card](https://code.kx.com/q/ref/)
- [Datatypes](https://code.kx.com/q/ref/#datatypes)


# Data structures
## Lists
Similar to vector
- Resource - [code.kx - q4m - lists](https://code.kx.com/q4m3/3_Lists/)

`simple list`  :
List has items of same type
- Creating from a table
```q
// Extracting a column
vtsfares:select fare from trips where date = 2009.01.01, vendor=`VTS

// Creating the list
fares: vtsfares`fare

// confirm if it is a list
type fares  // result 9h
// 0 < number < 20: a simple list
```

`general list` :
List with items of different types - `0h`

- `Pair` : 2 simple list
```q
// Extracting a column
vtsfares:select fare from trips where date = 2009.01.01, vendor=`VTS
vtstips:select tip from trips where date = 2009.01.01, vendor=`VTS

// Creating the list
fares: vtsfares`fare
tips: vtstips`tip

namePair:(fares;tips)
```

- `empty` list
```q
empty:()
```

list of 1 item:
```q
enlist 1023
(), 120332 // alternative
```

- `mixed` list - joining entities of different types with the comma operator
```q
general:2018.01.01,102,`hello,enlist "world"
// 2018.01.01 102 `hello "world
```


Amending a list:
A simple list can be indexed into using the `@` operator:
```q
2* til 5  //0 2 4 6 8
@[sampleFares;(2*til 5)]
```

The `@` operator can be applied with further arguments so that the list can be altered. Below we replace the items at positions `2*til 5` with `99f`.

```q
// index into sampleFares
// using list of indexes (2*til 5)
// assign these values - :
// the value 99f
@[sampleFares;(2*til 5);:;99f]
```

Below we use `+` instead of `:` – instead of replacing the items, we add `99f` to them.
```q
@[sampleFares;(2*til 5);+;99f]
```

The above is not a persistent change - it will make a copy of the `fares` list with a single value changed and display the result at the terminal, but there is no change to the `fares` list.

To persist the change, prefix the name of the list with a back-tick; or assign the result to a name:
```q
test:@[fares;(2*til 4);:;0Nf]
test
@[`fares;(2*til 4);:;0Nf]
fares
```

Extend a list by appending to to it using the [Join operator `,`](https://code.kx.com/q/ref/join/).
```q
fares,:12.34
-10#fares    // inspect the end of the list to see the appended value
```

_Amend the fares list to replace the null values to be equal to the average value._
```q
@[fares;where null fares;:;avg fares]
```


## Dictionary
[Dictionaries](https://code.kx.com/q/basics/dictsandtables/) are first-class objects in q.
- `Hashmaps`

Make a dictionary from a list of keys and a list of values :
- Use the [Dict operator `!`](https://code.kx.com/q/ref/dict/)
```q
d:`a`b!0 1
d
// a| 0
// b| 1
```

access a value:
```q
d[`a]
// 0
```

Update existing values:
```q
d[`a]:2
d
// a| 2
// b| 1
```

Add new key-value:
```q
d[`c]:3
d
// a| 2
// b| 1
// c| 3
```

Combine to another dictionary :
1. Add values of two dictionaries
```q
d1:`a`b`c`d!5 6 7 8
d+d1 // add values for common keys
// a| 7
// b| 7
// c| 10
// d| 8
```

2. Join two dictionaries, prioritising values from the right-hand dictionary
```q
// Typical application is updating a snapshot with deltas
d,d1 // catenation - updates values for common keys, inserts new keys. .
// a| 6
// b| 7
// c| 7
// d| 8
```

Get key / value :
```q
key k
value k
```

Dictionary of lists:
```q
x:`a`b`c!(til 5; 2*til 5; 3*til 5)

// Random vals
dict:`a`b`c!(3?10i;3?10i;3?10i)

// Add a new key, `d` with double the values of key `a`.
x[`d]:(2*x[`a])
```


## Tables
Any list of 'like dictionaries' is a table. 

Table from a list of like dictionaries:
-  Multiple dictionaries with the same key 
- `(dict1; dict2)`
```q
(`a`b!0 1;`a`b!2 3)
// a b
// ---
// 0 1
// 2 3
```

Table using [table notation](https://code.kx.com/q/kb/faq/#table-notation) :
```q
([]a:0 2;b:1 3)
```

Table from  [column dictionaries](https://code.kx.com/q/kb/faq/#flip-a-column-dictionary).  
- Making a table from dictionaries
```q
flip `a`b!(0 2;1 3)  // Transposing data

dict:`a`b`c!(3?10i;3?10i;3?10i)
tab:flip dict
```

Adding table :
```q
([]a:0 2;b:1 3)+([]a:4 5;b:6 7)
// a b
// ---
// 4 7
// 7 10
```

Tables can be keyed to create a [keyed table](https://code.kx.com/q/kb/faq/#keyed-tables).
1. Specify key columns with the [`xkey` keyword](https://code.kx.com/q/ref/xkey/)
```q
k:`a xkey ([]a:0 2;b:1 3)
// a| b
// -| -
// 0| 1
// 2| 3

tabKeyed:`b xkey tab2   // Making b column the key in tab2
```
2. Specify key columns in the table notation.
```q
([a:0 2]b:1 3)
([a:0 2;b:1 3]c:4 5)
// a b| c
// ---| -
// 0 1| 4
// 2 3| 5
```

_A keyed table is a dictionary where both key and values are tables:_

Get key / value :
```q
key k
value k
```

Join a table to the bottom of another:
```q
tabP:(tab,tab)
```

# null
Check for null 
- returns: `0b / 1b`
```q
any null fares
```

This is exactly equivalent to using `any[null[fares]]` 

The [`null` keyword](https://code.kx.com/q/ref/null/) flags nulls.
- Return indexes
```q
where null fares
```
# Casting
 `$` used to [cast](https://code.kx.com/q/ref/cast/)
```q
`float$1 2 //using it's symbol name 
"f"$1 2  //using it's character letter
9h$1 2   //using it's short value
`long$() //list of type long
```

# Subset
Use `n sublist name` to get  1st `n`  values of `name`
- `-n sublist name` to get  last `n`  values of `name`
```q
10 sublist fares
-10 sublist fares

// get the second 10 elements in the list
sub:20 sublist fares;
-10 sublist sub
//***Alternatives***
-10 sublist 20 sublist fares  // ...1
10 10 sublist fares           // ...2
```

<font color=#fd6206>Note</font> - number of elements it returns is _capped_ at the size of the list that it operates on

[Take operator `#`](https://code.kx.com/q/ref/take/) returns exactly the number of items you specify:
```q
count 10000000 # fares          // 10000000
count 10000000 sublist fares    // 124208
```

# Sorting
`asc`
- Sort in ascending order with duplicates

`asc distinct`
- Sort in ascending order, only unique

```q
sortedFares:asc fares
sortedUniqueFares:asc distinct fares
```

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

Access values in a list with indexing
```q
namelist[3] // value at index 3, starts from 0
```

_Use indexing to find the middle value in the `sortedFares` list. _
```q
size:"i"$((count sortedFares)%2)
sortedFares[size]

// or
sortedFares [`long$(count sortedFares)%2]
```

first & last values:
```q
//first
first listN
1 sublist listN

// last
last listN
-1 sublist listN
```
## till
Generates numbers or fetches value till the specified amount
```q
til 5  // 0 1 2 3 4

// 1st 5 values
fare[til 5]

// Extract the 11th to the 20th elements from the fares list
fares[10 + til 10]
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

## Random
Resource - [Roll, Deal, Permute](https://code.kx.com/q/ref/deal/)

Roll operator `?` :
- Get `n` random records from a list: `n?list`
```q
record:10?recordlist;
```
- Get `n` values:
```q
3?1 2 4 6 3 2
// 1 6 4
```


## Operators
Power:
```q
a xexp 2 // a^2
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
- meaning `join on`
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

## Bi - Temporal joins
Joins related to `time`

### As-of join
`aj[matching columns;t1;t2]` - [aj join](https://code.kx.com/q/ref/aj/)

Given the data we have, we could ask what were the latest pick-ups for each vendor, as of a particular time.
- Creates a temporary time table with a minimum date time for each vendor

Example
_3 reports of individuals who have lost their phone or wallet who were picked up shortly before the time who said how many passengers were in the taxi. Which vendor were they riding with?_
```q
// Records of time of travel
timetab:([] passengers:1 2 3; event_time:2009.01.06D03:30:00+00:30*til 3)

// look up the table to find out what was the last trip taken at each of the times above
aj[`passengers`event_time;timetab;
	select passengers, event_time:pickup_time, vendor, pickup_time 
	from jan09]
```

# Overloading
Resource - [Overloaded glyphs](https://code.kx.com/q/ref/overloads/)


# Functions
Functions are called with the arguments in square brackets `[]`
- unary (single argument) function, we can omit the square brackets
- Reference - [Function notation](https://code.kx.com/q/basics/function-notation/)
```q
max[10 11 12]  // functional notation
max[10;11;12]  // functional notation
max 10 11 12   // infix notation
```

Creating a function using `{}` :
- index the last `}` q is weird.
```q
f:{} // empty function

// Explicit parameters
fx:{[x; y]
	a: x%y;
	b:1.609*a;
	
	:b; // return
	}

// Implicit parameters
gx{
	a: x%y;
	1.609*a // return
	}
```

Print within the function:
```q
fx{[x; y]
	show a: x%y;  // Printing a
	b:1.609*a;
	
	:a; // return
	}
```
