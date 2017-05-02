# Cassandra

- highly scalable
- designed to manage very large amounts of structured data
- no single point of failure 
- NoSQL(Not only SQL) database
- Wide Column Stores

## 数据模型

- 二维的key-value 存储 `map<key1, map<key2,value>>`
- BigTable(Column Family) = Cassandra(table)
- BigTable(Column key) = Cassandra(Row key) = primary key

Cassandra的row key决定了该行数据存储在哪些节点中，因此row key需要按哈希来存储，不能顺序的扫描或读取，而一个row内的column key是顺序存储的，可以进行有序的扫描或范围查找

Cassandra的数据并不存储在分布式文件系统如GFS或HDFS中，而是直接存于本地

日志型数据库，即把新写入的数据存储在内存的Memtable中并通过磁盘中的CommitLog来做持久化，内存填满后将数据按照key的顺序写进一个只读文件SSTable中，每次读取数据时将所有SSTable和内存中的数据进行查找和合并

## Key Components of Cassandra

- Node: where data stored
- Data center: a collection of related nodes
- Cluster: contains one or more data centers
- Commit log: The commit log is a crash-recovery mechanism in Cassandra. Every write operation is written to the commit log.
- Mem-table: a memory-resident data structure. After commit log, the data will be written to the mem-table. Sometimes, for a single-column family, there will be multiple mem-tables.
- SSTable: a disk file to which the data is flushed from the mem-table when its reach a threshold value.
- Bloom filter: a special kind of cache

## Cassandra Query Language

### Write Operations

write activity -> commit logs
data -> mem-table -> SSTable
automatically partitioned and replicated throughout the cluster

### Read Operations

Cassandra gets values from the mem-table and checks the bloom filter to find the appropriate SSTable that holds the required data.

## cqlsh

Options | Usage
------- | ------
cqlsh --help | Shows help topics about the options of cqlsh commands.
cqlsh --version | Provides the version of the cqlsh you are using.
cqlsh --color | Directs the shell to use colored output.
cqlsh --debug | Shows additional debugging information.
cqlsh --execute cql_statement | Directs the shell to accept and execute a CQL command.
cqlsh --file= "file name" | If you use this option, Cassandra executes the command in the given file and exits.
cqlsh --no-color | Directs Cassandra not to use colored output.
cqlsh -u "user name" | Using this option, you can authenticate a user. The default user name is: cassandra.
cqlsh-p "pass word" | Using this option, you can authenticate a user with a password. The default password is: cassandra.

### Documented Shell Commands
Given below are the Cqlsh documented shell commands. These are the commands used to perform tasks such as displaying help topics, exit from cqlsh, describe,etc.

- **HELP** - Displays help topics for all cqlsh commands.

- **CAPTURE** - Captures the output of a command and adds it to a file.

- **CONSISTENCY** - Shows the current consistency level, or sets a new consistency level.

- **COPY** - Copies data to and from Cassandra.

> `COPY emp (emp_id, emp_city, emp_name, emp_phone,emp_sal) TO 'myfile';`

- **DESCRIBE** - Describes the current cluster of Cassandra and its objects.

> `describe cluster;`
> `describe keyspaces;`
> `describe tables;`
> `describe table emp;`
> `describe type card_details;`
> `DESCRIBE TYPES;`

- **EXPAND** - Expands the output of a query vertically.

- **EXIT** - Using this command, you can terminate cqlsh.

- **PAGING** - Enables or disables query paging.

- **SHOW** - Displays the details of current cqlsh session such as Cassandra version, host, or data type assumptions.

- **SOURCE** - Executes a file that contains CQL statements.

- **TRACING** - Enables or disables request tracing.

### CQL Data Definition Commands

- **CREATE KEYSPACE** - Creates a KeySpace in Cassandra.

> `CREATE KEYSPACE <identifier> WITH <properties>`
```
CREATE KEYSPACE "KeySpace Name"
WITH replication = {'class': 'Strategy name', 'replication_factor': 'No.Of replicas'}
AND durable_writes = 'Boolean value';
SELECT * FROM system.schema_keyspaces;
```
```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Create_KeySpace {

   public static void main(String args[]){

      //Query
      String query = "CREATE KEYSPACE tp WITH replication "
         + "= {'class':'SimpleStrategy', 'replication_factor':1};";
                    
      //creating Cluster object
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
    
      //Creating Session object
      Session session = cluster.connect();
     
      //Executing the query
      session.execute(query);
     
      //using the KeySpace
      session.execute("USE tp");
      System.out.println("Keyspace created"); 
   }
}
```

- **USE** - Connects to a created KeySpace.

- **ALTER KEYSPACE** - Changes the properties of a KeySpace.

> `ALTER KEYSPACE <identifier> WITH <properties>`
```
ALTER KEYSPACE "KeySpace Name"
WITH replication = {'class': 'Strategy name', 'replication_factor': 'No.Of replicas'};
ALTER KEYSPACE tutorialspoint
WITH replication = {'class':'NetworkTopologyStrategy', 'replication_factor': 3};
ALTER KEYSPACE test
WITH REPLICATION = {'class' : 'NetworkTopologyStrategy', 'datacenter1' : 3}
AND DURABLE_WRITES = true;
```
```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Alter_KeySpace {
   public static void main(String args[]){

      //Query
      String query = "ALTER KEYSPACE tp WITH replication " + "= {'class':'NetworkTopologyStrategy', 'datacenter1':3}"
         + "AND DURABLE_WRITES = false;";

      //Creating Cluster object
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
   
      //Creating Session object
      Session session = cluster.connect();
 
      //Executing the query
      session.execute(query);
 
      System.out.println("Keyspace altered");
   }
}
```

- **DROP KEYSPACE** - Removes a KeySpace

> `DROP KEYSPACE <identifier>;`
> `DROP KEYSPACE "keySpaceName";`
```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Drop_KeySpace {

   public static void main(String args[]){

      //Query
      String query = "Drop KEYSPACE tp";

      //creating Cluster object
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
    
      //Creating Session object
      Session session = cluster.connect();
    
      //Executing the query
      session.execute(query);
      System.out.println("Keyspace deleted");
   }
}
```

- **CREATE TABLE** - Creates a table in a KeySpace.

```
CREATE (TABLE | COLUMNFAMILY) <tablename>
('<column-definition>' , '<column-definition>')
(WITH <option> AND <option>)
```
```
CREATE TABLE tablename(
   column1 name datatype PRIMARYKEY,
   column2 name data type,
   column3 name data type.
   )
```
```
CREATE TABLE tablename(
   column1 name datatype PRIMARYKEY,
   column2 name data type,
   column3 name data type,
   PRIMARY KEY (column1)
   )
```
```
CREATE TABLE emp(
   emp_id int PRIMARY KEY,
   emp_name text,
   emp_city text,
   emp_sal varint,
   emp_phone varint
   );
```
```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Create_Table {

   public static void main(String args[]){

      //Query
      String query = "CREATE TABLE emp(emp_id int PRIMARY KEY, "
         + "emp_name text, "
         + "emp_city text, "
         + "emp_sal varint, "
         + "emp_phone varint );";
		
      //Creating Cluster object
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
   
      //Creating Session object
      Session session = cluster.connect("tp");
 
      //Executing the query
      session.execute(query);
 
      System.out.println("Table created");
   }
}
```

- **ALTER TABLE** - Modifies the column properties of a table.
 
`ALTER (TABLE | COLUMNFAMILY) <tablename> <instruction>;`
```
ALTER TABLE table name
ADD  new column datatype;
ALTER table name
DROP column name;
```
```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Add_column {

   public static void main(String args[]){

      //Query
      String query = "ALTER TABLE emp ADD emp_email text;";
      //Query
      //String query = "ALTER TABLE emp DROP emp_email;";

      //Creating Cluster object
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
     
      //Creating Session object
      Session session = cluster.connect("tp");
    
      //Executing the query
      session.execute(query);
  
      System.out.println("Column added");
   }
}
```

- **DROP TABLE** - Removes a table.

- **TRUNCATE** - Removes all the data from a table.

- **CREATE INDEX** - Defines a new index on a single column of a table.

> `CREATE INDEX <identifier> ON <tablename>;`
> `CREATE INDEX name ON emp1 (emp_name);`

```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Create_Index {
 
   public static void main(String args[]){

      //Query
      String query = "CREATE INDEX name ON emp1 (emp_name);";
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
       
      //Creating Session object
      Session session = cluster.connect("tp");
 
      //Executing the query
      session.execute(query);
      System.out.println("Index created");
   }
}
```

- **DROP INDEX** - Deletes a named index.
> `drop index name;`

### CQL Data Manipulation Commands

- **INSERT** - Adds columns for a row in a table.
```
INSERT INTO <tablename>
(<column1 name>, <column2 name>....)
VALUES (<value1>, <value2>....)
USING <option>;
```
> `INSERT INTO emp (emp_id, emp_name, emp_city, emp_phone, emp_sal) VALUES(1,'ram', 'Hyderabad', 9848022338, 50000);`
```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Create_Data {

   public static void main(String args[]){

      //queries
      String query1 = "INSERT INTO emp (emp_id, emp_name, emp_city, emp_phone,  emp_sal)"
		
         + " VALUES(1,'ram', 'Hyderabad', 9848022338, 50000);" ;
                             
      String query2 = "INSERT INTO emp (emp_id, emp_name, emp_city,
         emp_phone, emp_sal)"
      
         + " VALUES(2,'robin', 'Hyderabad', 9848022339, 40000);" ;
                             
      String query3 = "INSERT INTO emp (emp_id, emp_name, emp_city, emp_phone, emp_sal)"
       
         + " VALUES(3,'rahman', 'Chennai', 9848022330, 45000);" ;

      //Creating Cluster object
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
 
      //Creating Session object
      Session session = cluster.connect("tp");
       
      //Executing the query
      session.execute(query1);
        
      session.execute(query2);
        
      session.execute(query3);
        
      System.out.println("Data created");
   }
}
```

- **UPDATE** - Updates a column of a row.
```
UPDATE <tablename>
SET <column name> = <new value>
<column name> = <value>....
WHERE <condition>;
```

- **DELETE** - Deletes data from a table.
> `DELETE FROM emp WHERE emp_id=3;`

- **BATCH** - Executes multiple DML statements at once.
```
BEGIN BATCH
<insert-stmt>/ <update-stmt>/ <delete-stmt>
APPLY BATCH
```
```
BEGIN BATCH
INSERT INTO emp (emp_id, emp_city, emp_name, emp_phone, emp_sal) values(  4,'Pune','rajeev',9848022331, 30000);
UPDATE emp SET emp_sal = 50000 WHERE emp_id =3;
DELETE emp_city FROM emp WHERE emp_id = 2;
APPLY BATCH;
```
```
import com.datastax.driver.core.Cluster;
import com.datastax.driver.core.Session;

public class Batch {

   public static void main(String args[]){
    
      //query
      String query =" BEGIN BATCH INSERT INTO emp (emp_id, emp_city,
         emp_name, emp_phone, emp_sal) values( 4,'Pune','rajeev',9848022331, 30000);"
    
         + "UPDATE emp SET emp_sal = 50000 WHERE emp_id =3;"
         + "DELETE emp_city FROM emp WHERE emp_id = 2;"
         + "APPLY BATCH;";

      //Creating Cluster object
      Cluster cluster = Cluster.builder().addContactPoint("127.0.0.1").build();
 
      //Creating Session object
      Session session = cluster.connect("tp");
 
      //Executing the query
      session.execute(query);

      System.out.println("Changes done");
   }
}
```

### CQL Clauses

- **SELECT** - This clause reads data from a table

- **WHERE** - The where clause is used along with select to read a specific data.

- **ORDERBY** - The orderby clause is used along with select to read a specific data in a specific order.


## Data type
Data | Type |	Constants |	Description
----- | ---- | ---------- | -----------
ascii |	strings |	Represents | ASCII character string
bigint |	bigint |	Represents 64-bit signed long
blob |	blobs |	Represents arbitrary bytes
Boolean |	booleans |	Represents true or false
counter |	integers |	Represents counter column
decimal |	integers, floats |	Represents variable-precision decimal
double |	integers |	Represents 64-bit IEEE-754 floating point
float |	integers, floats |	Represents 32-bit IEEE-754 floating point
inet |	strings |	Represents an IP address, IPv4 or IPv6
int |	integers |	Represents 32-bit signed int
text |	strings |	Represents UTF8 encoded string
timestamp |	integers, strings |	Represents a timestamp
timeuuid |	uuids |	Represents type 1 UUID
uuid | 	uuids |	Represents type 1 or type 4
 | | UUID
varchar |	strings | Represents uTF8 encoded string
varint |	integers | Represents arbitrary-precision integer

Collection |	Description
---------- | ---------------
list |	A list is a collection of one or more ordered elements.
map |	A map is a collection of key-value pairs.
set |	A set is a collection of one or more elements.

### ser-defined datatypes

- **CREATE** TYPE - Creates a user-defined datatype.

- **ALTER** TYPE - Modifies a user-defined datatype.

- **DROP** TYPE - Drops a user-defined datatype.

- **DESCRIBE** TYPE - Describes a user-defined datatype.

- **DESCRIBE** TYPES - Describes user-defined datatypes.

# [more](https://www.tutorialspoint.com/cassandra/cassandra_cql_collections.htm)
