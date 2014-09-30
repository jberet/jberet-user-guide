# BeanIO ItemReader and ItemWriter

jberet-support module includes `beanIOItemReader` and `beanIOItemWriter` that employs [BeanIO](http://beanio.org/) to read and write data in various formats that are supported by BeanIO, e.g., fixed length file, CSV file, XML, etc. They also support restart, ranged reading, custom error handler, and dynamic BeanIO mapping properties. 

The following BeanIO dependencies are required for `beanIOItemReader` and `beanIOItemWriter`:

```xml
<dependency>
    <groupId>org.beanio</groupId>
    <artifactId>beanio</artifactId>
    <version>${version.org.beanio}</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.ow2.asm</groupId>
    <artifactId>asm</artifactId>
    <version>${version.org.ow2.asm}</version>
</dependency>
```

##Batch Configuration Properties in Job XML

`beanIOItemReader` and `beanIOItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type `String`, unless noted otherwise. The following is an example job xml that references `beanIOItemReader` and `beanIOItemWriter`:

```xml
<job id="BeanIOReaderWriterTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="BeanIOReaderWriterTest.step1">
        <chunk item-count="1000000">
            <reader ref="beanIOItemReader">
                <properties>
                    <property name="resource" value="#{jobParameters['resource']}"/>
                    <property name="start" value="1"/>
                    <property name="end" value="1000"/>
                    <property name="streamName" value="persons"/>
                    <property name="streamMapping" value="person-beanio-mapping.xml"/>
                    <property name="mappingProperties" value="zipCodeFieldName=zipCode, zipCodeFieldType=string"/>
                    <property name="errorHandler" value="org.beanio.BeanReaderErrorHandlerSupport"/>
                    <!--<property name="locale" value="en_US"/>-->
                    <property name="charset" value="UTF-8"/>
                </properties>
            </reader>
            
            <writer ref="beanIOItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="streamName" value="persons"/>
                    <property name="streamMapping" value="person-beanio-mapping.xml"/>
                    <property name="mappingProperties" value="zipCodeFieldName=zipCode, zipCodeFieldType=string"/>
                    <property name="charset" value="UTF-8"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###Batch Properties for Both `beanIOItemReader` and `beanIOItemWriter`
####resource
The resource to read from (for batch readers), or write to (for batch writers). Some reader or writer implementations may choose to ignore this property and instead use other properties that are more appropritate.

####streamName
Name of the BeanIO stream defined in BeanIO mapping file. It corresponds to the batch job property.

####streamMapping
Location of the BeanIO mapping file, which can be a file path, a URL, or a resource loadable by the current class loader.

####streamFactoryLookup
JNDI name for looking up `org.beanio.StreamFactory` when running in application server. When `streamFactoryLookup` property is specified in job xml and hence injected into batch reader or writer class, `org.beanio.StreamFactory` will be looked up with JNDI, and `streamMapping` and `mappingProperties` will be ignored.

####mappingProperties
`java.util.Map`

User properties that can be used for property substitution in BeanIO mapping file. When used with batch job JSL properties, they provide dynamic BeanIO mapping attributes. For example,

1. In batch job client class, set the properties values as comma-separated key-value pairs:
```java
params.setProperty("mappingProperties", "zipCodeFieldName=zipCode, zipCodeFieldType=string");
```

2. In job xml file, make the properties available for `beanIOItemReader` or `beanIOItemWriter` via `@BatchProperty` injection:
```xml
<property name="mappingProperties" value="#{jobParameters['mappingProperties']}"/>
```

3. In BeanIO mapping file, reference the properties defined above:
```xml
<field name="${zipCodeFieldName}" type="${zipCodeFieldType}" length="5"/> 
```

####charset
The name of the character set to be used for reading and writing data, e.g., UTF-8. This property is optional, and if not set, the platform default charset is used.



###Batch Properties for `beanIOItemReader` Only
In addition to the above common properties, `beanIOItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####start
`int`

A positive integer indicating the start position in the input resource. It is optional and defaults to `1` (starting from the 1st data item).

####end
`int`

A positive integer indicating the end position in the input resource. It is optional and defaults to `Integer.MAX_VALUE`.

####errorHandler
`java.lang.Class`

A class implementing `BeanReaderErrorHandler` for handling exceptions thrown by a `BeanReader`.

####locale
The locale name for this `beanIOItemReader`


###Batch Properties for `beanIOItemWriter` Only
None
 
