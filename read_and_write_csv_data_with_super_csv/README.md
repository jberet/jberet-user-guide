#Read and Write CSV Data with super-csv
jberet-support module contains `csvItemReader` and `csvItemWriter` that reads and writes CSV (Comma-Separated Values) resources respectively. Batch applications can reference them by name `csvItemReader` and `csvItemWriter` in job xml. They can also be configured to handle data with other delimiters such as tab, vertical bar, etc. For fixed-length flat file, see Chapter BeanIO ItemReader and ItemWriter.

The following dependency is required by `csvItemReader` and `csvItemWriter`:

```xml
<dependency>
    <groupId>net.sf.supercsv</groupId>
    <artifactId>super-csv</artifactId>
    <version>${version.net.sf.supercsv}</version>
</dependency>
```

jberet-support delegates most of the CSV data read, write and processing to [supercsv](http://supercsv.sourceforge.net/), and therefore the configuration of `csvItemReader` and `csvItemWriter` mirrors that of supercsv.

Besides `csvItemReader` and `csvItemWriter`, JBeret also offers other options for dealing with CSV data:
* write batch reader, processor or writer in script languages, which may have built-in support or libraries for CSV data format. For more details, refer to chapter Develop Batch Artifacts in Script Languages.
* use `beanIOItemReader` and `beanIOItemWriter`, which handles common data formats such as CSV, XML, JSON. See Chapter BeanIO ItemReader and ItemWriter for details.

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

###nameMapping
`java.lang.String[]`

Specify the bean fields or map keys corresponding to CSV columns in the same order. Not used if `beanType` property is set to `java.util.List`. If the CSV column names exactly match bean fields or map keys, then no need to specify this property. If the CSV column names are missing or differ from bean fields or map keys, then this property is required. An example of `nameMapping` value:

`"number, gender, title, givenName, middleInitial, surname"`

###beanType
`java.lang.Class`

Specifies a fully-qualified class or interface name that maps to a row of the source CSV file. For example,

* `java.util.List`
* `java.util.Map`
* `org.jberet.support.io.Person`
* `my.own.BeanType`

###preference
Specifies one of the 4 predefined CSV preferences:

* `STANDARD_PREFERENCE`
* `EXCEL_PREFERENCE`
* `EXCEL_NORTH_EUROPE_PREFERENCE`
* `TAB_PREFERENCE`

###quoteChar
The quote character (used when a cell contains special characters, such as the delimiter char, a quote char, or spans multiple lines). See [CSV Preferences](http://supercsv.sourceforge.net/preferences.html). The default quoteChar is double quote (`"`). If " is present in the CSV data cells, specify quoteChar to some other characters, e.g., `|`.

###delimiterChar
The delimiter character (separates each cell in a row). See [CSV Preferences](http://supercsv.sourceforge.net/preferences.html).

###endOfLineSymbols
The end of line symbols to use when writing (Windows, Mac and Linux style line breaks are all supported when reading, so this preference won't be used at all for reading). See [CSV Preferences](http://supercsv.sourceforge.net/preferences.html). See CSV Preferences.).

###surroundingSpacesNeedQuotes
Whether spaces surrounding a cell need quotes in order to be preserved (see below). The default value is false (quotes aren't required). See [CSV Preferences](http://supercsv.sourceforge.net/preferences.html). The default value is false (quotes aren't required). See CSV Preferences.).

###commentMatcher
Specifies a CommentMatcher for reading CSV resource. The CommentMatcher determines whether a line should be considered a comment. See [CSV Preferences](http://supercsv.sourceforge.net/preferences.html). For example,

* `"startsWith #"`
* `"matches 'regexp'"`
* `"my.own.CommentMatcherImpl"`

###encoder
Specifies a custom encoder when writing CSV. For example,
* `default`
* `select 1, 2, 3`
* `select true, true, false`
* `column 1, 2, 3`
* `column true, true, false`
* `my.own.MyCsvEncoder`
    
See [CSV Preferences](http://supercsv.sourceforge.net/preferences.html).

###quoteMode
Allows you to enable surrounding quotes for writing (if a column wouldn't normally be quoted because it doesn't contain special characters). For example,

* `default`
* `always`
* `select 1, 2, 3`
* `select true, true, false`
* `column 1, 2, 3`
* `column true, true, false`
* `my.own.MyQuoteMode`

See [CSV Preferences](http://supercsv.sourceforge.net/preferences.html).

###cellProcessors
Specifies a list of cell processors, one for each column. See [Super CSV docs](http://supercsv.sourceforge.net/cell_processors.html) for supported cell processor types. The rules and syntax are as follows:

* The size of the resultant list must equal to the number of CSV columns.
* Cell processors appear in the same order as CSV columns.
* If no cell processor is needed for a column, enter `null`.
* Each column may have `null`, `1`, `2`, or multiple cell processors, separated by comma (`,`)
* Cell processors for different columns must be separated with semi-colon (`;`).
* Cell processors may contain parameters enclosed in parenthesis, and multiple parameters are separated with comma (`,`).
string literals in cell processor parameters must be enclosed within single quotes, e.g., `'xxx'`

For example, to specify cell processors for 5-column CSV:
```xml
 <property name = "cellProcessors" value = "
      null;
      Optional, StrMinMax(1, 20);
      ParseLong;
      NotNull;
      Optional, ParseDate('dd/MM/yyyy')
 "/>
 ```
 
###charset
The name of the character set to be used for reading and writing data, e.g., UTF-8. This property is optional, and if not set, the platform default charset is used.


##Batch Configuration Properties for `csvItemReader` Only
In addition to the common properties listed above, `csvItemReader` also supports the following batch properties:

###skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO.  Optional property and defaults to false, i.e., the reader will validate data POJO bean where appropriate.

###start
`int`

Specifies the start position (a positive integer starting from `1`) to read the data. If reading from the beginning of the input CSV, there is no need to specify this property.

###end
`int`

Specify the end position in the data set (inclusive). Optional property, and defaults to `Integer.MAX_VALUE`. If reading till the end of the input CSV, there is no need to specify this property.

###headerless
`boolean`

Indicates that the input CSV resource does not contain header row. Optional property, valid values are `true` or `false`, and the default is `false`.

##Batch Configuration Properties for `csvItemWriter` Only
In addition to the common properties listed above, `csvItemWriter` also supports the following batch properties:

###header
`java.lang.String[]`

Specifies the CSV header row to write out.

###writeComments

Specifies the complete comment line that can be recognized by any tools or programs intended to read the current CSV output. The comments should already include the required comment-defining characters or regular expressions. The value of this property will be written out as a comment line verbatim as the first line.

###writeMode
Instructs `csvItemWriter`, when the target CSV resource already exists, whether to append to, or overwrite the existing resource, or fail. Valid values are: 
* `append` (default)
* `overwrite`
* `failIfExists`


