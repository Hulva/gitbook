## Common
```
SHOW DATABASES;
SHOW TABLES;
```

## Create Database
`CREATE DATABASE|SCHEMA [IF NOT EXISTS] <database name>`

## Drop Database
`DROP DATABASE StatementDROP (DATABASE|SCHEMA) [IF EXISTS] database_name 
[RESTRICT|CASCADE];`

## Create Table
```
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.] table_name
[(col_name data_type [COMMENT col_comment], ...)]
[COMMENT table_comment]
[ROW FORMAT row_format]
[STORED AS file_format]
```
### Example
```
hive> CREATE TABLE IF NOT EXISTS employee ( eid int, name String,
salary String, destination String)
COMMENT ‘Employee details’
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ‘\t’
LINES TERMINATED BY ‘\n’
STORED AS TEXTFILE;
```

## Load Data
```
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename 
[PARTITION (partcol1=val1, partcol2=val2 ...)]
```

### Example
sample.txt
```
1201  Gopal       45000    Technical manager
1202  Manisha     45000    Proof reader
1203  Masthanvali 40000    Technical writer
1204  Kiran       40000    Hr Admin
1205  Kranthi     30000    Op Admin
```
```
hive> LOAD DATA LOCAL INPATH '/home/user/sample.txt'
OVERWRITE INTO TABLE employee;
```

## Alter Table
```
ALTER TABLE name RENAME TO new_name
ALTER TABLE name ADD COLUMNS (col_spec[, col_spec ...])
ALTER TABLE name DROP [COLUMN] column_name
ALTER TABLE name CHANGE column_name new_name new_type
ALTER TABLE name REPLACE COLUMNS (col_spec[, col_spec ...])
```
Example
```
hive> ALTER TABLE employee RENAME TO emp;
hive> ALTER TABLE employee CHANGE name ename String;
hive> ALTER TABLE employee CHANGE salary salary Double;
hive> ALTER TABLE employee ADD COLUMNS ( 
dept STRING COMMENT 'Department name');
hive> ALTER TABLE employee REPLACE COLUMNS ( 
eid INT empid Int, 
ename STRING name String);
```

## Drop Table
`DROP TABLE [IF EXISTS] table_name;` <br>
Example <br>
`hive> DROP TABLE IF EXISTS employee;` <br>

## Adding a Partition 
```
ALTER TABLE table_name ADD [IF NOT EXISTS] PARTITION partition_spec
[LOCATION 'location1'] partition_spec [LOCATION 'location2'] ...;
partition_spec:
: (p_column = p_col_value, p_column = p_col_value, ...)
```
Example
```
hive> ALTER TABLE employee
> ADD PARTITION (year=’2013’)
> location '/2012/part2012';
```

## Renaming a Partition
```
ALTER TABLE table_name PARTITION partition_spec RENAME TO PARTITION partition_spec;
```
Example
```
hive> ALTER TABLE employee PARTITION (year=’1203’)
   > RENAME TO PARTITION (Yoj=’1203’);
```

## Dropping a Partition
```
ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec, PARTITION partition_spec,...;
```
Example
```
hive> ALTER TABLE employee DROP [IF EXISTS]
   > PARTITION (year=’1203’);
```

## Built-in Operators
* Relational Operators
* Arrihmetic Operators
* Logical Operators
* Complex Operators

### Relational Operators
Operator |	Operand |	Description 
-------- | -------- | -------------
A = B	 | all primitive types |	TRUE if expression A is equivalent to expression B otherwise FALSE.
A != B	 | all primitive types |    TRUE if expression A is not equivalent to expression B otherwise FALSE.
A < B	 | all primitive types |	TRUE if expression A is less than expression B otherwise FALSE.
A <= B	 | all primitive types |	TRUE if expression A is less than or equal to expression B otherwise FALSE.
A > B	 | all primitive types |	TRUE if expression A is greater than expression B otherwise FALSE.
A >= B	 | all primitive types |	TRUE if expression A is greater than or equal to expression B otherwise FALSE.
A IS NULL |	all types	       |    TRUE if expression A evaluates to NULL otherwise FALSE.
A IS NOT NULL |	all types	   |    FALSE if expression A evaluates to NULL otherwise TRUE.
A LIKE B |	Strings	           |    TRUE if string pattern A matches to B otherwise FALSE.
A RLIKE B |	Strings            |	NULL if A or B is NULL, TRUE if any substring of A matches the Java regular expression B , otherwise FALSE.
A REGEXP B |	Strings	       |    Same as RLIKE.

#### Example
```
+-----+--------------+--------+---------------------------+------+
| Id  | Name         | Salary | Designation               | Dept |
+-----+--------------+------------------------------------+------+
|1201 | Gopal        | 45000  | Technical manager         | TP   |
|1202 | Manisha      | 45000  | Proofreader               | PR   |
|1203 | Masthanvali  | 40000  | Technical writer          | TP   |
|1204 | Krian        | 40000  | Hr Admin                  | HR   |
|1205 | Kranthi      | 30000  | Op Admin                  | Admin|
+-----+--------------+--------+---------------------------+------+
```
```
hive> SELECT * FROM employee WHERE Id=1205;
```
It'll get:
```
+-----+-----------+-----------+----------------------------------+
| ID  | Name      | Salary    | Designation              | Dept  |
+-----+---------------+-------+----------------------------------+
|1205 | Kranthi   | 30000     | Op Admin                 | Admin |
+-----+-----------+-----------+----------------------------------+
```
```
hive> SELECT * FROM employee WHERE Salary>=40000;
```
It'll get:
```
+-----+------------+--------+----------------------------+------+
| ID  | Name       | Salary | Designation                | Dept |
+-----+------------+--------+----------------------------+------+
|1201 | Gopal      | 45000  | Technical manager          | TP   |
|1202 | Manisha    | 45000  | Proofreader                | PR   |
|1203 | Masthanvali| 40000  | Technical writer           | TP   |
|1204 | Krian      | 40000  | Hr Admin                   | HR   |
+-----+------------+--------+----------------------------+------+
```

### Arrihmetic Operators
Operators |	Operand |	Description
--------- | ------- | -----------------------
A + B     |	all number types |	Gives the result of adding A and B.
A - B	  | all number types |	Gives the result of subtracting B from A.
A * B	  | all number types |	Gives the result of multiplying A and B.
A / B	  | all number types |	Gives the result of dividing B from A.
A % B	  | all number types |	Gives the reminder resulting from dividing A by B.
A & B	  | all number types |	Gives the result of bitwise AND of A and B.
A &#124; B	  | all number types |	Gives the result of bitwise OR of A and B.
A ^ B	  | all number types |	Gives the result of bitwise XOR of A and B.
~A	      | all number types |	Gives the result of bitwise NOT of A.

#### Example
`hive> SELECT 20+30 ADD FROM temp;` <br>
It'll get:
```
+--------+
|   ADD  |
+--------+
|   50   |
+--------+
```

### Logical Operators
Operators |	Operands |	Description
:--------: | -------- | ------------------
A AND B   |	boolean  | TRUE if both A and B are TRUE, otherwise FALSE.
A && B	  | boolean  | Same as A AND B.
A OR B	  | boolean	 | TRUE if either A or B or both are TRUE, otherwise FALSE.
A &#124;&#124; B  | boolean	 | Same as A OR B.
NOT A	  | boolean  | TRUE if A is FALSE, otherwise FALSE.
!A	      | boolean  | Same as NOT A.

#### Example 
`hive> SELECT * FROM employee WHERE Salary>40000 && Dept=TP;` <br>
Got that:
```
+------+--------------+-------------+-------------------+--------+
| ID   | Name         | Salary      | Designation       | Dept   |
+------+--------------+-------------+-------------------+--------+
|1201  | Gopal        | 45000       | Technical manager | TP     |
+------+--------------+-------------+-------------------+--------+
```

### Complex Operators

Operator |	Operand |	Description
-------- | -------- | -------------
A[n]	 | A is an Array and n is an int	| It returns the nth element in the array A. The first element has index 0.
M[key]	 | M is a Map<K, V> and key has type K	| It returns the value corresponding to the key in the map.
S.x	     | S is a struct |	It returns the x field of S.

## Built-In Functions

Return | Type |	Signature |	Description
------ | ---- | --------- | -------------
BIGINT | round(double a) |	It returns the rounded BIGINT value of the double.
BIGINT | floor(double a) |	It returns the maximum BIGINT value that is equal or less than the double.
BIGINT | ceil(double a)  |	It returns the minimum BIGINT value that is equal or greater than the double.
double | rand(), rand(int seed) | It returns a random number that changes from row to row.
string | concat(string A, string B,...) | It returns the string resulting from concatenating B after A.
string | substr(string A, int start) |	It returns the substring of A starting from start position till the end of string A.
string | substr(string A, int start, int length) |	It returns the substring of A starting from start position with the given length.
string | upper(string A) |	It returns the string resulting from converting all characters of A to upper case.
string | ucase(string A) |	Same as above.
string | lower(string A) |	It returns the string resulting from converting all characters of B to lower case.
string | lcase(string A) |	Same as above.
string | trim(string A)	 | It returns the string resulting from trimming spaces from both ends of A.
string | ltrim(string A) |	It returns the string resulting from trimming spaces from the beginning (left hand side) of A.
string | rtrim(string A)	rtrim(string A) | It returns the string resulting from trimming spaces from the end (right hand side) of A.
string | regexp_replace(string A, string B, string C) |	It returns the string resulting from replacing all substrings in B that match the Java regular expression syntax with C.
int	| size(Map<K.V>) | It returns the number of elements in the map type.
int | size(Array<T>) | It returns the number of elements in the array type.
value of <type>	| cast(<expr> as <type>) |	It converts the results of the expression expr to <type> e.g. cast('1' as BIGINT) converts the string '1' to it integral representation. A NULL is returned if the conversion does not succeed.
string | from_unixtime(int unixtime)	| convert the number of seconds from Unix epoch (1970-01-01 00:00:00 UTC) to a string representing the timestamp of that moment in the current system time zone in the format of "1970-01-01 00:00:00"
string | to_date(string timestamp)	| It returns the date part of a timestamp string: to_date("1970-01-01 00:00:00") = "1970-01-01"
int | year(string date)	| It returns the year part of a date or a timestamp string: year("1970-01-01 00:00:00") = 1970, year("1970-01-01") = 1970
int | month(string date)	| It returns the month part of a date or a timestamp string: month("1970-11-01 00:00:00") = 11, month("1970-11-01") = 11
int | day(string date)	| It returns the day part of a date or a timestamp string: day("1970-11-01 00:00:00") = 1, day("1970-11-01") = 1
string | get_json_object(string json_string, string path) |	It extracts json object from a json string based on json path specified, and returns json string of the extracted json object. It returns NULL if the input json string is invalid.

### Example

**round()** function <br>
`hive> SELECT round(2.6) from temp;` <br>
That gets: <br>
`3.0` <br>
**floor()** function <br>
`hive> SELECT floor(2.6) from temp;` <br>
That gets: <br>
`2.0` <br>
**ceil()** function <br>
`>hive SELECT ceil(2.6) from temp;` <br>
That gets: <br>
`3.0` <br>

## Aggregate Functions

Return Type | Signature | Description
----------- | --------- | -----------
BIGINT | count(*), count(expr),	 | count(*) - Returns the total number of retrieved rows.
DOUBLE | sum(col), sum(DISTINCT col) |	It returns the sum of the elements in the group or the sum of the distinct values of the column in the group.
DOUBLE	| avg(col), avg(DISTINCT col) |	It returns the average of the elements in the group or the average of the distinct values of the column in the group.
DOUBLE |	min(col) |	It returns the minimum value of the column in the group.
DOUBLE |	max(col) |	It returns the maximum value of the column in the group.

## Views and Indexes
* Create View 
```
CREATE VIEW [IF NOT EXISTS] view_name [(column_name [COMMENT column_comment], ...) ]
[COMMENT table_comment]
AS SELECT ...
```
Example
```
+------+--------------+-------------+-------------------+--------+
| ID   | Name         | Salary      | Designation       | Dept   |
+------+--------------+-------------+-------------------+--------+
|1201  | Gopal        | 45000       | Technical manager | TP     |
|1202  | Manisha      | 45000       | Proofreader       | PR     |
|1203  | Masthanvali  | 40000       | Technical writer  | TP     |
|1204  | Krian        | 40000       | Hr Admin          | HR     |
|1205  | Kranthi      | 30000       | Op Admin          | Admin  |
+------+--------------+-------------+-------------------+--------+
```
```
hive> CREATE VIEW emp_30000 AS
SELECT * FROM employee
WHERE salary>30000;
```
* Drop View
`DROP VIEW view_name;` <br>
`hive> DROP VIEW emp_30000;`
* Create Index 
```
CREATE INDEX index_name
ON TABLE base_table_name (col_name, ...)
AS 'index.handler.class.name'
[WITH DEFERRED REBUILD]
[IDXPROPERTIES (property_name=property_value, ...)]
[IN TABLE index_table_name]
[PARTITIONED BY (col_name, ...)]
[
   [ ROW FORMAT ...] STORED AS ...
   | STORED BY ...
]
[LOCATION hdfs_path]
[TBLPROPERTIES (...)]
```
Example
```
hive> CREATE INDEX inedx_salary ON TABLE employee(salary)
AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler';
```
* Drop Index
```
DROP INDEX <index_name> ON <table_name>;
```
```
hive> DROP INDEX index_salary ON employee;
```
