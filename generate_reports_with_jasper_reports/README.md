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

##Batch Configuration Properties
`jasperReportsBatchlet` can have the following batch properties in job xml.  All properties are of type java.lang.String, unless otherwise noted.
####resource
The resource that provides the data source for generating report. Optional property, and defaults to null. If specified, it may be a URL, a file path, or any resource path that can be loaded by the current application class loader. If this property is not specified, the application should inject appropriate `JRDataSource` into `jrDataSourceInstance`. If neither `resource` nor `jrDataSourceInstance` is specified, an `net.sf.jasperreports.engine.JREmptyDataSource` is used.

`JRDataSource` injection allows for more flexible instantiation and configuration, such as setting `locale`, `datePattern`, `numberPattern`, `timeZone`, `recordDelimiter`, `useFirstRowAsHeader`, `columnNames`, `fieldDelimiter`, etc, before making the instance available to this class.

This property has higher precedence than `jrDataSourceInstance` injection.

####recordDelimiter
If `resource` is specified, and is a csv resource, this property specifies the delimiter between records, typically new line character (CR/LF). Optional property. See `net.sf.jasperreports.engine.data.JRCsvDataSource` for details.

####useFirstRowAsHeader
If `resource` is specified, and is a csv, xls, or xlsx resource, this property specifies whether to use the first row as header. Optional property and valid values are "true" and "false". See `net.sf.jasperreports.engine.data.JRCsvDataSource` or `net.sf.jasperreports.engine.data.AbstractXlsDataSource` for details.

####fieldDelimiter
If `resource` is specified, and is a CSV resource, this property specifies the field or column delimiter. Optional property. See `net.sf.jasperreports.engine.data.JRCsvDataSource` for details.

####columnNames
`java.lang.String[]`

If `resourc`e is specified, and is a csv, xls, or xlsx resource, this property specifies an array of strings representing column names matching field names in the report template. Optional property. See `net.sf.jasperreports.engine.data.JRCsvDataSource` or `net.sf.jasperreports.engine.data.AbstractXlsDataSourcefor` details.

####datePattern
If `resource` is specified, this property specifies the date pattern string value. Optional property. See `net.sf.jasperreports.engine.data.JRAbstractTextDataSource#setDatePattern(java.lang.String)` for details.

####numberPattern
If `resource` is specified, this property specifies the number pattern string value. Optional property. See `net.sf.jasperreports.engine.data.JRAbstractTextDataSource#setNumberPattern(java.lang.String)` for details.

####timeZone
If `resource` is specified, this property specifies the time zone string value. Optional property. See `net.sf.jasperreports.engine.data.JRAbstractTextDataSource#setTimeZone(java.lang.String)` for details.

####locale
If `resource` is specified, this property specifies the locale string value. Optional property. See `net.sf.jasperreports.engine.data.JRAbstractTextDataSource#setLocale(java.lang.String)` for details.

####charset
If `resource` is specified, and is a csv resource, this property specifies the charset name for reading the csv resource. Optional property. See `net.sf.jasperreports.engine.data.JRCsvDataSource#JRCsvDataSource(java.io.File, java.lang.String)` for detail.

####template
Resource path of the compiled Jasper Reports template (*.jasper file). Required property. It may be a URL, a file path, or any resource path that can be loaded by the current application class loader.

####outputType
The format of report output. Optional property and defaults to "pdf". Valid values are:

* pdf
* html
* jrprint

If other formats are desired, the application should inject into `exporterInstance` with a properly configured instance of `net.sf.jasperreports.export.Exporter`.

####outputFile
The file path of the generated report. Optional property and defaults to null. When this property is not specified, the application should inject a `java.io.OutputStream` into `outputStreamInstance`. For instance, in order to stream the report to servlet response, a `javax.servlet.ServletOutputStream` can be injected into `outputStreamInstance`.

This property has higher precedence than `outputStreamInstance` injection.

####reportParameters
`java.util.Map`

Report parameters for generating the report. Optional property and defaults to null. This property can be used to specify string-based key-value pairs as report parameters. For more complex report parameters with object types, use injection into `reportParametersInstance`.

This property has higher precedence than `reportParametersInstance` injection.

###Dependency Injection Fields
In addition to the above batch properties configured in job.xml, some `jasperReportsBatchlet` configurations can be alternatively specified with CDI injection fields:

####outputStreamInstance
`javax.enterprise.inject.Instance<java.io.OutputStream>`

Optional injection of report output stream, which allows for more control over the output stream creation, sharing, and configuration. The injected `OutputStream` is closed at the end of `process()` method.

####jrDataSourceInstance
`javax.enterprise.inject.Instance<net.sf.jasperreports.engine.JRDataSource>`

Optional injection of Jasper Reports `net.sf.jasperreports.engine.JRDataSource`, which allows for more control over the `JRDataSource` creation and configuration.

####reportParametersInstance
`javax.enterprise.inject.Instance<java.util.Map<java.lang.String,java.lang.Object>>`

Optional injection of Jasper Reports report parameters, which allows for more complex, non-string values.

####exporterInstance
`javax.enterprise.inject.Instance<net.sf.jasperreports.export.Exporter>`

Optional injection of an implementation of Jasper Reports `net.sf.jasperreports.export.Exporter`. The injected instance should have been properly configured, including appropriate `net.sf.jasperreports.export.ExporterOutput`. `net.sf.jasperreports.export.ExporterInput` will be set to a `net.sf.jasperreports.engine.JasperPrint` by this class.

Some built-in implementations of `net.sf.jasperreports.export.ExporterOutput`:

* `net.sf.jasperreports.engine.export.JRPdfExporter`
* `net.sf.jasperreports.engine.export.HtmlExporter`
* `net.sf.jasperreports.engine.export.ooxml.JRDocxExporter`
* `net.sf.jasperreports.engine.export.ooxml.JRPptxExporter`
* `net.sf.jasperreports.engine.export.JRXlsExporter`
* `net.sf.jasperreports.engine.export.ooxml.JRXlsxExporter`
* `net.sf.jasperreports.engine.export.JRTextExporter`
* `net.sf.jasperreports.engine.export.JRRtfExporter`
* `net.sf.jasperreports.engine.export.JRXmlExporter`
* `net.sf.jasperreports.engine.export.JRCsvExporter`
* `net.sf.jasperreports.engine.export.JsonExporter`
* `net.sf.jasperreports.engine.export.oasis.JROdsExporter`
* `net.sf.jasperreports.engine.export.oasis.JROdtExporter`






