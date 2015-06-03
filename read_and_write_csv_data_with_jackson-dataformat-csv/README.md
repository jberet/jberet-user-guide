# Read and Write CSV Data with jackson-dataformat-csv

In addition to item reader and writer based on super-csv library, jberet-support also contains `jacksonCsvItemReader` and `jacksonCsvItemWriter`, which implement reading from and writing to CSV using jackson-dataformat-csv library. This offers a convenient alternative especially for those applications that already depend on jackson family of libraries.

The following dependency is required for `jacksonCsvItemReader` and `jacksonCsvItemWriter`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-jaxb-annotations</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-csv</artifactId>
</dependency>
```

## Configure `jacksonCsvItemReader` and `jacksonCsvItemWriter` in job xml

The following is a partial example job xml defining `jacksonCsvItemReader` and `jacksonCsvItemWriter` with selected batch properties:

```xml
<job id="MovieTestWithJacksonCsv" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="MovieTestWithJacksonCsv.step1">
        <chunk>
            <reader ref="jacksonCsvItemReader">
                <properties>
                    <property name="resource" value="movies-2012.csv"/>
                    <property name="start" value="#{jobParameters['start']}"/>
                    <property name="end" value="#{jobParameters['end']}"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>
                    <property name="columns" value="#{jobParameters['columns']}"/>
                    <property name="useHeader" value="#{jobParameters['useHeader']}?:false;"/>
                </properties>
            </reader>
            <writer ref="jacksonCsvItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>
                    <property name="writeMode" value="overwrite"/>
                    <property name="columns" value="#{jobParameters['columns']}"/>
                    <property name="useHeader" value="#{jobParameters['useHeader']}?:false;"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

##Batch Configuration Properties for Both `jacksonCsvItemReader` and `jacksonCsvItemWriter`

###resource
The resource to read from (for batch readers), or write to (for batch writers).

###beanType
`java.lang.Class`

Specifies a fully-qualified class or interface name that maps to a row of the source CSV file. For example,

* a custom java type that represents data item and serves as the CSV schema class
* `java.util.Map`
* `java.util.List`
* `java.lang.String[]`
* `com.fasterxml.jackson.databind.JsonNode`

When using `java.util.List` or `java.lang.String[]` for reading, it is deemed raw access, and CSV schema will not be configured and any schema-related properties are ignored. Specifically, CSV header and comment lines are read as raw access content.

###columns

Specifies CSV schema in one of the 2 ways:
* columns = "&lt;fully-qualified class name&gt;":

CSV schema is defined in the named POJO class, which typically has class-level annotation com.fasterxml.jackson.annotation.JsonPropertyOrder to define property order corresponding to CSV column order.

* columns = "&lt;comma-separated list of column names, each of which may be followed by a space and column type&gt;":
    
use the value to manually build CSV schema. Valid column types are defined in `com.fasterxml.jackson.dataformat.csv.CsvSchema.ColumnType`, including:
  * `STRING`
  * `STRING_OR_LITERAL`
  * `NUMBER`
  * `NUMBER_OR_STRING`
  * `BOOLEAN`
  * `ARRAY`
    
For complete list and descriptioin, see `com.fasterxml.jackson.dataformat.csv.CsvSchema.ColumnType` javadoc.

For example,

 `columns = "org.jberet.support.io.StockTrade"`
 
 `columns = "firstName STRING, lastName STRING, age NUMBER"`
 

In `jacksonCsvItemReader`, if this property is not defined and `useHeader` is true (CSV input has a header), the header is used to create CSV schema. However, when `beanType` is `java.util.List` or `java.lang.String[]`, the reader is considered raw access, and all schema-related properties are ignored.

This property is optional for reader and required for writer class.

See Also: `com.fasterxml.jackson.dataformat.csv.CsvSchema`

###useHeader
`boolean`

whether the first line of physical document defines column names (true) or not (false): if enabled, parser will take first-line values to define column names; and generator will output column names as the first line. Optional property.

For `jacksonCsvItemReader`, if `beanType` is `java.util.List` or `java.lang.String[]`, it is considered raw access, `useHeader` property is ignored and no CSV schema is used.

valid values are true or false, and the default is false.

See Also: `com.fasterxml.jackson.dataformat.csv.CsvSchema`

###quoteChar

Character used for quoting values that contain quote characters or linefeeds. Optional property and defaults to " (double-quote character).

See Also: `com.fasterxml.jackson.dataformat.csv.CsvSchema`

###columnSeparator

Character used to separate values.

Optional property and defaults to , (comma character). Other commonly used values include tab (`\t`) and pipe (`|`)

See Also: `com.fasterxml.jackson.dataformat.csv.CsvSchema`


###Other Jackson Configuration
The following batch properties can be used to configure jackson json objects such as `JsonFactory`, `ObjectMapper`, serialization, deserialization, and custom jackson modules.  
* `jsonFactoryFeatures`
* `mapperFeatures`
* `jsonFactoryLookup`
* `serializationFeatures`
* `customSerializers`
* `deserializationFeatures`
* `customDeserializers`
* `customDataTypeModules`

See Chapter JsonItemReader and JsonItemWriter for more details.


##Batch Configuration Properties for `jacksonCsvItemReader` Only
In addition to the common properties listed above, `jacksonCsvItemReader` also supports the following batch properties:

###skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO.  Optional property and defaults to false, i.e., the reader will validate data POJO bean where appropriate.

###start
`int`

Specifies the start position (a positive integer starting from `1`) to read the data. If reading from the beginning of the input CSV, there is no need to specify this property.

###end
`int`

Specify the end position in the data set (inclusive). Optional property, and defaults to `Integer.MAX_VALUE`. If reading till the end of the input CSV, there is no need to specify this property.

###skipFirstDataRow

Whether the first data line (either first line of the document, if `useHeader`=`false`, or second, if `useHeader`=`true`) should be completely ignored by parser. Needed to support CSV-like file formats that include additional non-data content before real data begins (specifically some database dumps do this)

Optional property, valid values are `true` and `false` and defaults to `false`.

See Also: `com.fasterxml.jackson.dataformat.csv.CsvSchema`

###escapeChar
Character, if any, used to escape values. Most commonly defined as backslash (`\`). Only used by parser; generator only uses quoting, including doubling up of quotes to indicate quote char itself. Optional protected and defaults to `null`.

See Also: `com.fasterxml.jackson.dataformat.csv.CsvSchema`

###jsonParserFeatures
`java.util.Map<String, String>`

A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.core.JsonParser` features. Optional property and defaults to `null`. For example,

`ALLOW_COMMENTS=true, ALLOW_YAML_COMMENTS=true, ALLOW_NUMERIC_LEADING_ZEROS=true, STRICT_DUPLICATE_DETECTION=true`
 
See Also: `com.fasterxml.jackson.core.JsonParser.Feature`

###csvParserFeatures
`java.util.Map<String, String>`

A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.dataformat.csv.CsvParser.Feature`. Optional property and defaults to null. For example,

 `TRIM_SPACES=false, WRAP_AS_ARRAY=false`
 
See Also: `com.fasterxml.jackson.dataformat.csv.CsvParser.Feature`

###deserializationProblemHandlers
A comma-separated list of fully-qualified names of classes that implement `com.fasterxml.jackson.databind.deser.DeserializationProblemHandler`, which can be registered to get called when a potentially recoverable problem is encountered during deserialization process. Handlers can try to resolve the problem, throw an exception or do nothing. Optional property and defaults to `null`. For example,

 `org.jberet.support.io.JsonItemReaderTest$UnknownHandler, org.jberet.support.io.JsonItemReaderTest$UnknownHandler2`
 
See Also: `com.fasterxml.jackson.databind.deser.DeserializationProblemHandler`, `com.fasterxml.jackson.databind.ObjectMapper#addHandler(com.fasterxml.jackson.databind.deser.DeserializationProblemHandler)`

###inputDecorator
`java.lang.Class`

Fully-qualified name of a class that extends `com.fasterxml.jackson.core.io.InputDecorator`, which can be used to decorate input sources. Typical use is to use a filter abstraction (filtered stream, reader) around original input source, and apply additional processing during read operations. Optional property and defaults to null. For example,

 `org.jberet.support.io.JsonItemReaderTest$NoopInputDecorator`

See Also: `com.fasterxml.jackson.core.JsonFactory#setInputDecorator(com.fasterxml.jackson.core.io.InputDecorator)`, `com.fasterxml.jackson.core.io.InputDecorator`



##Batch Configuration Properties for `jacksonCsvItemWriter` Only
In addition to the common properties listed above, `jacksonCsvItemWriter` also supports the following batch properties:

###nullValue

When asked to write Java `null`, this String value will be used instead. Optional property and defaults to empty string.

See Also `com.fasterxml.jackson.dataformat.csv.CsvSchema`


###writeMode
Instructs `csvItemWriter`, when the target CSV resource already exists, whether to append to, or overwrite the existing resource, or fail. Valid values are: 
* `append` (default)
* `overwrite`
* `failIfExists`

###lineSeparator

Character used to separate data rows. Only used by generator; parser accepts three standard linefeeds (`\r`, `\r\n`, `\n`). Optional protected and defaults to `\n`.

See Also: `com.fasterxml.jackson.dataformat.csv.CsvSchema`

###jsonGeneratorFeatures
`java.util.Map<String, String>`

A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.core.JsonGenerator` features. Optional property and defaults to null. Keys and values must be defined in `com.fasterxml.jackson.core.JsonGenerator.Feature`. For example,

 `WRITE_BIGDECIMAL_AS_PLAIN=true, WRITE_NUMBERS_AS_STRINGS=true, QUOTE_NON_NUMERIC_NUMBERS=false`
 
See Also: `com.fasterxml.jackson.core.JsonGenerator.Feature`

###csvGeneratorFeatures
`java.util.Map<String, String>`

A comma-separated list of key-value pairs that specify `com.fasterxml.jackson.dataformat.csv.CsvGenerator.Feature`. Optional property and defaults to `null`. For example,

 `STRICT_CHECK_FOR_QUOTING=false, OMIT_MISSING_TAIL_COLUMNS=false, ALWAYS_QUOTE_STRINGS=false`
 
See Also: `com.fasterxml.jackson.dataformat.csv.CsvGenerator.Feature`

###outputDecorator
`java.lang.Class`

Fully-qualified name of a class that implements `com.fasterxml.jackson.core.io.OutputDecorator`, which can be used to decorate output destinations. Typical use is to use a filter abstraction (filtered output stream, writer) around original output destination, and apply additional processing during write operations. Optional property and defaults to `null`. For example,

 `org.jberet.support.io.JsonItemReaderTest$NoopOutputDecorator`
 
See Also: `com.fasterxml.jackson.core.io.OutputDecorator`, `com.fasterxml.jackson.core.JsonFactory#setOutputDecorator(com.fasterxml.jackson.core.io.OutputDecorator)`
