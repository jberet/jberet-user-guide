# JmsItemReader and JmsItemWriter

jberet-support module includes `jmsItemReader` and `jmsItemWriter` to handle JMS messages. `jmsItemReader` reads and binds data to instances of custom POJO bean provided by the batch application. 

`jmsItemReader` and `jmsItemWriter` invokes standard JMS API (1.1 or later) for reading and writing JMS messages. Any compliant JMS implementation (e.g., HornetQ) can be used as the MQ provider.  The following dependencies are required:

```xml
<dependency>
    <groupId>org.jboss.spec.javax.jms</groupId>
    <artifactId>jboss-jms-api_2.0_spec</artifactId>
    <scope>compile</scope>
</dependency>

<!-- HornetQ client-side dependencies, if HornetQ is used as the MQ provider -->
<dependency>
    <groupId>org.hornetq</groupId>
    <artifactId>hornetq-core-client</artifactId>
</dependency>
<dependency>
    <groupId>org.hornetq</groupId>
    <artifactId>hornetq-commons</artifactId>
</dependency>
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
</dependency>

<!-- HornetQ server-side dependencies, if the applications runs embedded HornetQ server -->
<dependency>
    <groupId>org.hornetq</groupId>
    <artifactId>hornetq-server</artifactId>
</dependency>
<dependency>
    <groupId>org.hornetq</groupId>
    <artifactId>hornetq-jms-server</artifactId>
</dependency>
```

##Batch Configuration Properties in Job XML

`jmsItemReader` and `jmsItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml, and CDI field injections into `org.jberet.support.io.JmsItemReader` and `org.jberet.support.io.JmsItemWriter`. All properties are of type `String`, unless noted otherwise. The following is an example job xml that references `jmsItemReader` and `jmsItemWriter`:

```xml
<job id="JmsReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="JmsReaderTest.step1">
        <chunk item-count="100">
            <reader ref="jmsItemReader">
                <properties>
                    <!--<property name="connectionFactoryLookupName" value="/cf"/>-->
                    <!--<property name="destinationLookupName" value="/queue/queue1"/>-->
                    <!-- wait for 2 seconds for any more messages -->
                    <property name="receiveTimeout" value="2000"/>
                    <property name="messageSelector" value=""/>
                    <property name="sessionMode" value="DUPS_OK_ACKNOWLEDGE"/>
                </properties>
            </reader>

            <writer ref="csvItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="beanType" value="org.jberet.support.io.StockTrade"/>
                    <property name="writeMode" value="overwrite"/>
                    <property name="header" value="Date,Time,Open,High,Low,Close,Volume"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

```xml
<job id="JmsWriterTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="JmsWriterTest.step1">
        <chunk item-count="100">
            <reader ref="csvItemReader">
                <properties>
                    <property name="resource" value="IBM_unadjusted.txt"/>
                    <property name="headerless" value="true"/>
                    <property name="beanType" value="java.util.Map"/>
                    <property name="start" value="#{jobParameters['start']}"/>
                    <property name="end" value="#{jobParameters['end']}"/>
                    <property name="nameMapping" value="Date,Time,Open,High,Low,Close,Volume"/>

                    <!-- JMS MapMessage cannot take java.util.Date as keyed value, so leave Date as string-->
                    <property name="cellProcessors" value="null; null; ParseDouble; ParseDouble; ParseDouble; ParseDouble; ParseDouble"/>
                </properties>
            </reader>
            <writer ref="jmsItemWriter">
                <properties>
                    <!--<property name="connectionFactoryLookupName" value="/cf"/>-->
                    <!--<property name="destinationLookupName" value="/queue/queue1"/>-->
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###Batch Properties for `jmsItemReader` and `jmsItemWriter`

####destinationLookupName
JNDI lookup name for the JMS `Destination`. Optional property and defaults to `null`. When specified in job xml, it has higher precedence over `destinationInstance` injection

####connectionFactoryLookupName
JNDI lookup name for the JMS `ConnectionFactory`. Optional property and defaults to `null`. When specified in job xml, it has higher precedence over `connectionFactoryInstance` injection.

####sessionMode
The string name of the `sessionMode` used to create JMS session from a JMS connection. Optional property, and defaults to `null`. When not specified, JMS API `Connection.createSession()` is invoked to create the JMS session. When this property is specified, its value must be either `AUTO_ACKNOWLEDGE` or `DUPS_OK_ACKNOWLEDGE`.

An example property in job xml:
```xml
<property name="sessionMode" value="DUPS_OK_ACKNOWLEDGE"/>
```
See JMS API `Connection.createSession(int)` for more details.


###CDI Field Injections for `jmsItemReader` and `jmsItemWriter`
####destinationInstance
`javax.enterprise.inject.Instance<javax.jms.Destination>`

This field holds an optional injection of `javax.jms.Destination`. When `destinationLookupName` property is specified in job xml, this field is ignored and `destinationLookupName` is used to look up JMS destination. The application may implement a `javax.enterprise.inject.Produces` method to satisfy this dependency injection.

####connectionFactoryInstance
`javax.enterprise.inject.Instance<javax.jms.ConnectionFactory>`

This field holds an optional injection of `javax.jms.ConnectionFactory`. When `connectionFactoryLookupName` property is specified in job xml, this field is ignored and `connectionFactoryLookupName` is used to look up JMS `ConnectionFactory`. The application may implement a `javax.enterprise.inject.Produces` method to satisfy this dependency injection.


###Batch Properties for `jmsItemReader` Only
In addition to the above common properties, `jmsItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####receiveTimeout
`long`
The number of milliseconds a JMS `MessageConsumer` blocks until a message arrives. Optional property, and defaults to `0`, which means it blocks indefinitely.

####messageSelector
Only messages with properties matching the message selector expression are delivered. A value of `null` or an empty string indicates that there is no message selector for the message consumer. See JMS API `Session.createConsumer(javax.jms.Destination, java.lang.String)`

####beanType
`java.lang.Class`

The fully-qualified class name of the data item to be returned from `readItem()` method. Optional property and defaults to `null`. If it is specified, its valid value is:

* `javax.jms.Message`: an incoming JMS message is returned as is.

When this property is not specified, `readItem()` method returns an object whose actual type is determined by the incoming JMS message type.

###Batch Properties for `jmsItemWriter` Only
None


