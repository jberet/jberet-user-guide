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

##Configure `mongoItemReader` and `mongoItemWriter in Job XML
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
####resource

The resource to read from (for batch readers), or write to (for batch writers).

####beanType
`java.lang.Class`

####mongoClientLookup

####uri

####host

####database

####user

####password

####options

####collection


###Batch Properties for `mongoItemReader` Only
In addition to the above common properties, `mongoItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####criteria

####projection

####limit
`int`

####batchSize
`int`

####sort

####skip
`int`


###Batch Properties for `mongoItemWriter` Only
None
