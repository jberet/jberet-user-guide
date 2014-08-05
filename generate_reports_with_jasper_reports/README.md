# Generate Reports with Jasper Reports
jberet-support module includes a batchlet that generates reports with Jasper Reports.  Batch applications can reference this batchlet by name `jasperReportsBatchlet` to produce all types of reports supported by Jasper Reports.  See [org.jberet.support.io.JasperReportsBatchlet javadoc](http://docs.jboss.org/jberet/) for more information.

##Dependencies
In addition to the dependencies for batch applications (see Chapter Set up JBeret), applications need the following compile dependencies to use `jasperReportsBatchlet`

```xml
<dependency>
    <groupId>net.sf.jasperreports</groupId>
    <artifactId>jasperreports</artifactId>
</dependency>
```

##Configure `jasperReportsBatchlet` in job xml
The following is an example how to reference and configure `jasperReportsBatchlet` in job xml. Details on each batch property is explained in the next section.
```xml
<job id="jasperReportsTest"
     xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="step1">
        <batchlet ref="jasperReportsBatchlet">
            <properties>
                <property name="resource" value="movies-2012.csv"/>
                <property name="useFirstRowAsHeader" value="true"/>
                <!-- \n as record/row delimiter -->
                <property name="recordDelimiter" value="&#xA;"/>

                <property name="charset" value="UTF-8"/>
                <property name="template" value="movies.jasper"/>
                <property name="outputType" value="pdf"/>
                <property name="outputFile" value="#{systemProperties['java.io.tmpdir']}/report.pdf"/>
                <property name="reportParameters" value="#{jobParameters['reportParameters']}"/>
            </properties>
        </batchlet>
    </step>
</job>
```

##Configuration Properties
###


