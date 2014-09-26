# Set up JBeret

## Maven Repositories
All JBeret artifacts are available in eitehr [Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cjberet) or [jboss.org repository](https://repository.jboss.org/nexus/index.html). To use jboss.org repository, add the following to pom.xml:

```xml
<repositories>
        <repository>
            <id>jboss-public-repository-group</id>
            <name>JBoss Public Repository Group</name>
            <url>http://repository.jboss.org/nexus/content/groups/public/</url>
        </repository>
    </repositories>
```

## JBeret Artifacts
JBeret project produces the following major artifacts:
### jberet-core
contains all classes of JBeret batch runtime. This library is required to run batch applications in both Java SE and Java EE environment.
### jberet-se
contains implementation classes of JBeret batch runtime specifically for Java SE environment.
### jberet-distribution
a zip file containing the following content, and provides a good start to write and run batch applications with JBeret in Java SE.

* all libraries required to run batch applications in Java SE environment
* default configuration file, `jberet.properties` for configuring JBeret batch runtime in Java SE environment
* jberet-support library and its common transitive dependencies

### jberet-support
contains built-in batch artifacts (e.g., `ItemReader`, `ItemWriter`, `Batchlet` classes) for handling common data types. This library can be optionally referenced by batch applications to simplify development. The application should make sure to satisfy appropriate transitive dependencies originating from jberet-support, depending on its usage. See [JBeret javadocs](http://docs.jboss.org/jberet/) for details.

##Batch application dependencies
###Minimal application dependencies:
```xml
<dependency>
    <groupId>org.jboss.spec.javax.batch</groupId>
    <artifactId>jboss-batch-api_1.0_spec</artifactId>
</dependency>
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
</dependency>
<dependency>
    <groupId>javax.enterprise</groupId>
    <artifactId>cdi-api</artifactId>
</dependency>
<dependency>
    <groupId>org.jboss.spec.javax.transaction</groupId>
    <artifactId>jboss-transaction-api_1.2_spec</artifactId>
</dependency>
<dependency>
    <groupId>org.jberet</groupId>
    <artifactId>jberet-core</artifactId>
</dependency>
<dependency>
    <groupId>org.jboss.marshalling</groupId>
    <artifactId>jboss-marshalling</artifactId>
</dependency>
<dependency>
    <groupId>org.jboss.logging</groupId>
    <artifactId>jboss-logging</artifactId>
</dependency>
<dependency>
    <groupId>org.jboss.weld</groupId>
    <artifactId>weld-core</artifactId>
</dependency>
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
</dependency>
```
A note on webapp or Java EE application packaging: Java EE API jars (batch-api, cdi-api, javax.inject, transaction-api)
are already available in the appserver, and should not be included in WAR, JAR, or EAR files. Their maven dependency
scope should be set to `provided`.

###Additional dependencies for Java SE batch applications
h2 can be omitted when using in-memory batch job repository:
```xml
<dependency>
    <groupId>org.jberet</groupId>
    <artifactId>jberet-se</artifactId>
</dependency>
<dependency>
    <groupId>org.jboss.weld.se</groupId>
    <artifactId>weld-se</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```
###Optional application dependencies:
```xml
<!-- any JDBC driver jars  when using jdbc batch job repository -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>

<!-- For Weld 2.2.2.Final or later, Jandex is required -->
<dependency>
    <groupId>org.jboss</groupId>
    <artifactId>jandex</artifactId>
</dependency>

<!-- You can choose to replace Java built-in StAX provider with
     aalto-xml or woodstox (woodstox dependencies not shown here)
-->
<dependency>
    <groupId>com.fasterxml</groupId>
    <artifactId>aalto-xml</artifactId>
</dependency>
<dependency>
    <groupId>org.codehaus.woodstox</groupId>
    <artifactId>stax2-api</artifactId>
</dependency>

<!-- jberet-support includes common reusable batch ItemReader,
ItemWriter or Batchlet classes for common data types -->
<dependency>
    <groupId>org.jberet</groupId>
    <artifactId>jberet-support</artifactId>
</dependency>
```
