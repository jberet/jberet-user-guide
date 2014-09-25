# Read and Write Data through JDBC
jberet-support module contains `JdbcItemReader` and `JdbcItemWriter` that reads from and write to database through JDBC. Batch applications can reference them by name `jdbcItemReader` and `jdbcItemWriter` in job xml. 

`jdbcItemReader` can read a row of data into one of three types, configured through `beanType` batch property:
* `java.util.List`: populated with column value in the order as returned by the query result set
* `java.util.Map`: with column name as the key
* any custom POJO, e.g., `StockTrade`, `Person`, `Employee`, etc
 
Likewise, `jdbcItemWriter` can obtain data from one of three types for database insertion, configured through `beanType` batch property:
* `java.util.List`: used to populate the insert statement
* `java.util.Map`: used to set parameter values of the insert statement with map key as the parameter name
* any custom POJO, e.g., `StockTrade`, `Person`, `Employee`, etc

When `beanType` is configured to a custom POJO bean in `jdbcItemReader` and `jdbcItemWriter`, they use jackson-databind library to perform transformation between `java.util.Map` and POJO bean. So in this case, the following jackson dependencies are needed at runtime:

```xml
<!-- needed if beanType is set to custom POJO bean -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>${version.com.fasterxml.jackson}</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${version.com.fasterxml.jackson}</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>${version.com.fasterxml.jackson}</version>
</dependency>
```

Of course all applications should include any database-specific JDBC driver jars in the runtime classpath.

##Configure `jdbcItemReader` and `jdbcItemWriter` in job xml
The following sample job xml demonstrates how to reference and configure `jdbcItemReader` and `jdbcItemWriter` in job xml. Each batch property will be explained in the next section.

```xml
<job id="JdbcReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="step1">
        <chunk item-count="100">
            <reader ref="jdbcItemReader">
                <properties>
                    <property name="beanType" value="org.jberet.support.io.StockTrade"/>
                    <property name="sql" value="select TRADEDATE, TRADETIME, OPEN, HIGH, LOW, CLOSE, VOLUMN from STOCK_TRADE"/>
                    <property name="url" value="jdbc:h2:~/test"/>
                    <property name="columnMapping" value="Date,Time,Open,High,Low,Close,Volume"/>
                    <property name="columnTypes" value="Date, String, Double, Double, Double, Double, Double"/>
                    <property name="start" value="1"/>
                    <property name="end" value="10"/>
                    <property name="resultSetProperties" value="fetchSize=1000, resultSetConcurrency=CONCUR_UPDATABLE, fetchDirection=FETCH_REVERSE, resultSetType=TYPE_SCROLL_SENSITIVE, resultSetHoldability=HOLD_CURSORS_OVER_COMMIT"/>
                </properties>
            </reader>
            ...
        </chunk>
    </step>
</job>

<job id="JdbcWriterTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="step1">
        <chunk item-count="100">
            ...
            <writer ref="jdbcItemWriter">
                <properties>
                    <property name="sql" value="insert into STOCK_TRADE (TRADEDATE, TRADETIME, OPEN, HIGH, LOW, CLOSE, VOLUMN) VALUES(?, ?, ?, ?, ?, ?, ?)"/>
                    <property name="url" value="jdbc:h2:~/test"/>
                    <property name="parameterNames" value="Date,Time,Open,High,Low,Close,Volume"/>
                    <property name="parameterTypes" value="Date, String, Double, Double, Double, Double, Double"/>
                    <property name="beanType" value="org.jberet.support.io.StockTrade"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

##Batch Configuration Properties for Both `jdbcItemReader` and `jdbcItemWriter`
###sql
The sql statement for reading data from database, or inserting data into database. It should include parameter markers that will be filled in with real data by the current batch `ItemReader` or `ItemWriter`.

###beanType
`java.lang.Class`

For `ItemReader`, it's the java type that each data item should be converted to; for `ItemWriter`, it's the java type for each incoming data item. In either case, the valid values are:

* java.util.Map
* java.util.List
* a custom java type that represents data item;

###dataSourceLookup
JNDI lookup name of the `javax.sql.DataSource`. Optional property, and defaults to null. If specified, it will be used to look up the target DataSource, and other database connection batch properties for this writer class will be ignored.

###url
JDBC connection url

###user
User name for the JDBC connection

###password
Password for the JDBC connection

###properties
`java.util.Map<String, String>`

Additional properties for the JDBC connection


##Batch Configuration Properties for `jdbcItemReader` Only
In addition to the common batch properties listed above, `jdbcItemReader` may also be configured through the following batch properties:

###skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO.  Optional property and defaults to false, i.e., the reader will validate data POJO bean where appropriate.

###start
`int`

The row number in the ResultSet to start reading. It's a positive integer starting from 1.

###end
`int`

The row number in the ResultSet to end reading (inclusive). It's a positive integer starting from 1.

###columnMapping
`java.lang.String[]`

String keys used in target data structure for database columns. Optional property, and if not specified, it defaults to column labels from `java.sql.ResultSetMetaData`. This property should have the same length and order as `columnTypes`, if the latter is specified.

For example, if `sql` is

    SELECT NAME, ADDRESS, AGE FROM PERSON

And you want to map the data to the following form:

    {"fn" = "Jon", "addr" = "1 Main st", "age" = 30}

then `columnMapping` should be specified as follows in job xml:

    "fn, addr, age"
    
###columnTypes
`java.lang.String[]`

Tells this class which `java.sql.ResultSet` getter method to call to get `ResultSet` field value. It should have the same length and order as `columnMapping`. Optional property, and if not set, this class calls `ResultSet.getObject(java.lang.String)` for all columns. For example, this property can be configured as follows in job xml:

    "String, String, Int"

And this class will call `ResultSet.getString(java.lang.String)`, `ResultSet.getString(java.lang.String)`, and `ResultSet.getInt(java.lang.String)`.

###resultSetProperties
`java.lang.String[]`

The following `resultSetProperties` can be optionally configured in job xml:

* fetchSize (use driver default)
* fetchDirection
    + FETCH_FORWARD (default)
    + FETCH_REVERSE
    + FETCH_UNKNOWN
* resultSetType:
    + TYPE_FORWARD_ONLY (default)
    + TYPE_SCROLL_INSENSITIVE
    + TYPE_SCROLL_SENSITIVE
* resultSetConcurrency:
    + CONCUR_READ_ONLY (default)
    + CONCUR_UPDATABLE
* resultSetHoldability:
    + HOLD_CURSORS_OVER_COMMIT (default)
    + CLOSE_CURSORS_AT_COMMIT

See [`java.sql.ResultSet` javadoc](http://docs.oracle.com/javase/7/docs/api/java/sql/ResultSet.html) for detailed explanation of these properties. For example:

    <property name="resultSetProperties" 
              value="fetchSize=1000, resultSetConcurrency=CONCUR_UPDATABLE"/>
              
##Batch Configuration Properties for `jdbcItemWriter` Only
In addition to the common batch properties listed above, `jdbcItemWriter` may also be configured through the following batch properties:

###parameterNames
`java.lang.String[]`

String keys used to retrieve values from incoming data and apply to SQL insert statement parameters. It should have the same length and order as SQL insert statement parameters. Optional property and if not set, it is initialized from the columns part of SQL insert statement. This property is not used when `beanType` is `java.util.List`, which assumes that incoming data is already in the same order as SQL parameters.

If `beanType` is `java.util.Map`, and any of its key is different than the target table column names, `parameterNames` should be specified. For example, if an incoming data item is:

    {"name" = "Jon", "address" = "1 Main st", "age" = 30}

And `sql` is

    INSERT INTO PERSON(NAME, ADDRESS, AGE) VALUES(?, ?, ?)

then `parameterNames` should be specified as follows in job xml:

    "name, address, age"

If `beanType` is custom bean type, custom mapping may be achieved with either `parameterNames`, or in bean class with annotations, e.g., JAXB or Jackson annotations. If the bean class does not contain field mapping, or the field mapping is intended for other part of the application (e.g., `ItemReader`), `parameterNames` can be used to customize mapping.

###parameterTypes
`java.lang.String[]`

Tells this class which `PreparedStatement` setter method to call to set insert statement parameters. It should have the same length and order as SQL insert statement parameters. Optional property, and if not set, this class calls `PreparedStatement.setObject(int, Object)` for all parameters. For example, this property can be configured as follows in job xml:

    "String, String, Int"

And this class will call `PreparedStatement.setString(int, String)`, `PreparedStatement.setString(int, String)`, and `PreparedStatement.setInt(int, int)`.

##Batch Configuration Properties for jackson-databind library
When `jdbcItemReader` or `jdbcItemWriter` uses custom POJO bean as `beanType`, `jackson-databind` performs data transformation behind the scene, and this step can be configured through the following batch configuration properties in job xml, though the defaults should suffice in most cases:

###jsonFactoryFeatures

###mapperFeatures

###jsonFactoryLookup

###serializationFeatures

###customSerializers

###deserializationFeatures

###customDeserializers

