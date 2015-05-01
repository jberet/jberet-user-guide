# Excel ItemReaders and ItemWriters

jberet-module includes `excelEventItemReader`, `excelUserModelItemReader`, `excelUserModelItemWriter`, `excelStreamingItemReader`, `excelStreamingItemWriter` for working with Excel files, including both binary Excel files (.xls) and OOXML Excel files (.xlsx). They are all based on Apache POI library for low-level Excel operations.

| Name | Excel File Type | Underlying POI or XML API | Benefits |
| -- | -- | -- |
| `excelEventItemReader` | .xls | POI event model API | small memory footprint |
| `excelUserModelItemReader` | .xls & .xlsx | POI user model API | simple API for in-memory access |
| `excelUserModelItemWriter` | .xls & .xlsx | POI user model API | simple API for in-memory content generation |
| `excelStreamingItemReader` | .xlsx | POI XSSF streaming reader API & StAX | stream read large data set |
| `excelStreamingItemWriter` | .xlsx | POI SXSSF (buffered streaming) API | write large data set | 


The following dependencies are required:
```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>${version.org.apache.poi}</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>${version.commons-codec}</version>
</dependency>

<!-- XML-related libraries needed for working with OOXML Excel files -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>${version.org.apache.poi}</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml-schemas</artifactId>
    <version>${version.org.apache.poi}</version>
</dependency>
<dependency>
    <groupId>org.apache.xmlbeans</groupId>
    <artifactId>xmlbeans</artifactId>
    <version>${version.org.apache.xmlbeans}</version>
    <exclusions>
        <exclusion>
            <groupId>stax</groupId>
            <artifactId>stax-api</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>${version.dom4j}</version>
    <exclusions>
        <exclusion>
            <groupId>xml-apis</groupId>
            <artifactId>xml-apis</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```
##Batch Configuration in Job XML
`excelEventItemReader`, `excelUserModelItemReader`, `excelUserModelItemWriter`, `excelStreamingItemReader`, `excelStreamingItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type `String`, unless noted otherwise. The following is example job xml files that reference these readers and writers:

###`excelEventItemReader`
```xml
<job id="ExcelReaderIBMTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="ExcelReaderIBMTest.step1">
        <chunk item-count="100">
            <reader ref="excelEventItemReader">
                <properties>
                    <property name="resource" value="IBM_unadjusted.xls"/>
                    <property name="beanType" value="org.jberet.support.io.StockTrade"/>
                    <property name="sheetName" value="#{jobParameters['sheetName']}"/>
                    <property name="start" value="1"/>
                    <property name="headerRow" value="0"/>

                    <!-- do not take header values; use headerRow (its default value 0) instead -->
                    <!--<property name="header" value="#{jobParameters['header']}"/>-->

                    <!-- example how to skip performing Bean Validation on the data POJO -->
                    <property name="skipBeanValidation" value="true"/>
                </properties>
            </reader>
            <writer ref="csvItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="writeMode" value="overwrite"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>

                    <!-- for csvItemWriter, use the header passed from jobParameters -->
                    <property name="header" value="#{jobParameters['header']}"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```
###`excelUserModelItemReader`
```xml
<job id="ExcelReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="ExcelReaderTest.step1">
        <chunk item-count="100">
            <reader ref="excelUserModelItemReader">
                <properties>
                    <property name="resource" value="person-movies.xlsx"/>
                    <property name="beanType" value="org.jberet.support.io.Person"/>
                    <property name="sheetName" value="Sheet1"/>
                    <property name="start" value="1"/>
                    <property name="headerRow" value="0"/>

                    <!-- need to ignore unknown properties as some additional properties in data -->
                    <property name="deserializationFeatures" value="FAIL_ON_UNKNOWN_PROPERTIES=false"/>
                </properties>
            </reader>
            
            <writer ref="jsonItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="writeMode" value="overwrite"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###`excelUserModelItemWriter`
```xml
<job id="ExcelWriterTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="ExcelWriterTest.step1">
        <chunk item-count="100">
            <reader ref="jsonItemReader">
                <properties>
                    <property name="resource" value="#{jobParameters['resource']}"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>
                    <property name="start" value="#{jobParameters['start']}"/>
                    <property name="end" value="#{jobParameters['end']}"/>
                </properties>
            </reader>
            
            <writer ref="excelUserModelItemWriter">
                <properties>
                    <property name="resource" value="testMoviesBeanTypeFullTemplate.xlsx"/>
                    <property name="writeMode" value="overwrite"/>
                    <property name="beanType" value="org.jberet.support.io.Movie"/>
                    <property name="sheetName" value="Movies 2012"/>
                    <property name="templateResource" value="person-movies-company.xltx"/>
                    <property name="templateSheetName" value="Movies"/>
                    <property name="templateHeaderRow" value="0"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###`excelStreamingItemReader`
```xml
<reader ref="excelStreamingItemReader">
    <properties>
        <property name="resource" value="person-movies.xlsx"/>
        <property name="beanType" value="org.jberet.support.io.Person"/>
        <property name="sheetName" value="Sheet1"/>
        <property name="start" value="1"/>
        <property name="headerRow" value="0"/>

        <!-- need to ignore unknown properties as some additional properties in data -->
        <property name="deserializationFeatures" value="FAIL_ON_UNKNOWN_PROPERTIES=false"/>
    </properties>
</reader>
```

###`excelStreamingItemWriter`
```xml
<job id="ExcelStreamingWriterTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="ExcelStreamingWriterTest.step1">
        <chunk item-count="100">
            <reader ref="csvItemReader">
                <properties>
                    <property name="resource" value="#{jobParameters['resource']}"/>
                    <property name="headerless" value="#{jobParameters['headerless']}"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>
                    <property name="start" value="#{jobParameters['start']}"/>
                    <property name="end" value="#{jobParameters['end']}"/>
                    <property name="nameMapping" value="#{jobParameters['nameMapping']}"/>
                    <property name="cellProcessors" value="#{jobParameters['cellProcessors']}"/>
                </properties>
            </reader>
            
            <writer ref="excelStreamingItemWriter">
                <properties>
                    <property name="resource" value="testMoviesBeanTypeFullStreaming.xlsx"/>
                    <property name="writeMode" value="overwrite"/>
                    <property name="beanType" value="org.jberet.support.io.Movie"/>
                    <property name="sheetName" value="Movies 2012"/>
                    <property name="serializationFeatures" value="WRITE_DATES_AS_TIMESTAMPS=false"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

---------------------

###Batch Properties for All Excel Readers and Writers

####resource
The resource to read from (for batch readers), or write to (for batch writers).

####beanType
`java.lang.Class`

####header
`java.lang.String[]`

Specifies the header as an ordered string array. For reader, header information must be specified with either this property or `ExcelUserModelItemReader.headerRow` property. This property is typically specified when there is no header row in the Excel file. For example,

`"id, name, age"` specifies 1st column is `id`, 2nd column is `name` and 3rd column is `age`.

This is a required property for writer.

####sheetName
The optional name of the target sheet. When specified for a reader, it has higher precedence over `ExcelUserModelItemReader.sheetIndex`

####additional jackson-related batch properties
All Excel readers and writers use `jackson-databind` for data transformation behind the scene, and this step can be configured through the following batch configuration properties in job xml, though the defaults should suffice in most cases:

* `jsonFactoryFeatures`
* `mapperFeatures`
* `jsonFactoryLookup`
* `serializationFeatures`
* `customSerializers`
* `deserializationFeatures`
* `customDeserializers`
* `customDataTypeModules`
 
See Chapter JsonItemReader and JsonItemWriter for more details.

-----------------

###Batch Properties for `excelUserModelItemReader`
In additional to the shared batch properties listed above, `excelUserModelItemReader` can also be configured through the following batch properties:

####start
`int`

A positive integer indicating the start position in the input resource. It is optional and defaults to `0` (starting from the 1st data item). If a header row is present, the start point should be after the header row.

####end
`int`

A positive integer indicating the end position in the input resource. It is optional and defaults to `Integer.MAX_VALUE`.

####sheetIndex
`int`

The index (`0`-based) of the target sheet to read, defaults to `0`.

####headerRow
`java.lang.Integer`

The physical row number of the header.

-------------

###Batch Properties for `excelEventItemReader`
####All the shared batch properties for all Excel readers and writers

####All the batch properties for `excelUserModelItemReader`

####queueCapacity
`int`

the capacity of the queue used by `org.apache.poi.hssf.eventusermodel.HSSFListener` to hold pre-fetched data rows. Optional property and defaults to `ExcelEventItemReader.MAX_WORKSHEET_ROWS` (`65536`).

--------------

###Batch Properties for `excelStreamingItemReader`
####All the shared batch properties for all Excel readers and writers

####All the batch properties for `excelUserModelItemReader`

--------------

###Batch Properties for `excelUserModelItemWriter`
In addition to the shared batch properties for all Excel readers and writers, `excelUserModelItemWriter` also support the following batch properties:

####writeMode
Valid `writeMode` for this writer class is `overwrite` and `failIfExists`.

####templateResource
The resource of an existing Excel file or template file to be used as a base for generating output Excel. Its format is similar to `resource` batch property.

####templateSheetName
The sheet name in the template file to be used for generating output Excel. If `templateResource` is specified but this property is not specified, `templateSheetIndex` is used instead.

####templateSheetIndex
`int`

The sheet index (`0`-based) in the template file to be used for generating output Excel.

####templateHeaderRow
`java.lang.Integer`

The row number (`0`-based) of the header in the template sheet. If `header` property is provided in job xml file, then this property is ignored. Otherwise, it is used to retrieve header values.

------------

###Batch Properties for `excelStreamingItemWriter`
####All the shared batch properties for all Excel readers and writers

####All the batch properties for `excelUserModelItemWriter`

####compressTempFiles
`java.lang.Boolean`

Whether to compress the temp files in the course of generating Excel file, defaults to `false`.


