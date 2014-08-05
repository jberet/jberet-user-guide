# Batch JSL Inheritance

Batch job xml inheritance is not included in JSR 352 Batch Spec 1.0. JBeret implements JSL inheritance based on the draft [JSL Inheritance v1](https://java.net/projects/jbatch/downloads). Refer to that document for inheritance rules and restrictions.  Note that this is an experimental feature and may undergo significant changes in future releases.

## JSL Inheritance Examples
Here are some examples illustrating how to use JSL inheritance with JBeret.

### Inherit step and flow within the same job xml document
Parent elements (step, flow, etc) are marked with attribute `abstract = "true"` to exclude them from direct execution.  Child elements contains `parent` attribute, which points to the parent element.

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

Child job xml
```xml
<job id="chunk-child" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="chunk-child-step" parent="chunk-parent-step" jsl-name="chunk-parent">
    </step>
</job>
```

Parent job xml
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

