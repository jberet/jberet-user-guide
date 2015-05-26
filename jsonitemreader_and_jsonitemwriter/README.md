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

<!-- POJO beans may contain either jackson annotations, or JAXB annotations -->
<!-- Include this dependency if JAXB annotation introspector is needed -->
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-jaxb-annotations</artifactId>
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
A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.core.JsonFactory.Feature`s. Optional property and defaults to `null`. Only keys and values defined in `com.fasterxml.jackson.core.JsonFactory.Feature` are accepted. For example,

`INTERN_FIELD_NAMES=false, CANONICALIZE_FIELD_NAMES=false`

See `com.fasterxml.jackson.core.JsonFactory.Feature`, and `org.jberet.support.io.NoMappingJsonFactoryObjectFactory.configureJsonFactoryFeatures(com.fasterxml.jackson.core.JsonFactory, java.lang.String)` for more details.

####mapperFeatures
A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.databind.ObjectMapper` features. Optional property and defaults to `nul`l. Only keys and values defined in `com.fasterxml.jackson.databind.MapperFeature` are accepted. For example,

`USE_ANNOTATIONS=false, AUTO_DETECT_FIELDS=false, AUTO_DETECT_GETTERS=true`
 
See `com.fasterxml.jackson.databind.MapperFeature`, and `org.jberet.support.io.MappingJsonFactoryObjectFactory.configureMapperFeatures(com.fasterxml.jackson.databind.ObjectMapper, java.lang.String)` for more details.

####jsonFactoryLookup
JNDI lookup name for `com.fasterxml.jackson.core.JsonFactory`, which is used for constructing `com.fasterxml.jackson.core.JsonParser` in `jsonItemReader` and `com.fasterxml.jackson.core.JsonGenerator` in `jsonItemWriter`. Optional property and defaults to `null`. When this property is specified, its value is used to look up an instance of `com.fasterxml.jackson.core.JsonFactory`, which is typically created and administrated externally (e.g., inside application server). Otherwise, a new instance of `com.fasterxml.jackson.core.JsonFactory` is created instead of lookup.

####serializationFeatures
A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.databind.SerializationFeature`s. Optional property and defaults to `null`. Only keys and values defined in `com.fasterxml.jackson.databind.SerializationFeature` are accepted. For example,

`WRAP_ROOT_VALUE=true, INDENT_OUTPUT=true, FAIL_ON_EMPTY_BEANS=false`
  
See `com.fasterxml.jackson.databind.SerializationFeature`, and `org.jberet.support.io.MappingJsonFactoryObjectFactory.configureSerializationFeatures(com.fasterxml.jackson.databind.ObjectMapper, java.lang.String) ` for more details.

####customSerializers
A comma-separated list of fully-qualified name of classes that implement `com.fasterxml.jackson.databind.JsonSerializer`, which can serialize Objects of arbitrary types into JSON. For example,

`org.jberet.support.io.JsonItemReaderTest$JsonSerializer, org.jberet.support.io.JsonItemReaderTest$JsonSerializer2`
 
See `com.fasterxml.jackson.databind.JsonSerializer`, and `org.jberet.support.io.MappingJsonFactoryObjectFactory.configureCustomSerializersAndDeserializers(com.fasterxml.jackson.databind.ObjectMapper, java.lang.String, java.lang.String, java.lang.ClassLoader)` for more details.

####deserializationFeatures
A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.databind.DeserializationFeature`s. Optional property and defaults to `null`. Only keys and values defined in `com.fasterxml.jackson.databind.DeserializationFeature` are accepted. For example,

`USE_BIG_DECIMAL_FOR_FLOATS=true, USE_BIG_INTEGER_FOR_INTS=true, USE_JAVA_ARRAY_FOR_JSON_ARRAY=true`
 
See `com.fasterxml.jackson.databind.DeserializationFeature`, and `org.jberet.support.io.MappingJsonFactoryObjectFactory.configureDeserializationFeatures(com.fasterxml.jackson.databind.ObjectMapper, java.lang.String)` for more details.

####customDeserializers
A comma-separated list of fully-qualified name of classes that implement `com.fasterxml.jackson.databind.JsonDeserializer`, which can deserialize Objects of arbitrary types from JSON. For example,

`org.jberet.support.io.JsonItemReaderTest$JsonDeserializer, org.jberet.support.io.JsonItemReaderTest$JsonDeserializer2`
 
See `com.fasterxml.jackson.databind.JsonDeserializer`, and `org.jberet.support.io.MappingJsonFactoryObjectFactory.configureCustomSerializersAndDeserializers(com.fasterxml.jackson.databind.ObjectMapper, java.lang.String, java.lang.String, java.lang.ClassLoader)` for more details.

####customDataTypeModules
A comma-separated list of Jackson datatype module type ids that extend `com.fasterxml.jackson.databind.Module`. These modules will be registered with `objectMapper`. For example,

`com.fasterxml.jackson.datatype.joda.JodaModule, com.fasterxml.jackson.datatype.jsr353.JSR353Module, com.fasterxml.jackson.datatype.jsr310.JSR310Module, com.fasterxml.jackson.module.afterburner.AfterburnerModule`
 

###Batch Properties for `jsonItemReader` Only
In addition to the above common properties, `jsonItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####beanType
`java.lang.Class`

The bean type that represents individual data item in the source Json `resource`. Required property, and valid values are:
`
* any custom bean type, for example `org.jberet.support.io.StockTrade`
* `java.util.Map`
* `com.fasterxml.jackson.databind.JsonNode`

####start
`int`

Specifies the start position (a positive integer starting from `1`) to read the data. If reading from the beginning of the input Json resource, there is no need to specify this property.

####end
`int`

Specify the end position in the data set (inclusive). Optional property, and defaults to `Integer.MAX_VALUE`. If reading till the end of the input Json resource, there is no need to specify this property.

####jsonParserFeatures
`java.util.Map<String, String>`

A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.core.JsonParser` features. Optional property and defaults to `null`. For example,

`ALLOW_COMMENTS=true, ALLOW_YAML_COMMENTS=true, ALLOW_NUMERIC_LEADING_ZEROS=true, STRICT_DUPLICATE_DETECTION=true`
 
See `com.fasterxml.jackson.core.JsonParser.Feature` for more details.

####deserializationProblemHandlers
A comma-separated list of fully-qualified names of classes that implement `com.fasterxml.jackson.databind.deser.DeserializationProblemHandler`, which can be registered to get called when a potentially recoverable problem is encountered during deserialization process. Handlers can try to resolve the problem, throw an exception or do nothing. Optional property and defaults to `null`. For example,

`org.jberet.support.io.JsonItemReaderTest$UnknownHandler, org.jberet.support.io.JsonItemReaderTest$UnknownHandler2`
 
See `com.fasterxml.jackson.databind.deser.DeserializationProblemHandler`, `com.fasterxml.jackson.databind.ObjectMapper#addHandler(com.fasterxml.jackson.databind.deser.DeserializationProblemHandler)`, `org.jberet.support.io.MappingJsonFactoryObjectFactory.configureDeserializationProblemHandlers(com.fasterxml.jackson.databind.ObjectMapper, java.lang.String, java.lang.ClassLoader)` for more details.

####inputDecorator
`java.lang.Class`

Fully-qualified name of a class that extends `com.fasterxml.jackson.core.io.InputDecorator`, which can be used to decorate input sources. Typical use is to use a filter abstraction (filtered stream, reader) around original input source, and apply additional processing during read operations. Optional property and defaults to `null`. For example,

`org.jberet.support.io.JsonItemReaderTest$NoopInputDecorator`
 
See `com.fasterxml.jackson.core.JsonFactory#setInputDecorator(com.fasterxml.jackson.core.io.InputDecorator)`, and `com.fasterxml.jackson.core.io.InputDecorator` for more details.


###Batch Properties for `jsonItemWriter` Only
In addition to the above common properties, `jsonItemWriter` also supports the following batch properties in job xml:

####writeMode
Instructs this class, when the target Json resource already exists, whether to append to, or overwrite the existing resource, or fail. Valid values are `append`, `overwrite`, and `failIfExists`. Optional property, and defaults to `append`.

####jsonGeneratorFeatures
`java.util.Map<String, String>`

A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.core.JsonGenerator` features. Optional property and defaults to `null`. Keys and values must be defined in `com.fasterxml.jackson.core.JsonGenerator.Feature`. For example,

`WRITE_BIGDECIMAL_AS_PLAIN=true, WRITE_NUMBERS_AS_STRINGS=true, QUOTE_NON_NUMERIC_NUMBERS=false`
 
See `com.fasterxml.jackson.core.JsonGenerator.Feature` for more details.

####prettyPrinter
`java.lang.Class`

Fully-qualified name of a class that implements `com.fasterxml.jackson.core.PrettyPrinter`, which implements pretty printer functionality, such as indentation. Optional property and defaults to `null` (the system default pretty printer is used). For example,

`com.fasterxml.jackson.core.util.MinimalPrettyPrinter`
 
See `com.fasterxml.jackson.core.PrettyPrinter`, and `com.fasterxml.jackson.core.JsonGenerator#setPrettyPrinter(com.fasterxml.jackson.core.PrettyPrinter)` for more details.

####outputDecorator
`java.lang.Class`

Fully-qualified name of a class that implements `com.fasterxml.jackson.core.io.OutputDecorator`, which can be used to decorate output destinations. Typical use is to use a filter abstraction (filtered output stream, writer) around original output destination, and apply additional processing during write operations. Optional property and defaults to `null`. For example,

`org.jberet.support.io.JsonItemReaderTest$NoopOutputDecorator`
 
See `com.fasterxml.jackson.core.io.OutputDecorator`, and `com.fasterxml.jackson.core.JsonFactory#setOutputDecorator(com.fasterxml.jackson.core.io.OutputDecorator)` for more details.
