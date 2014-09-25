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


###Batch Properties for `xmlItemReader` Only
In addition to the above common properties, `xmlItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####beanType
`java.lang.Class`

####start
`int`

####end
`int`

####inputDecorator
`java.lang.Class`

####xmlTextElementName


###Batch Properties for `xmlItemWriter` Only
In addition to the above common properties, `xmlItemWriter` also supports the following batch properties in job xml:

####writeMode

####defaultUseWrapper

####rootElementName

####rootElementPrefix

####rootElementNamespaceURI

####prettyPrinter
`java.lang.Class`

####outputDecorator
`java.lang.Class`


