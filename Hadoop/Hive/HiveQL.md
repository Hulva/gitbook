# HiveQL
> Hive Query Language

```
SELECT [ALL | DISTINCT] select_expr, select_expr, ... 
FROM table_reference 
[WHERE where_condition] 
[GROUP BY col_list] 
[HAVING having_condition] 
[CLUSTER BY col_list | [DISTRIBUTE BY col_list] [SORT BY col_list]] 
[LIMIT number];
```

## Example
SELECT Where 
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
hive> SELECT * FROM employee WHERE salary>30000;
+------+--------------+-------------+-------------------+--------+
| ID   | Name         | Salary      | Designation       | Dept   |
+------+--------------+-------------+-------------------+--------+
|1201  | Gopal        | 45000       | Technical manager | TP     |
|1202  | Manisha      | 45000       | Proofreader       | PR     |
|1203  | Masthanvali  | 40000       | Technical writer  | TP     |
|1204  | Krian        | 40000       | Hr Admin          | HR     |
+------+--------------+-------------+-------------------+--------+
```
SELECT ORDER BY
```
hive> SELECT Id, Name, Dept FROM employee ORDER BY DEPT;
+------+--------------+-------------+-------------------+--------+
| ID   | Name         | Salary      | Designation       | Dept   |
+------+--------------+-------------+-------------------+--------+
|1205  | Kranthi      | 30000       | Op Admin          | Admin  |
|1204  | Krian        | 40000       | Hr Admin          | HR     |
|1202  | Manisha      | 45000       | Proofreader       | PR     |
|1201  | Gopal        | 45000       | Technical manager | TP     |
|1203  | Masthanvali  | 40000       | Technical writer  | TP     |
+------+--------------+-------------+-------------------+--------+
```
SELECT GROUP BY
```
hive> SELECT Dept,count(*) FROM employee GROUP BY DEPT;
+------+--------------+ 
| Dept | Count(*)     | 
+------+--------------+ 
|Admin |    1         | 
|PR    |    2         | 
|TP    |    3         | 
+------+--------------+
```
SELECT JOINS
```
+----+----------+-----+-----------+----------+ 
| ID | NAME     | AGE | ADDRESS   | SALARY   | 
+----+----------+-----+-----------+----------+ 
| 1  | Ramesh   | 32  | Ahmedabad | 2000.00  |  
| 2  | Khilan   | 25  | Delhi     | 1500.00  |  
| 3  | kaushik  | 23  | Kota      | 2000.00  | 
| 4  | Chaitali | 25  | Mumbai    | 6500.00  | 
| 5  | Hardik   | 27  | Bhopal    | 8500.00  | 
| 6  | Komal    | 22  | MP        | 4500.00  | 
| 7  | Muffy    | 24  | Indore    | 10000.00 | 
+----+----------+-----+-----------+----------+
```
```
+-----+---------------------+-------------+--------+ 
|OID  | DATE                | CUSTOMER_ID | AMOUNT | 
+-----+---------------------+-------------+--------+ 
| 102 | 2009-10-08 00:00:00 |           3 | 3000   | 
| 100 | 2009-10-08 00:00:00 |           3 | 1500   | 
| 101 | 2009-11-20 00:00:00 |           2 | 1560   | 
| 103 | 2008-05-20 00:00:00 |           4 | 2060   | 
+-----+---------------------+-------------+--------+
```
* JOIN 
```
hive> SELECT c.ID, c.NAME, c.AGE, o.AMOUNT 
FROM CUSTOMERS c JOIN ORDERS o 
ON (c.ID = o.CUSTOMER_ID);
+----+----------+-----+--------+ 
| ID | NAME     | AGE | AMOUNT | 
+----+----------+-----+--------+ 
| 3  | kaushik  | 23  | 3000   | 
| 3  | kaushik  | 23  | 1500   | 
| 2  | Khilan   | 25  | 1560   | 
| 4  | Chaitali | 25  | 2060   | 
+----+----------+-----+--------+
```
* LEFT OUTER JOIN
```
hive> SELECT c.ID, c.NAME, o.AMOUNT, o.DATE 
FROM CUSTOMERS c 
LEFT OUTER JOIN ORDERS o 
ON (c.ID = o.CUSTOMER_ID);
+----+----------+--------+---------------------+ 
| ID | NAME     | AMOUNT | DATE                | 
+----+----------+--------+---------------------+ 
| 1  | Ramesh   | NULL   | NULL                | 
| 2  | Khilan   | 1560   | 2009-11-20 00:00:00 | 
| 3  | kaushik  | 3000   | 2009-10-08 00:00:00 | 
| 3  | kaushik  | 1500   | 2009-10-08 00:00:00 | 
| 4  | Chaitali | 2060   | 2008-05-20 00:00:00 | 
| 5  | Hardik   | NULL   | NULL                | 
| 6  | Komal    | NULL   | NULL                | 
| 7  | Muffy    | NULL   | NULL                | 
+----+----------+--------+---------------------+
```
* RIGHT OUTER JOIN 
```
hive> SELECT c.ID, c.NAME, o.AMOUNT, o.DATE 
FROM CUSTOMERS c 
RIGHT OUTER JOIN ORDERS o 
ON (c.ID = o.CUSTOMER_ID);
+------+----------+--------+---------------------+ 
| ID   | NAME     | AMOUNT | DATE                | 
+------+----------+--------+---------------------+ 
| 3    | kaushik  | 3000   | 2009-10-08 00:00:00 | 
| 3    | kaushik  | 1500   | 2009-10-08 00:00:00 | 
| 2    | Khilan   | 1560   | 2009-11-20 00:00:00 | 
| 4    | Chaitali | 2060   | 2008-05-20 00:00:00 | 
+------+----------+--------+---------------------+
```
* FULL OUTER JOIN 
```
hive> SELECT c.ID, c.NAME, o.AMOUNT, o.DATE 
FROM CUSTOMERS c 
FULL OUTER JOIN ORDERS o 
ON (c.ID = o.CUSTOMER_ID);
+------+----------+--------+---------------------+ 
| ID   | NAME     | AMOUNT | DATE                | 
+------+----------+--------+---------------------+ 
| 1    | Ramesh   | NULL   | NULL                | 
| 2    | Khilan   | 1560   | 2009-11-20 00:00:00 | 
| 3    | kaushik  | 3000   | 2009-10-08 00:00:00 | 
| 3    | kaushik  | 1500   | 2009-10-08 00:00:00 | 
| 4    | Chaitali | 2060   | 2008-05-20 00:00:00 | 
| 5    | Hardik   | NULL   | NULL                | 
| 6    | Komal    | NULL   | NULL                |
| 7    | Muffy    | NULL   | NULL                |  
| 3    | kaushik  | 3000   | 2009-10-08 00:00:00 | 
| 3    | kaushik  | 1500   | 2009-10-08 00:00:00 | 
| 2    | Khilan   | 1560   | 2009-11-20 00:00:00 | 
| 4    | Chaitali | 2060   | 2008-05-20 00:00:00 | 
+------+----------+--------+---------------------+
```
