#kdp #quant #q 
Table query language like #sql.

# Resources
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

Select all :
- `qSQL` is like `SQL` but does not require a `*` to select all columns
```q
select from tableName
```

Select specific columns :
```q
select col1, col3, col2 from tableName
```


## Filter
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
