# HBase Shell

## General Commands 

* status
* version
* table_help
* whoami

## Data Definition Language 
These are the commands that operate on the tables in HBase.
* **create** - Creates a table.
* **list** - Lists all the tables in HBase.
* **disable** - Disables a table.
* **is_disabled** - Verifies whether a table is disabled.
* **enable** - Enables a table.
* **is_enabled** - Verifies whether a table is enabled.
* **describe** - Provides the description of a table.
* **alter** - Alters a table.
* **exists** - Verifies whether a table exists.
* **drop** - Drops a table from HBase.
* **drop_all** - Drops the tables matching the ‘regex’ given in the command.
* **Java Admin API** - Prior to all the above commands, Java provides an Admin API to achieve DDL functionalities through programming. Under **org.apache.hadoop.hbase.client** package, HBaseAdmin and HTableDescriptor are the two important classes in this package that provide DDL functionalities.

## Data Manipulation Language
* **put** - Puts a cell value at a specified column in a specified row in a particular table.
* **get** - Fetches the contents of row or a cell.
* **delete** - Deletes a cell value in a table.
* **deleteall** - Deletes all the cells in a given row.
* **scan** - Scans and returns the table data.
* **count** - Counts and returns the number of rows in a table.
* **truncate** - Disables, drops, and recreates a specified table.
* **Java client API** - Prior to all the above commands, Java provides a client API to achieve DML functionalities, **CRUD** (Create Retrieve Update Delete) operations and more through programming, under **org.apache.hadoop.hbase.client** package. **HTable Put** and **Get** are the important classes in this package.
 
### Start HBase Shell 

`hbase shell`

`list`


### HBaseAdmin

[Admin API](https://www.tutorialspoint.com/hbase/hbase_admin_api.htm)

## Creating a Table useing HBase Shell

`create '<table name>','<column family>'`
