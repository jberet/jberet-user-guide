# Develop Batch Artifacts in Script Languages
Whether it's for data ETL or quick testing, script language offers a valuable alternative to Java in developing batch applications. JBeret supports writing `Batchlet`, `ItemReader`, `ItemProcessor` and `ItemWriter` in popular script languages. A job xml may reference an external script resource, or directly include script as CDATA or PCDATA.

JBeret relies on JSR-223 Scripting for the Java&trade; Platform (available in Java SE) standard API to run script batch artifacts. Each script language used by a batch application requires its JSR-223-compliant script engine as runtime dependency. The following table lists some common script languages and script engines:

| Script Language | Script Engine | Obtain from | Impl javax.script.Invocable?| Suitable for |
| -- | -- | -- | -- | -- |
| JavaScript | Mozilla Rhino | included in Oracle JDK 5, 6 & 7 | <span style="color:green">Yes</span> | reader, processor, writer & batchlet |
| JavaScript | Oracle Nashorn | included in Oracle JDK 8 | <span style="color:green">Yes</span> | reader, processor, writer & batchlet |
| Groovy | org.codehaus.groovy:groovy-jsr223 | Maven Central | <span style="color:green">Yes</span> | reader, processor, writer & batchlet |
| Jython / Python | org.python:jython | Maven Central | <span style="color:green">Yes</span> | reader, processor, writer & batchlet |
| JRuby / Ruby | org.jruby:jruby | Maven Central | <span style="color:green">Yes</span> | reader, processor, writer & batchlet |
| Scala | org.scala-lang:scala-compiler | Maven Central | <span style="color:red">No</span> | processor & batchlet |
| PHP | com.caucho:resin-quercus | caucho-repository http://caucho.com/m2/ | <span style="color:red">No</span> | processor & batchlet |
| R (Renjin) | org.renjin:renjin-script-engine | http://nexus.bedatadriven.com/content/groups/public | <span style="color:red">Yes</span> | processor & batchlet |

The following XML snippet includes all the above script engine dependencies. A batch application should only include what is really needed at runtime.

```xml
<dependencies>
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-jsr223</artifactId>
    </dependency>
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.jruby</groupId>
        <artifactId>jruby</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.python</groupId>
        <artifactId>jython</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-compiler</artifactId>
    </dependency>

    <dependency>
        <groupId>com.caucho</groupId>
        <artifactId>resin-quercus</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.renjin</groupId>
        <artifactId>renjin-script-engine</artifactId>
    </dependency>
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
    </dependency>
</dependencies>

<repositories>
    <repository>
        <id>caucho-repository</id>
        <url>http://caucho.com/m2/</url>
    </repository>
    <repository>
        <id>bedatadriven</id>
        <name>bedatadriven public repo</name>
        <url>http://nexus.bedatadriven.com/content/groups/public/</url>
    </repository>
</repositories>
```

##Write `Batchlet`, `ItemReader`, `ItemProcessor` or `ItemWriter` in Scripts
Batch artifacts written in script languages must follow the following rules:
* Rules for `ItemReader` script:

    * The script engine must implement `javax.script.Invocable` so JBeret can invoke various methods defined in `ItemReader` interface.
    * An `ItemReader` script must implement `readItem` function, or another function mapped to `readItem` method. Other methods from `ItemReader` interface (`open`, `close` & `checkpointInfo`) may be optionally implemented.

* Rules for `ItemProcessor` script:

    * If the script engine implements `javax.script.Invocable`, and the `ItemProcessor` script implements `processItem` function, or another function mapped to `processItem` method, that function will be invoked.
    * Otherwise, the script content is evaluated to fulfill `processItem` method.

* Rules for `ItemWriter` script:
    * The script engine must implement `javax.script.Invocable` so JBeret can invoke various methods defined in `ItemWriter` interface.
    * An `ItemWriter` script must implement `writeItems` function, or another function mapped to `writeItems` method. Other methods from `ItemWriter` interface (`open`, `close` & `checkpointInfo`) may be optionally implemented.
    
* Rules for `Batchlet` script:
    * `process` method requirement:    
        * If the script engine implements `javax.script.Invocable`, and the `Batchlet` script implements `process` function, or another function mapped to `process` method, that function will be invoked to fulfill `process` method.
        * Otherwise, the script content is evaluated to fulfill `process` method.
    * `stop` method requirement:
        *  If the script engine implements `javax.script.Invocable`, and the `Batchlet` script implements `stop` function, or another function mapped to `stop` method, that function will be invoked to fulfill `stop` method.
        * Otherwise, nothing is done.

###Method-to-function Mapping
By default, script function names are the same as method names in batch API interfaces. In certain cases, custom names may be needed to avoid naming conflict, or to follow different naming convention. This can be achieved with `methodMapping` property under `<reader>`, `<processor>`, `<writer>`, or `<batchlet>` element in job xml. 

The property value is a comma-separated list of key-value pairs, with the key as method name in batch API interfaces, and value as the function name in script.

```xml
<property name="methodMapping" value="open=openBatchReader, close=closeBatchReader"/>
```

Note that batch API method names like `open` or `close` may be reserved words or built-in functioins in certain script languages. If so, they must be mapped to different function names.

###Access Batch Objects from Script
JBeret exposes the following batch objects to the script in the scope of `javax.script.ScriptContext#ENGINE_SCOPE`, so the script application can access information and interact with batch runtime.

| Name | Description | Type | Example (syntax varies) |
| -- | -- | -- | -- |
| `jobContext` | `JobContext` of the current job execution | `javax.batch.runtime.context.JobContext` | `jobContext.getJobName()`, `jobContext.setExitStatus('xxx')` |
| `stepContext` | `StepContext` of the current step execution | `javax.batch.runtime.context.StepContext` | `stepContext.getStepName()` |
| `batchProperties` | properties specified in job xml under the current batch artifact | `java.util.Properties` | `batchProperties.get('testName')` |


##Configure Script in Job XML
Script may be configured in job xml either as a direct sub-element, or as an external reference. To achieve that, a `<script>` element is added to the standard schema, as a sub-element of `<batchlet>`, `<reader>`, `<processor>`, or `<writer>`. `<script>` element appears after any `<properties>` element for the same artifact definition.

`<script>` has 2 attributes:

| Attribute Name | Description | Required? | Examples |
| -- | -- | -- | -- |
| type | Identify the script language being used. Its value should be recognizable by the underlying script engine, either as a MIME-type, or an engine name. Some typical values are: javascript, groovy, jython, php, jruby, scala. | required for inline script, and optional for external script if its file extension can already identify the script language | type = "javascript" |
| src | resource path, file path or URL to the external script | required for external script, and must not be used for inline script | src = "javascript/item-reader.js" |


If `<script>` element is present, `ref` attribute of the same `batchlet` , `<reader>`, `<processor>`, or `<writer>`element must not be specified, because the script already defines the artifact and there is no need for artifact `ref` name.

###Inline Script in Job XML

```xml
<job id="batchletJavascriptInline" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="batchletJavascriptInline.step1">
        <batchlet>
            <properties>
                <property name="testName" value="#{jobParameters['testName']}"/>
            </properties>
            <script type="javascript">
            <![CDATA[
                function stop() {
                    print('In stop function\n');
                }

                //access built-in variables: jobContext, stepContext and batchProperties,
                //set job exit status to the value of testName property, and
                //return the value of testName property as step exit status,
                //
                function process() {
                    print('jobName: ' + jobContext.getJobName() + '\n');
                    print('stepName: ' + stepContext.getStepName() + '\n');
                    var testName = batchProperties.get('testName');
                    jobContext.setExitStatus(testName);
                    return testName;
                }
            ]]>
            </script>
        </batchlet>
    </step>
</job>
```

The script content may be CDATA or element text, and CDATA should be the preferred method to avoid issues with special characters.

Note that in certian script languages (e.g., Python), indentation is significant. So any inline script content should be indented from the left-most column, regardless of XML indentation.

Inline script is convenient for short, or even no-op batch artifact, for example,

```xml
<writer>
    <script type="javascript">
        function writeItems(items) {}
    </script>
</writer>
```

###External Script Resource Referenced by Job XML
To reference an external script file or resource, simply specify `src` attribute, and optionally `type` attribute.

```xml
<job id="batchletGroovySrc" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="batchletGroovySrc.step1">
        <batchlet>
            <properties>
                <property name="testName" value="#{jobParameters['testName']}"/>
            </properties>
            <script src="groovy/simple-batchlet.groovy"/>
        </batchlet>
    </step>
</job>

```

##Examples
###External Jython Reader, Inline Processor and Writer:

```xml
<job id="chunkPython" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="chunkPython.step1">
        <chunk item-count="3">
            <reader>
                <properties>
                    <property name="resource" value="#{systemProperties['java.io.tmpdir']}/numbers.csv"/>

                    <!-- open and close are built-in functions in Python, so need to map batch API's open & close
                    methods to some other names to avoid overriding python built-in functions.
                    The following methodMapping property maps batch API's open method to openBatch function in the
                    src script.
                    -->
                    <property name="methodMapping" value="open=openBatchReader, close=closeBatchReader"/>
                </properties>
                <script type="python" src="python/item-reader.py"/>
            </reader>

            <!-- Indentation is significant in Python, so left-justify all embedded python script -->
            <processor>
                <script type="python">
<![CDATA[
#access built-in variables: jobContext, stepContext and batchProperties,
#set job exit status to the value of testName property.
#
def processItem(item):
    print('In processItem(), jobName: ' + jobContext.getJobName() + ', stepName: ' + stepContext.getStepName())
    print(item)
    testName = batchProperties.get('testName')
    jobContext.setExitStatus(testName)
    return item
]]>
                </script>
            </processor>
            <writer>
                <properties>
                    <property name="methodMapping" value="open=openBatchWriter, close=closeBatchWriter"/>
                </properties>
                <script type="python">
<![CDATA[
def writeItems(items):
    print('items to write: ')
    print(items)
]]>
                </script>
            </writer>
        </chunk>
        <end on="*" exit-status="chunkPython"/>
    </step>
</job>
```
-----------------------
__item-reader.py:__

```python
rows = []
position = 0

def openBatchReader(checkpoint):
    global rows
    resource = batchProperties.get("resource")

    f = open(resource, 'rb')
    try:
        for row in f.readlines():
            columnValues = row.split(",")
            rows.append(columnValues)
    finally:
        f.close()

    if (checkpoint is None):
        position = checkpoint


def checkpointInfo():
    return position


def readItem():
    global position
    if (position >= len(rows)):
        return None

    item = rows[position]
    position += 1;
    return item;

```

###External Groovy Reader, Inline Processor and Writer:

```xml
<job id="chunkGroovy" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="chunkGroovy.step1">
        <chunk item-count="3">
            <reader>
                <properties>
                    <property name="resource" value="numbers.csv"/>
                </properties>
                <script type="groovy" src="groovy/ItemReader.groovy"/>
            </reader>
            <processor>
                <script type="groovy">
                    <![CDATA[
                    //access built-in variables: jobContext, stepContext and batchProperties,
                    //set job exit status to the value of testName property.
                    //
                    def processItem(item) {
                        println('In processItem(), jobName: ' + jobContext.getJobName() + ', stepName: '
                            + stepContext.getStepName() + ', item: ' + item + '\n');
                        testName = batchProperties.get('testName');
                        jobContext.setExitStatus(testName);
                        return item;
                    }
                ]]>
                </script>
            </processor>
            <writer>
                <script type="groovy">
                    <![CDATA[
                    def writeItems(items) {
                        println('items to write: ' + items + '\n');
                    }
                ]]>
                </script>
            </writer>
        </chunk>
        <end on="*" exit-status="chunkGroovy"/>
    </step>
</job>
```
-----------------------------
__ItemReader.groovy__

```groovy
package groovy

import groovy.transform.Field

@Field List<String[]> rows;
@Field int position = 0;

def open(checkpoint) {
    String resourcePath = batchProperties.get("resource");
    InputStream inputFile = this.class.getClassLoader().getResourceAsStream(resourcePath);

    String[] lines = inputFile.text.split('\n');
    rows = lines.collect { it.split(',') };
    inputFile.close();

    if (checkpoint != null) {
        position = checkpoint;
    }
    println("ItemReader.groovy open, rows: " + rows);
}

def checkpointInfo() {
    return position;
}

def readItem() {
    if (position >= rows.size()) {
        return null;
    }
    return rows.get(position++);
}
```

###External Ruby Reader, Inline Processor and Writer:
```xml
<job id="chunkRuby" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="chunkRuby.step1">
        <chunk item-count="3">
            <reader>
                <properties>
                    <property name="resource" value="#{systemProperties['java.io.tmpdir']}/numbers.csv"/>

                    <!-- open and close are built-in functions in Ruby, so need to map batch API's open & close
                    methods to some other names to avoid overriding ruby built-in functions.
                    The following methodMapping property maps batch API's open method to openBatch function in the
                    src script.
                    -->
                    <property name="methodMapping" value="open=openBatchReader, close=closeBatchReader"/>
                </properties>
                <script type="jruby" src="ruby/item-reader.rb"/>
            </reader>

            <processor>
                <script type="jruby">
                <![CDATA[
                #access built-in variables: jobContext, stepContext and batchProperties,
                #set job exit status to the value of testName property.
                #
                def processItem(item)
                    puts('In processItem(), jobName: ' + $jobContext.getJobName() + ', stepName: ' + $stepContext.getStepName())
                    puts(item)
                    testName = $batchProperties.get('testName')
                    $jobContext.setExitStatus(testName)
                    return item
                end
                ]]>
                </script>
            </processor>
            <writer>
                <properties>
                    <property name="methodMapping" value="open=openBatchWriter, close=closeBatchWriter"/>
                </properties>
                <script type="jruby">
                <![CDATA[
                def openBatchWriter(checkpoint)
                    puts('In writer open')
                end

                def closeBatchWriter()
                    puts('In writer close')
                end

                def writeItems(items)
                    puts('items to write: ')
                    puts(items)
                end
                ]]>
                </script>
            </writer>
        </chunk>
        <end on="*" exit-status="chunkRuby"/>
    </step>
</job>
```
--------------------
__item-reader.rb__

```ruby
require 'csv'

$rows = []
$position = 0

def openBatchReader(checkpoint)
    resource = $batchProperties.get("resource")
    puts(resource)
    $rows = CSV.read(resource)

    if checkpoint != nil
        $position = checkpoint
    end

    puts($rows)
end

def closeBatchReader()
    puts('In reader close')
end


def checkpointInfo()
    return $position
end


def readItem()
    if $position >= $rows.length
        return nil
    end

    item = $rows[$position]
    $position += 1
    return item
end
```
###Inline Scala Batchlet:

```xml
<job id="batchletScalaInline" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="batchletScalaInline.step1">
        <batchlet>
            <properties>
                <property name="testName" value="#{jobParameters['testName']}"/>
            </properties>
            <script type="scala">
                import java.util.Properties
                import javax.batch.runtime.context.{StepContext, JobContext}

                val jobContext1 = jobContext.asInstanceOf[JobContext]
                val stepContext1 = stepContext.asInstanceOf[StepContext]
                val batchProperties1 = batchProperties.asInstanceOf[Properties]

                println("jobName: " + jobContext1.getJobName())
                println("stepName: " + stepContext1.getStepName())
                val testName : String = batchProperties1.get("testName").asInstanceOf[String]
                jobContext1.setExitStatus(testName)
                return testName;

            </script>
        </batchlet>
    </step>
</job>
```

###Inline R (Renjin) Batchlet:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE job [
        <!ENTITY batchlet-properties-segment SYSTEM "batchlet-properties-segment.xml">
        ]>

<job id="batchletRInline" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="batchletRInline.step1">
        <batchlet>
            &batchlet-properties-segment;
            <script type="Renjin">
                <![CDATA[
                    stop <- function() {
                        print("In stop function\n");
                    }

                    # reads numbers.csv file and calculate statistics of all these numbers
                    #
                    process <- function() {
                        tbl <- read.table(file.path(tempdir(), "numbers.csv"), header=FALSE, sep=",")
                        numbers <- tbl[2]
                        print(summary(numbers))
                    }
                ]]>
            </script>
        </batchlet>
    </step>
</job>
```

