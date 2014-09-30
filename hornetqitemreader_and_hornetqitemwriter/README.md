# HornetQItemReader and HornetQItemWriter

jberet-support module includes `hornetQItemReader` and `hornetQItemWriter` for reading messages from and writing messages to HornetQ server. They can be used by batch applications that wish to directly work with HornetQ API instead of the standard JMS API.

`hornetQItemReader` handles the following types of HornetQ messages:
* If `beanType` batch property is set to `org.hornetq.api.core.client.ClientMessage`, the incoming message is immediately returned as is from `readItem()`.
* If the message type is `org.hornetq.api.core.client.ClientMessage#TEXT_TYPE`, the string text is returned from `readItem()`.
* Otherwise, a byte array is retrieved from the message body buffer, deserialize to an object, and returned from `readItem()`.

This reader ends when either of the following occurs:

* `receiveTimeout` (in milliseconds) has elapsed when trying to receive a message from the destination;
* the size of the incoming message body is 0;

`hornetQItemWriter` can send the following HornetQ message types:

* If the data item is of type `java.lang.String`, a `org.hornetq.api.core.client.ClientMessage#TEXT_TYPE` message is created with the text content in the data item, and sent.
* Else if the data is of type `org.hornetq.api.core.client.ClientMessage`, it is sent as is.
* Else an `org.hornetq.api.core.client.ClientMessage#OBJECT_TYPE` message is created with the data item object, and sent.

`durableMessage` property can be configured to send either durable or non-durable (default) messages.

##Dependencies for `hornetQItemReader` and `hornetQItemWriter`
```xml
<!-- HornetQ client-side dependencies -->
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
```

##Batch Configuration Properties in Job XML

`hornetQItemReader` and `hornetQItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml, and CDI field injections into `org.jberet.support.io.HornetQItemReader` and `org.jberet.support.io.HornetQItemWriter`. All properties are of type `String`, unless noted otherwise. The following is an example job xml that references `hornetQItemReader` and `hornetQItemWriter`:

```xml
<job id="HornetQReaderTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="HornetQReaderTest.step1">
        <chunk item-count="100">
            <reader ref="hornetQItemReader">
                <properties>
                    <property name="receiveTimeout" value="2000"/>
                    <property name="queueParams" value="address=example, durable=false"/>
                    <property name="serverLocatorParams" value="HA=false, ConsumerMaxRate=100"/>
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

<job id="HornetQWriterTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="HornetQWriterTest.step1">
        <chunk item-count="100">
            <reader ref="csvItemReader">
                <properties>
                    <property name="resource" value="IBM_unadjusted.txt"/>
                    <property name="headerless" value="true"/>
                    <property name="beanType" value="#{jobParameters['beanType']}"/>
                    <property name="start" value="#{jobParameters['start']}"/>
                    <property name="end" value="#{jobParameters['end']}"/>
                    <property name="nameMapping" value="#{jobParameters['nameMapping']}"/>
                    <property name="cellProcessors" value="#{jobParameters['cellProcessors']}"/>
                </properties>
            </reader>
            
            <writer ref="hornetQItemWriter">
                <properties>
                    <property name="queueParams" value="address=example"/>
                    <!-- For illustration only, default is already false -->
                    <property name="durableMessage" value="false"/>  
                    <property name="sendAcknowledgementHandler" value="org.jberet.support.io.HornetQReaderWriterTest$HornetQSendAcknowledgementHandler"/>
                    <property name="serverLocatorParams" value="HA=false, ProducerMaxRate=100"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```

###Batch Properties for Both `hornetQItemReader` and `hornetQItemWriter`
####connectorFactoryParams
`java.util.Map`

Key-value pairs to identify and configure HornetQ `org.hornetq.api.core.TransportConfiguration`, which is used to create HornetQ `ServerLocator`. Optional property and defaults to `null`. When this property is present, it will be used to create HornetQ `ServerLocator`, and the injection fields `serverLocatorInstance` and `sessionFactoryInstance` will be ignored. Valid keys and values are:

* `factory-class`, the fully-qualified class name of a HornetQ connector factory, required if this property is present;
* any param keys and values appropriate for the above-named HornetQ connector factory class

An example of this property in job xml:

```xml
<property name="connectorFactoryParams" value="factory-class=org.hornetq.core.remoting.impl.netty.NettyConnectorFactory, host=localhost, port=5445"/>
```

####serverLocatorParams
`java.util.Map`

Key-value pairs to configure HornetQ `ServerLocator`. Optional property and defaults to `null`. Valid keys are:

* HA: `true` or `false` (default), `true` if the `ServerLocator` receives topology updates from the cluster
* Properties in `ServerLocator` class that have corresponding setter method, starting with either upper or lower case character

See the current version of HornetQ `ServerLocator` javadoc for supported keys and values, e.g., [ServerLocator 2.4 Javadoc](http://docs.jboss.org/hornetq/2.4.0.Final/docs/api/hornetq-client/org/hornetq/api/core/client/ServerLocator.html)

An example of this property in job xml:

```xml
<property name="serverLocatorParams" value="HA=false, AckBatchSize=5, ProducerMaxRate=10, BlockOnAcknowledge=false, ConfirmationWindowSize=5"/>
```

####queueParams
`java.util.Map`

Key-value pairs to identify and configure the target HornetQ queue. Required property. The following keys are supported:

* `address`, required
* `durable`, optional
* `filter`, optional
* `name`, optional
* `shared`, optional
* `temporary`, optional

An example of `queueParams` property in job xml:

```xml
<property name="queueParams" value="address=example, durable=false"/>
```

####sendAcknowledgementHandler
`java.lang.Class`

The fully-qualified name of a class that implements `org.hornetq.api.core.client.SendAcknowledgementHandler`. A `SendAcknowledgementHandler` notifies a client when an message sent asynchronously has been received by the server. See current version of HornetQ documentation for details, e.g., [SendAcknowledgementHandler 2.4 Javadoc](https://docs.jboss.org/hornetq/2.4.0.Final/docs/api/hornetq-client/org/hornetq/api/core/client/SendAcknowledgementHandler.html)

An example `sendAcknowledgementHandler` property in job xml:

```xml
<property name="sendAcknowledgementHandler" value="org.jberet.support.io.HornetQReaderWriterTest$HornetQSendAcknowledgementHandler"/>
```

###CDI Field Injections for Both `hornetQItemReader` and `hornetQItemWriter`

####serverLocatorInstance
`javax.enterprise.inject.Instance<org.hornetq.api.core.client.ServerLocator>`

This field holds an optional injection of HornetQ `ServerLocator`. When `connectorFactoryParams` is not specified, and `sessionFactoryInstance` is not satisfied, this field will be queried to obtain an instance of HornetQ `ServerLocator`. The application may implement a `javax.enterprise.inject.Produces` method to satisfy this dependency injection.

####sessionFactoryInstance
`javax.enterprise.inject.Instance<org.hornetq.api.core.client.ClientSessionFactory>`

This field holds an optional injection of HornetQ `ClientSessionFactory`. If this injection is satisfied, `serverLocatorInstance` will be ignored. The application may implement a `javax.enterprise.inject.Produces` method to satisfy this dependency injection.


###Batch Properties for `hornetQItemReader` Only
In addition to the above common properties, `hornetQItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####receiveTimeout
`long`

The number of milliseconds a HornetQ `ClientConsumer` blocks until a message arrives. Optional property, and defaults to `0`, which means it blocks indefinitely.

####beanType
`java.lang.Class`

The fully-qualified class name of the data item to be returned from `readItem()` method. Optional property and defaults to `null`. If it is specified, its valid value is:

* `org.hornetq.api.core.client.ClientMessage`: an incoming HornetQ message is returned as is.

When this property is not specified, `readItem()` method returns an object whose actual type is determined by the incoming HornetQ message type.


###Batch Properties for `hornetQItemWriter` Only
In addition to the above common properties, `hornetQItemWriter` also supports the following batch properties in job xml:

####durableMessage
`boolean`

Whether the message to be produced is durable or not. Optional property and defaults to `false`. Valid values are `true` and `false`.


