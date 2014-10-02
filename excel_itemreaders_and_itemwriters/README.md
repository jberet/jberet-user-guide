# Excel ItemReaders and ItemWriters

jberet-module includes `excelEventItemReader`, `excelUserModelItemReader`, `excelUserModelItemWriter`, `excelStreamingItemReader`, `excelStreamingItemWriter` for working with Excel files, including both binary Excel files (.xls) and OOXML Excel files (.xlsx). They are all based on Apache POI library for low-level Excel operations.

| Name | Excel File Type | Corresponding POI Component | Benefits |
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
