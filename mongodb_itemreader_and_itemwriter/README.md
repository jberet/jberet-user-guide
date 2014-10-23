# MongoDB ItemReader and ItemWriter

jberet-support module includes `mongoItemReader` and `mongoItemWriter` for interacting with MongoDB NoSQL database. `mongoItemReader` reads data from MongoDB and binds to a custom POJO bean provided by the batch application. `mongoItemWriter` writes instances of the custom POJO bean type into MongoDB.

The following dependencies are required for `mongoItemReader` and `mongoItemWriter`:

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>${version.org.mongodb}</version>
</dependency>

<!-- mongojack and jackson are used to handle data binding -->
<dependency>
    <groupId>org.mongojack</groupId>
    <artifactId>mongojack</artifactId>
    <version>${version.org.mongojack}</version>
    <exclusions>
        <exclusion>
            <groupId>javax.persistence</groupId>
            <artifactId>persistence-api</artifactId>
        </exclusion>
    </exclusions>
</dependency>

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

<!-- POJO beans may contain either jackson annotations, or JAXB annotations -->
<!-- Include this dependency if JAXB annotation introspector is needed -->
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-jaxb-annotations</artifactId>
    <version>${version.com.fasterxml.jackson}</version>
</dependency>
```

##Configure `mongoItemReader` and `mongoItemWriter` in Job XML
`mongoItemReader` and `mongoItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type `String`, unless noted otherwise. The following is an example job xml that references `mongoItemReader` and `mongoItemWriter`:

```xml
<job id="MongoItemReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="MongoItemReaderTest.step1">
        <chunk>
            <reader ref="mongoItemReader">
                <properties>
                    <property name="uri" value="mongodb://localhost/testData"/>
                    <property name="beanType" value="org.jberet.support.io.Movie"/>
                    <property name="host" value="localhost"/>
                    <property name="database" value="testData"/>
                    <property name="collection" value="movies"/>
                    <property name="skip" value="1"/>
                    <property name="limit" value="50"/>
                    <property name="projection" value="{_id : 0}"/>
                </properties>
            </reader>
            <writer ref="mongoItemWriter">
                <properties>
                    <property name="uri" value="mongodb://localhost/testData"/>
                    <property name="beanType" value="org.jberet.support.io.Movie"/>
                    <property name="host" value="localhost"/>
                    <property name="database" value="testData"/>
                    <property name="collection" value="movies.out"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###Batch Properties for Both `mongoItemReader` and `mongoItemWriter`

####beanType
`java.lang.Class`

For `mongoItemReader`, it's the java type that each data item should be converted to; for `mongoItemWriter`, it's the java type for each incoming data item. Required property, and valid values are any data-representing bean class, for example,

* `org.jberet.support.io.StockTrade`
* `org.jberet.support.io.Person`
* `my.own.custom.ItemBean`


####mongoClientLookup
JNDI lookup name for `com.mongodb.MongoClient`. Optional property and defaults to `null`. When this property is specified, its value will be used to look up an instance of `com.mongodb.MongoClient`, which is typically created and administrated externally (e.g., inside application server). Otherwise, a new instance of `com.mongodb.MongoClient` will be created instead of lookup. See `org.jberet.support.io.MongoClientObjectFactory` and `com.mongodb.MongoClient` for more details.
    
####uri
The Mongo client URI. See `com.mongodb.MongoClientURI` docs for syntax and details. Optional property. When this property is present, it should encompass all information necessary to establish a client connection. When this property is not present, other properties (e.g., `host`, `database`, `user`, `password`, `options`, and `collection`) should be specified to satisfy `com.mongodb.MongoClientURI` requirement.

####host
Host and optional port information for creating `com.mongodb.MongoClientURI`. It can be single host and port, or multiple host and port specification in the format `host1[:port1][,host2[:port2],...[,hostN[:portN]]]`. See `com.mongodb.MongoClientURI`, `MongoClientObjectFactory.createMongoClientURI` for more details.

####database
MongoDB database name, e.g., testData. Optional property and defaults to `null`. See `com.mongodb.MongoClientURI`, `MongoClientObjectFactory.createMongoClientURI` for more details.

####user
MongoDB username. Optional property and defaults to `null`.

####password
MongoDB password. Optional property and defaults to `null`.

####options
MongoDB client options, e.g., `safe=true&wtimeout=1000`. Optional property and defaults to `null`. See `com.mongodb.MongoClientURI` for more details.

####collection
MongoDB collection name, e.g., `movies`. Optional property and defaults to `null`.


###Batch Properties for `mongoItemReader` Only
In addition to the above common properties, `mongoItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####criteria
Query criteria or conditions, which identify the documents that MongoDB returns to the client. Its value is a JSON string. Optional property and defaults to `null`. For example,

  `{ age: { $gt: 18 } }`
 
####projection
Specifies the fields from the matching documents to return. Its value is a JSON string. Optional property and defaults to `null`. For example,

`{ name: 1, address: 1}`
 
####limit
`int`

Modifies the query to limit the number of matching documents to return to the client. Optional property and defaults to `null` (limit is not set).

####batchSize
`int`

Limits the number of elements returned in one batch. A cursor typically fetches a batch of result objects and store them locally. If `batchSize` is positive, it represents the size of each batch of objects retrieved. It can be adjusted to optimize performance and limit data transfer. If `batchSize` is negative, it will limit of number objects returned, that fit within the max batch size limit (usually 4MB), and cursor will be closed. For example if batchSize is `-10`, then the server will return a maximum of `10` documents and as many as can fit in 4MB, then close the cursor.

Note that this feature is different from `limit()` in that documents must fit within a maximum size, and it removes the need to send a request to close the cursor server-side. The batch size can be changed even after a cursor is iterated, in which case the setting will apply on the next batch retrieval.

See `com.mongodb.DBCursor#batchSize(int)` for more details.

####sort
Specifies how to sort the cursor's elements. Optional property and defaults to `null`. Its value is a JSON string, for example,

`{ age: 1 }`
 
####skip
`int`

Specifies the number of elements to discard at the beginning of the cursor. Optional property and defaults to `0` (do not discard any elements). See `com.mongodb.DBCursor#skip(int)` for more details.
    
###Batch Properties for `mongoItemWriter` Only
None
