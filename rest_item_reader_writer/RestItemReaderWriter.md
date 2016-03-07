# RestItemReader and RestItemWriter

`jberet-support` module includes `restItemReader` and `restItemWriter` to support reading and writing batch data from and to a REST resource destination. `restItemReader` reads and binds data to instances of custom POJO bean provided by the batch application.

`restItemReader` and `restItemWriter` leverages the standard JAX-RS API to interact with REST resources. The following dependencies are required:

```xml
<dependency>
  <groupId>org.jboss.spec.javax.ws.rs</groupId>
  <artifactId>jboss-jaxrs-api_2.0_spec</artifactId>
  <scope>provided</scope>
</dependency>
```

Depending on your JAX-RS implementation, for instance, Jersey or RESTEasy, you may need additional dependencies. For example, when using JBoss RESTEasy, you will typically need the following additional dependencies:

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-core</artifactId>
  <!--<scope>runtime</scope>-->
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <!--<scope>runtime</scope>-->
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-annotations</artifactId>
  <!--<scope>runtime</scope>-->
</dependency>
<dependency>
  <groupId>com.fasterxml.jackson.module</groupId>
  <artifactId>jackson-module-jaxb-annotations</artifactId>
  <!--<scope>runtime</scope>-->
</dependency>
```

If your batch application is a web application or Java EE application, the target application server, e.g., WildFly or JBoss EAP, already provides all Java EE API classes. So there is no need to package JAX-RS API jar in your application archive, and its dependency scope can be `provided`. When deployed to WildFly or JBoss EAP 7, your application can declare the above jackson-related implementation dependencies by referencing corresponding JBoss modules hosted in the application server. For example, this can be done by adding `WEB-INF/jboss-deployment-structure` file:

```xml
<jboss-deployment-structure>
  <deployment>
    <dependencies>
        <module name="com.fasterxml.jackson.core.jackson-core"/>
        <module name="com.fasterxml.jackson.core.jackson-databind"/>
        <module name="com.fasterxml.jackson.core.jackson-annotations"/>
        <module name="com.fasterxml.jackson.jaxrs.jackson-jaxrs-json-provider"/>
    </dependencies>
  </deployment>
</jboss-deployment-structure>
```

## Batch Configuration Properties in Job XML

`restItemReader` and `restItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type String, unless noted otherwise. The following is an example job xml that references `restItemReader` and `restItemWriter`:

```xml
<chunk>
 <reader ref="restItemReader">
   <properties>
     <property name="restUrl" value="http://localhost:8080/appName/rest-api/movies"/>
     <!-- starts from item 3 for the initial reading, skipping the first 3 elements (0, 1, 2) -->
     <!-- if offset not set, will start reading from the beginning -->
     <property name="offset" value="3"/>

     <!-- configure each REST call to return a maximum 20 items -->
     <property name="limit" value="20"/>

     <!-- type of each element in REST response entity -->
     <property name="beanType" value="org.jberet.samples.wildfly.common.Movie"/>
   </properties>
 </reader>
 ...
<chunk>
```

```xml
<chunk>
  ...
  <writer ref="restItemWriter">
    <properties>
      <!-- required property -->
      <property name="restUrl" 
       value="http://localhost:8080/appName/rest-api/movies?param1=value1"/>
       
      <!-- optional properties -->
      <property name="httpMethod" value="POST"/>
      <property name="mediaType" value="application/json"/>
    </properties>
  </writer>
</chunk>
```

### Batch Properties for Both `RestItemReader` and `RestItemWriter`

#### restUrl
`java.net.URI`

The base URI for the REST call. It usually points to a collection resource URI. For `RestItemReader`, data may be retrieved via HTTP GET or less commonly DELETE method. The URI may include additional query parameters other than `offset` (starting position to read) and `limit` (maximum number of items to return in each response). Query parameter `offset` and `limit` are specified by their own batch properties.

For example, http://localhost:8080/restReader/api/movies

For `RestItemWriter`, data may be submitted via HTTP POST or PUT method. The URI may include additional query parameters.

For example, http://localhost:8080/restReader/api/movies?param1=value1

This is a required batch property.

#### httpMethod

HTTP method to use in the REST call to read or write data. Its value should corresponds to the media types accepted by the target REST resource.

For `RestItemReader`, valid values are GET and less commonly DELETE. If not specified, this property defaults to GET.

For `RestItemWriter`, valid values are POST and PUT. If not specified, this property defaults to POST.

### Batch Properties for `RestItemReader` Only

In addition to the above common properties, `RestItemReader` also supports the following batch properties in job xml:

#### offsetKey

Configures the key of the query parameter that specifies the starting position to read in the target REST resource. For example, some REST resource may require `start` instead of `offset` query parameter for the same purpose.

This batch property is optional. If not set, the default key `offset` is used.

#### offset

The value of the offset property, which specifies the starting point for reading. If not specified, it defaults to `0`.

#### limitKey

Configures the key of the query parameter that specifies the maximum number of items to return in the REST response. For example, some REST resource may require `count` instead of `limit` query parameter for the same purpose.

This batch property is optional. If not set, the default key `limit` is used.

#### limit

The value of the `limit` property, which specifies the maximum number of items to read. If not specified, it defaults to `10`.

#### beanType

The class of individual element of the response message entity. For example,
* java.lang.String
* org.jberet.samples.wildfly.common.Movie

### Batch Properties for `RestItemWriter` Only

In addition to the above common properties, `restItemWriter` also supports the following batch properties in job xml:

#### mediaType

Media type to use in the REST call to write data. Its value should be valid for the target REST resource. If not specified, this property defaults to `application/json`.