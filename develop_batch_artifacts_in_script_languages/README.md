# Develop Batch Artifacts in Script Languages
Whether it's for data ETL or quick testing, script language offers a valuable alternative to Java in developing batch applications. JBeret supports writing `Batchlet`, `ItemReader`, `ItemProcessor` and `ItemWriter` in popular script languages. A job xml may reference an external script resource, or directly include script as CDATA or PCDATA.

JBeret relies on JSR-223 Scripting for the Java Platform (available in Java SE) standard API to run script batch artifacts. Each script language used by a batch application requires its JSR-223-compliant script engine as runtime dependency. The following table lists some common script languages and script engines:

| Script Language | Script Engine | Obtain from | Implements javax.script.Invocable?|
| -- | -- | -- | -- |
| JavaScript | Mozilla Rhino | included in Oracle JDK 5, 6 & 7 | Yes |
| JavaScript | Oracle Nashorn | included in Oracle JDK 8 | Yes |
| Groovy | org.codehaus.groovy:groovy-jsr223 | Maven Central | Yes |
| Jython / Python | org.python:jython | Maven Central | Yes |
| JRuby / Ruby | org.jruby:jruby | Maven Central | Yes |
| Scala | org.scala-lang:scala-compiler | Maven Central | No |
| PHP | com.caucho:resin-quercus | caucho-repository http://caucho.com/m2/ | No |

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
</dependencies>

<repositories>
    <repository>
        <id>caucho-repository</id>
        <url>http://caucho.com/m2/</url>
    </repository>
</repositories>
```
##Develop `Batchlet` in Script Languages
JBeret supports writing `Batchlet` in any of the above 5 script languages: 
* JavaScript
* Groovy
* Jython / Python
* JRuby / Ruby
* Scala
* PHP

###Configure Script in Job XML
Script may be configured in job xml either as a direct sub-element, or as an external reference. To achieve that, a `<script>` element is added to the standard schema, as a sub-element of `<batchlet>`, `<reader>`, `<processor>`, or `<writer>`. `<script>` element appears after any `<properties>` element for the same batchlet artifact definition.

`<script>` has 2 attributes:

| Attribute Name | Description | Required? |
| -- | -- | -- | -- |
| type | identify the script language being used | required for inline script, and optional for external script |
| src | resource path or URL to the external script | required for external script, and must not be used for inline script |


If `<script>` element is present, `ref` attribute of the same `batchlet` , `<reader>`, `<processor>`, or `<writer>`element must not be specified, because the script already defines the artifact and there is no need for artifact `ref` name.

####Inline Script in Job XML

When using inline script, the `type` attribute of `<script>` element must be specified to identify the script language being used. Its value should be recognizable by the underlying script engine, either as a MIME-type, or an engine name. Some typical values are:

* javascript
* groovy
* jython
* php
* jruby
* scala
 

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

####External Script Resource Referenced by Job XML

