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

`jsonItemReader` and `jsonItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type `String`, unless noted otherwise. The following is an example job xml that references `jsonItemReader` and `jsonItemWriter`:

```xml
<job id="JsonItemReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="JsonItemReaderTest.step1">
        <chunk item-count="100">
            <reader ref="jsonItemReader">
                <properties>
                    <property name="resource" value="#{jobParameters['resource']}"/>
                    <property name="start" value="1"/>
                    <property name="end" value="199"/>
                    <property name="beanType" value="org.jberet.support.io.Movie"/>
                    <property  name="inputDecorator" value="org.jberet.support.io.JsonItemReaderTest$NoopInputDecorator"/>
                    <property name="customDeserializers" value="org.jberet.support.io.JsonItemReaderTest$JsonDeserializer"/>
                    <property name="deserializationProblemHandlers"
                              value="org.jberet.support.io.JsonItemReaderTest$UnknownHandler, org.jberet.support.io.JsonItemReaderTest$Unknown2Handler"/>
                </properties>
            </reader>
            
            <writer ref="jsonItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="jsonGeneratorFeatures" value=" QUOTE_FIELD_NAMES = false , STRICT_DUPLICATE_DETECTION = false"/>
                    <property name="serializationFeatures" value="WRITE_ENUMS_USING_INDEX=true"/>
                    <property name="outputDecorator" value="org.jberet.support.io.JsonItemReaderTest$NoopOutputDecorator"/>
                    <property name="customSerializers" value="org.jberet.support.io.JsonItemReaderTest$JsonSerializer"/>
                    <property name="writeMode" value="overwrite"/>
                    <!--<property name="prettyPrinter" value="com.fasterxml.jackson.core.util.MinimalPrettyPrinter"/>-->
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###Batch Properties for Both `jsonItemReader` and `jsonItemWriter`

####resource

The resource to read from (for batch readers), or write to (for batch writers). 

####jsonFactoryFeatures

####mapperFeatures

####jsonFactoryLookup

####serializationFeatures

####customSerializers

####deserializationFeatures

####customDeserializers


###Batch Properties for `jsonItemReader` Only
In addition to the above common properties, `jsonItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

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



###Batch Properties for `jsonItemWriter` Only
In addition to the above common properties, `jsonItemWriter` also supports the following batch properties in job xml:

####writeMode

####jsonGeneratorFeatures
`java.util.Map<String, String>`

####prettyPrinter
`java.lang.Class`

####outputDecorator
`java.lang.Class`


