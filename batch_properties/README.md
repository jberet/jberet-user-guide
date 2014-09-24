# Batch Properties
## Batch Property Scopes
Batch properties are defined in job xml to configure the job. There are 3 groups of batch properties, according to their scopes:
### Job-level batch properties:

To define job-level batch properties in job xml:

```xml
<job version="1.0" id="job1" xmlns="http://xmlns.jcp.org/xml/ns/javaee">
    <properties>
        <property name="jobProp1" value="value1"/>
    </properties>
    <!-- other elements omitted -->
</job>
```

Access job-level batch properties from a batch artifact class with `javax.batch.runtime.context.JobContext#getProperties`

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    JobContext jobContext;

    private Properties jobProperties;

    @Override
    public String process() throws Exception {
        jobProperties = jobContext.getProperties();
        String value1 = jobProperties.getProperty("jobProp1");
        // ...
        return "Processed";
    }
}
```

Note that the properties obtained from `JobContext#getProperties` contains job-level batch properties as defined in job xml, and should not be used by application to store application data.  For this purpose, `JobContext#setTransientUserData` should be used instead.

### Step-level batch properties

To define step-level batch properties in job xml:
```xml
<step id="step1" start-limit="5" allow-start-if-complete="true">
    <properties>
        <property name="stepProp1" value="value1"/>
    </properties>
    <!-- other elements omitted -->
</step>
```

Access step-level batch properties from a batch artifact class with `javax.batch.runtime.context.StepContext#getProperties`

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    StepContext stepContext;

    private Properties stepProperties;

    @Override
    public String process() throws Exception {
        stepProperties = stepContext.getProperties();
        String value1 = stepProperties.getProperty("stepProp1");
        // ...
        return "Processed";
    }
}
```
Note that the properties obtained from `StepContext#getProperties` contains step-level batch properties as defined in job xml, and should not be used by application to store application data.  For this purpose, `StepContext#setTransientUserData` or `StepContext#setPersistentUserData` should be used instead.

### Batch artifact properties

To define batch artifact properties in job xml:
```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="name1" value="value1"/>
    </properties>
</batchlet>
```

Batch artifact properties can be accessed from a batch artifact with injection.  The optional attribute `javax.batch.api.BatchProperty#name` tells which batch property to inject into the annotated field.  When `name` attribute is omitted, the target batch property name is the same as the field name.

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    String name1;

    @Inject
    @BatchProperty(name = "name1")
    String title;

    //...
}
```

Note that job- and step-level batch properties may not be directly injected into batch artifact classes.  However, application may inject an batch artifact property that references a job- or step-level batch property via `jobProperties` substitution.  For example,

```xml
<job id="job1" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <properties>
        <property name="jobProp1" value="value1"/>
    </properties>

    <step id="step1">
        <properties>
            <property name="stepProp1" value="value1"/>
        </properties>

        <batchlet ref="myBatchlet">
            <properties>
                <property name="prop1"
                        value="#{jobProperties['jobProp1']}" />
                <property name="prop2"
                        value="#{jobProperties['stepProp1']}"/>
            </properties>
            <!-- other elements omitted -->
        </batchlet>
    </step>
</job>
```

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    String prop1;

    @Inject
    @BatchProperty
    String prop2;

    //...
}
```

## Batch Property Injection with @BatchProperty
A batch artifact class can contain fields annotated with `@Inject` and `@BatchProperty` to inject properties defined in job xml into these fields.  The injection field can be any of the following java types:

* `java.lang.String`
* `java.lang.StringBuilder`
* `java.lang.StringBuffer`
* any primitive type, and its wrapper type:
    * `boolean, Boolean`
    * `int, Integer`
    * `double, Double`
    * `long, Long`
    * `char, Character`
    * `float, Float`
    * `short, Short`
    * `byte, Byte`
* `java.math.BigInteger`
* `java.math.BigDecimal`
* `java.net.URL`
* `java.net.URI`
* `java.io.File`
* `java.util.jar.JarFile`
* `java.util.Date`
* `java.lang.Class`
* `java.net.Inet4Address`
* `java.net.Inet6Address`
* `java.util.List, List<?>, List<String>`
* `java.util.Set, Set<?>, Set<String>`
* `java.util.Map, Map<?, ?>, Map<String, String>, Map<String, ?>`
* `java.util.logging.Logger`
* `java.util.regex.Pattern`
* `javax.management.ObjectName`

The following array types are also supported:

* `java.lang.String[]`
* any primitive type, and its wrapper type:
    * `boolean[], Boolean[]`
    * `int[], Integer[]`
    * `double[], Double[]`
    * `long[], Long[]`
    * `char[], Character[]`
    * `float[], Float[]`
    * `short[], Short[]`
    * `byte[], Byte[]`
* `java.math.BigInteger[]`
* `java.math.BigDecimal[]`
* `java.net.URL[]`
* `java.net.URI[]`
* `java.io.File[]`
* `java.util.jar.JarFile[]`
* `java.util.zip.ZipFile[]`
* `java.util.Date[]`
* `java.lang.Class[]`

The following example injects a number into the batchlet class as various suitable types:

```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="number" value="10"/>
    </properties>
</batchlet>
```
```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    int number;    // field name is the same as batch property name

    @Inject
    @BatchProperty (name = "number")
    long asLong;   // use name attribute to locate batch property

    @Inject
    @BatchProperty (name = "number")
    Double asDouble;  // inject as Double wrapper type

    @Inject
    @BatchProperty (name = "number")
    private String asString;   // inject as private String

    @Inject
    @BatchProperty (name = "number")
    BigInteger asBigInteger;   // inject as BigInteger

    @Inject
    @BatchProperty (name = "number")
    BigDecimal asBigDecimal;   // inject as BigDecimal
}
```

The following example injects a number sequence into the batchlet class as various arrays and collections:

```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="weekDays" value="1,2,3,4, 5, 6, 7"/>
    </properties>
</batchlet>
```

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    int[] weekDays;

    @Inject
    @BatchProperty (name = "weekDays")
    Integer[] asIntegers;

    @Inject
    @BatchProperty (name = "weekDays")
    String[] asStrings;

    @Inject
    @BatchProperty (name = "weekDays")
    byte[] asBytes;

    @Inject
    @BatchProperty (name = "weekDays")
    BigInteger[] asBigIntegers;

    @Inject
    @BatchProperty (name = "weekDays")
    BigDecimal[] asBigDecimals;

    @Inject
    @BatchProperty (name = "weekDays")
    List asList

    @Inject
    @BatchProperty (name = "weekDays")
    List<String> asListString;

    @Inject
    @BatchProperty (name = "weekDays")
    Set asSet;

    @Inject
    @BatchProperty (name = "weekDays")
    Set<String> asSetString;
}
```

The following example injects key-value pairs batch property into the batchlet class as a Map:
```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="pairs"
                  value="id=1, name=Jon, age=30"/>
    </properties>
</batchlet>
```

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    protected Map<String, String> pairs;

    @Inject
    @BatchProperty (name = "pairs")
    private Map asMap;
}
```

The following example injects class batch property into the batchlet class:
```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="clazz" value="org.jberet.support.io.Person"/>
    </properties>
</batchlet>
```

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    private Class clazz;
}
```

The following example injects java.util.Date batch property into the batchlet class.  Its string value is converted to java.util.Date based on the current system locale.
```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="date" value="05/09/2013 7:03 AM"/>
    </properties>
</batchlet>
```

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    private java.util.Date date;
}
```

The following example injects URL or URI batch property into the batchlet class:
```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="url" value="www.jboss.org"/>
    </properties>
</batchlet>
```

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    private URL url;

    @Inject
    @BatchProperty (name = "url")
    private URI uri;
}
```

The following example injects File or JarFile batch property into the batchlet class:
```xml
<batchlet ref="myBatchlet">
    <properties>
        <property name="file"
            value="#{systemProperties['java.home']}/lib/jce.jar"/>
    </properties>
</batchlet>
```

```java
@Named
public class MyBatchlet extends AbstractBatchlet {
    @Inject
    @BatchProperty
    private File file;

    @Inject
    @BatchProperty (name = "file")
    private JarFile asJarFile
}
```
