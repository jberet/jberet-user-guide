# JsonItemReader and JsonItemWriter

jberet-support module contains `jsonItemReader` and `jsonItemWriter` batch artifacts for reading and writing JSON data. `jsonItemReader` reads JSON data and binds them to `java.util.Map`, `com.fasterxml.jackson.databind.JsonNode`, or any custom POJO bean provided by the batch application.

`jsonItemReader` and `jsonItemWriter` leverages `jackson` library to handle JSON data parsing and binding. The following `jackson` dependencies are required:

```xml
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

##Batch Configuration Properties in Job XML

`jsonItemReader` and `jsonItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type `String`, unless noted otherwise.

###Batch Properties for Both `jsonItemReader` and `jsonItemWriter`

####resource

The resource to read from (for batch readers), or write to (for batch writers). 

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####jsonFactoryFeatures

####mapperFeatures

####jsonFactoryLookup

####serializationFeatures

####customSerializers

####deserializationFeatures

####customDeserializers


###Batch Properties for `jsonItemReader` only

####beanType
`java.lang.Class`

####start
`int`

####end
`int`

####jsonParserFeatures
`java.util.Map<String, String>`

####deserializationProblemHandlers

####inputDecorator
`java.lang.Class`



###Batch Properties for `jsonItemWriter` only

####writeMode

####jsonGeneratorFeatures
`java.util.Map<String, String>`

####prettyPrinter
`java.lang.Class`

####outputDecorator
`java.lang.Class`


