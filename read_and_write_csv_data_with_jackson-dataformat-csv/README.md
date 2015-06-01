# Read and Write CSV Data with jackson-dataformat-csv

In addition to item reader and writer based on super-csv library, jberet-support also contains `jacksonCsvItemReader` and `jacksonCsvItemWriter`, which implement reading from and writing to CSV using jackson-dataformat-csv library. This offers a convenient alternative especially for those applications that already depend on jackson family of libraries.

The following dependency is required for `jacksonCsvItemReader` and `jacksonCsvItemWriter`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.module</groupId>
    <artifactId>jackson-module-jaxb-annotations</artifactId>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-csv</artifactId>
</dependency>
```

## Configure `jacksonCsvItemReader` and `jacksonCsvItemWriter` in job xml

The following is a short example job xml defining `jacksonCsvItemReader` and `jacksonCsvItemWriter` with selected batch properties:

```xml
<job id="MovieTestWithJacksonCsv" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="MovieTestWithJacksonCsv.step1">
        <chunk>
            <reader ref="jacksonCsvItemReader">
                <properties>
                    <property name="resource" value="movies-2012.csv"/>
                    <property name="start" value="#{jobParameters['start']}"/>
                    <property name="end" value="#{jobParameters['end']}"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>
                    <property name="columns" value="#{jobParameters['columns']}"/>
                    <property name="useHeader" value="#{jobParameters['useHeader']}?:false;"/>
                </properties>
            </reader>
            <writer ref="jacksonCsvItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>
                    <property name="writeMode" value="overwrite"/>
                    <property name="columns" value="#{jobParameters['columns']}"/>
                    <property name="useHeader" value="#{jobParameters['useHeader']}?:false;"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```


