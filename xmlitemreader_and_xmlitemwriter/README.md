# XmlItemReader and XmlItemWriter

jberet-support module includes `xmlItemReader` and `xmlItemWriter` to handle XML chunk-oriented data. `xmlItemReader` reads and binds data to instances of custom POJO bean provided by the batch application. 

`xmlItemReader` and `xmlItemWriter` leverages `jackson` and `jackson-dataformat-xml` libraries to handle XML data parsing and binding. The following `jackson` dependencies are required:

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

<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
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
`xmlItemReader` and `xmlItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type `String`, unless noted otherwise. The following is an example job xml that references `xmlItemReader` and `xmlItemWriter`:

```xml
<job id="XmlItemReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="XmlItemReaderTest.step1">
        <chunk item-count="100">
            <reader ref="xmlItemReader">
                <properties>
                    <property name="resource" value="#{jobParameters['resource']}"/>
                    <property name="start" value="1"/>
                    <property name="end" value="#{jobParameters['end']}"/>
                    <property name="beanType" value="org.jberet.support.io.Movie"/>
                </properties>
            </reader>
            <writer ref="xmlItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="rootElementName" value="movies"/>
                    <property name="writeMode" value="#{jobParameters['writeMode']}?:overwrite;"/>
                    <!--<property name="prettyPrinter" value="com.fasterxml.jackson.core.util.MinimalPrettyPrinter"/>-->
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###Batch Properties for Both `xmlItemReader` and `xmlItemWriter`

####resource

The resource to read from (for batch readers), or write to (for batch writers). 

####xmlFactoryLookup

JNDI lookup name for `com.fasterxml.jackson.dataformat.xml.XmlFactory`, which is used for constructing `com.fasterxml.jackson.dataformat.xml.deser.FromXmlParser` in `xmlItemReader` and `com.fasterxml.jackson.dataformat.xml.ser.ToXmlGenerator` in `xmlItemWriter`. See class [org.jberet.support.io.XmlFactoryObjectFactory](http://docs.jboss.org/jberet/latest/javadoc/jberet-support/org/jberet/support/io/XmlFactoryObjectFactory.html) for more details.

####customDataTypeModules

A comma-separated list of Jackson datatype module type ids that extend `com.fasterxml.jackson.databind.Module`. These modules will be registered with `xmlMapper`. For example,

`com.fasterxml.jackson.datatype.joda.JodaModule,
com.fasterxml.jackson.datatype.jsr353.JSR353Module,
com.fasterxml.jackson.datatype.jsr310.JSR310Module,
com.fasterxml.jackson.module.afterburner.AfterburnerModule`
 

###Batch Properties for `xmlItemReader` Only
In addition to the above common properties, `xmlItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####beanType
`java.lang.Class`

The bean class that represents individual data item in the `resource` XML, and the `readItem()` method reads one item at a time and binds it to the provided bean type. Required property. For example,

* `org.jberet.support.io.StockTrade`
* `org.jberet.support.io.Person`
* `my.own.custom.ItemBean`


####start
`int`

Specifies the start position (a positive integer starting from `1`) to read the data. If reading from the beginning of the input XML, there is no need to specify this property.

####end
`int`

Specify the end position in the data set (inclusive). Optional property, and defaults to `Integer.MAX_VALUE`. If reading till the end of the input XML, there is no need to specify this property.

####inputDecorator
`java.lang.Class`

Fully-qualified class name of `com.fasterxml.jackson.core.io.InputDecorator`, which can be used to decorate input sources. Optional property, and defaults to `null`. See `com.fasterxml.jackson.core.io.InputDecorator`, and  `com.fasterxml.jackson.core.JsonFactory#setInputDecorator(com.fasterxml.jackson.core.io.InputDecorator)` for more details.

####xmlTextElementName
Alternate "virtual name" to use for XML CDATA segments; that is, text values. Optional property and defaults to `null` (empty String, "", is used as the virtual name). See `com.fasterxml.jackson.dataformat.xml.JacksonXmlModule#setXMLTextElementName(java.lang.String)` for more details.

###Batch Properties for `xmlItemWriter` Only
In addition to the above common properties, `xmlItemWriter` also supports the following batch properties in job xml:

####writeMode
Instructs this class, when the target XML resource already exists, whether to append to, or overwrite the existing resource, or fail. Valid values are `"append"`, `"overwrite"`, and `"failIfExists"`. Optional property, and defaults to append.

####defaultUseWrapper
Whether use wrapper for indexed (List, array) properties or not, if there are no explicit annotations. Optional property, valid values are `"true"` and `"false"`, and defaults to `"true"`. See `com.fasterxml.jackson.dataformat.xml.JacksonXmlModule#setDefaultUseWrapper(boolean)`, and  ` com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlElementWrapper` for more details.

####rootElementName
Local name of the output XML root element. Required property. See `javax.xml.stream.XMLStreamWriter#writeStartElement` for more details.

####rootElementPrefix
The prefix of the XML root element tag. Optional property and defaults to `null`. See `javax.xml.stream.XMLStreamWriter#writeStartElement(java.lang.String, java.lang.String, java.lang.String)` for more details.

####rootElementNamespaceURI
The namespaceURI of the prefix to use. Optional property and defaults to `null`. See `javax.xml.stream.XMLStreamWriter#writeStartElement(java.lang.String, java.lang.String, java.lang.String)`, and `javax.xml.stream.XMLStreamWriter#writeStartElement(java.lang.String, java.lang.String)` for more details.

####prettyPrinter
`java.lang.Class`

The fully-qualified class name of `com.fasterxml.jackson.core.PrettyPrinter`, which implements pretty printer functionality, such as indentation. Optional property and defaults to `null` (default pretty printer is used). See `com.fasterxml.jackson.core.PrettyPrinter`, and `com.fasterxml.jackson.dataformat.xml.ser.ToXmlGenerator#setPrettyPrinter(com.fasterxml.jackson.core.PrettyPrinter)` for more details.

####outputDecorator
`java.lang.Class`

The fully-qualified class name of `com.fasterxml.jackson.core.io.OutputDecorator`, which can be used to decorate output destinations. Optional property and defaults to null. See `com.fasterxml.jackson.core.io.OutputDecorator`, and `com.fasterxml.jackson.core.JsonFactory#setOutputDecorator(com.fasterxml.jackson.core.io.OutputDecorator)` for more details.

