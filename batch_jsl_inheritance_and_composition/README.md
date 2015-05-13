# Batch JSL Inheritance and Composition

In batch applications, it is a common practice to factor out common parts of job definition. It can be achieved with:

* inheritance: inherit common parts from parent job xml;
* composition: reference common parts as external entities.

Batch job xml inheritance is not included in JSR 352 Batch Spec 1.0. JBeret implements JSL inheritance based on the draft [JSL Inheritance v1](https://java.net/projects/jbatch/downloads). Refer to that document for inheritance rules and restrictions.  Note that this is an experimental feature and may undergo significant changes in future releases.

## JSL Inheritance Examples
Here are some examples illustrating how to use JSL inheritance with JBeret.

### Inherit step and flow within the same job xml document
Parent elements (step, flow, etc) are marked with attribute `abstract = "true"` to exclude them from direct execution.  Child elements contains `parent` attribute, which points to the parent element.

<p align="center"><b>inheritance.xml</b></p>

```xml
<job id="inheritance" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <!-- abstract step and flow -->
    <step id="step0" abstract="true">
        <batchlet ref="batchlet0"/>
    </step>

    <flow id="flow0" abstract="true">
        <step id="flow0.step1" parent="step0"/>
    </flow>

    <!-- concrete step and flow -->
    <step id="step1" parent="step0" next="flow1"/>

    <flow id="flow1" parent="flow0"/>
</job>

```

### Inherit step from a different job xml document
Child elements (step, job, etc) contain a `jsl-name` attribute, which specifies the job xml name (without `.xml` extension) containing the parent element.

<p align="center"><b>chunk-child.xml</b></p>

```xml
<job id="chunk-child" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="chunk-child-step" parent="chunk-parent-step" jsl-name="chunk-parent">
    </step>
</job>
```

<p align="center"><b>chunk-parent.xml</b></p>

```xml
<job id="chunk-parent" >
    <step id="chunk-parent-step" abstract="true">
        <chunk checkpoint-policy="item" skip-limit="5" retry-limit="5">
            <reader ref="R1"></reader>
            <processor ref="P1"></processor>
            <writer ref="W1"></writer>

            <checkpoint-algorithm ref="parent">
                <properties>
                    <property name="parent" value="parent"></property>
                </properties>
            </checkpoint-algorithm>
            <skippable-exception-classes>
                <include class="java.lang.Exception"></include>
                <exclude class="java.io.IOException"></exclude>
            </skippable-exception-classes>
            <retryable-exception-classes>
                <include class="java.lang.Exception"></include>
                <exclude class="java.io.IOException"></exclude>
            </retryable-exception-classes>
            <no-rollback-exception-classes>
                <include class="java.lang.Exception"></include>
                <exclude class="java.io.IOException"></exclude>
            </no-rollback-exception-classes>
        </chunk>
    </step>
</job>

```
## Resolve Custom XML External Entities
Compared to inheritance, custom XML external entity offers a more direct, low-lelvel means of JSL composition. For example,

<p align="center"><b>job-with-xml-entities.xml</b></p>

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE job [
        <!ENTITY job-segment SYSTEM "job-segment.xml">
        ]>

<job id="job-with-xml-entities" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">

    &job-segment;

    <step id="job-with-xml-entities.step1">
        <batchlet ref="batchlet1"/>
    </step>
</job>
```

<p align="center"><b>job-segment.xml</b></p>

```xml
<?xml version="1.0" encoding="UTF-8"?>

<properties>
    <property name="common.property.key" value="common.property.value"/>
</properties>

<listeners>
    <listener ref="EL1"></listener>
    <listener ref="EL2"></listener>
</listeners>
```

The target file of the entity (`job-segment.xml` in the above example) should be accessible and loadable by JBeret batch runtime, and typically reside in the same location as the referencing job xml (`job-with-xml-entities.xml` in the above example).
