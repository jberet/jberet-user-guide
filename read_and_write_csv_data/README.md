#Read and Write CSV Data
jberet-support module contains `CsvItemReader` and `CsvItemWriter` that reads and writes CSV (Comma-Separated Values) resources respectively. Batch applications can reference them by name `csvItemReader` and `csvItemWriter` in job xml. They can also be configured to handle data with other delimiters such as tab, vertical bar, etc.  The following dependency is required by `CsvItemReader` and `CsvItemWriter`:

```xml
<dependency>
    <groupId>net.sf.supercsv</groupId>
    <artifactId>super-csv</artifactId>
    <version>${version.net.sf.supercsv}</version>
</dependency>
```

jberet-support delegates most of the CSV data read, write and processing to [supercsv](http://supercsv.sourceforge.net/), and therefore the configuration of `csvItemReader` and `csvItemWriter` mirrors that of supercsv.

Another option for dealing with CSV data is to use write batch reader, processor or writer in script languages, which may have built-in support or libraries for CSV data format. For more details, refer to chapter Develop Batch Artifacts in Script Languages.

##Configure `csvItemReader` and `csvItemWriter` in job xml
The following is a sample job xml that references `csvItemReader` and `csvItemWriter` to read and write CSV data. Each batch property will be explained in the next section. Javadoc of [CsvItemReader](http://docs.jboss.org/jberet/latest/javadoc/jberet-support/org/jberet/support/io/CsvItemReader.html) and [CsvItemWriter](http://docs.jboss.org/jberet/latest/javadoc/jberet-support/org/jberet/support/io/BeanIOItemWriter.html) also contains details for each batch configuration property.
```xml
<job id="CsvReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="step1">
        <chunk item-count="100">
            <reader ref="csvItemReader">
                <properties>
                    <property name="resource" value="#{jobParameters['resource']}"/>
                    <property name="start" value="7"/>
                    <property name="end" value="9"/>
                    <property name="preference" value="STANDARD_PREFERENCE"/>
                    <property name="delimiterChar" value=","/>
                    <property name="quoteChar" value="|"/>
                    <property name="beanType" value="org.jberet.support.io.Person"/>
                    <property name="commentMatcher" value="starts with '#'"/>
                    <property name="nameMapping" value="number, gender, title, givenName"/>
                    <property name="cellProcessors" value= "
                        NotNull, UniqueHashCode, LMinMax(1, 99999); 
                        Token('male', 'M'), Token('female', 'F');
                        null; 
                        StrNotNullOrEmpty"/>
                </properties>
            </reader>
            <writer ref="csvItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <!--
                    <property name="writeMode" value="#{jobParameters['writeMode']}?:append;"/>
                    <property name="writeMode" value="#{jobParameters['writeMode']}?:failIfExists;"/>
                    -->
                    <property name="writeMode" value="overwrite"/>
                    <property name="preference" value="STANDARD_PREFERENCE"/>
                    <property name="delimiterChar" value=","/>
                    <property name="quoteChar" value="^"/>
                    <property name="beanType" value="java.util.Map"/>
                    <property name="header" value="number, gender, title, givenName"/>
                    <property name="writeComments" value="# Comments written by csv writer."/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

##Batch Configuration Properties for Both `csvItemReader` and `csvItemWriter`

###resource
The resource to read from (for batch readers), or write to (for batch writers).

###skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO.  Optional property and defaults to false, i.e., the reader will validate data POJO bean where appropriate.

###nameMapping
`java.lang.String[]`

###beanType
`java.lang.Class`

###preference

###quoteChar

###delimiterChar

###endOfLineSymbols

###surroundingSpacesNeedQuotes

###commentMatcher

###encoder

###quoteMode

###cellProcessors

###charset
The name of the character set to be used for reading and writing data, e.g., UTF-8. This property is optional, and if not set, the platform default charset is used.


##Batch Configuration Properties for `csvItemReader`
In addition to the common properties listed above, `csvItemReader` also supports the following batch properties:

###start
`int`

###end
`int`

###headerless
`boolean`


##Batch Configuration Properties for `csvItemWriter`
In addition to the common properties listed above, `csvItemWriter` also supports the following batch properties:

###header
`java.lang.String[]`

###writeComments

###writeMode



