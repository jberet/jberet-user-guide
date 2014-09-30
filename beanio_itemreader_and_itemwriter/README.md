# BeanIO ItemReader and ItemWriter

jberet-support module includes `beanIOItemReader` and `beanIOItemWriter` that employs [BeanIO](http://beanio.org/) to read and write data in various formats that are supported by BeanIO, e.g., fixed length file, CSV file, XML, etc. They also support restart, ranged reading, custom error handler, and dynamic BeanIO mapping properties. 

The following BeanIO dependencies are required for `beanIOItemReader` and `beanIOItemWriter`:

```xml
<dependency>
    <groupId>org.beanio</groupId>
    <artifactId>beanio</artifactId>
    <version>${version.org.beanio}</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.ow2.asm</groupId>
    <artifactId>asm</artifactId>
    <version>${version.org.ow2.asm}</version>
</dependency>
```

##Batch Configuration Properties in Job XML

`beanIOItemReader` and `beanIOItemWriter` are configured through `<reader>` or `<writer>` batch properties in job xml. All properties are of type `String`, unless noted otherwise. The following is an example job xml that references `beanIOItemReader` and `beanIOItemWriter`:

```xml
<job id="BeanIOReaderWriterTest" xmlns="http://xmlns.jcp.org/xml/ns/javaee" version="1.0">
    <step id="BeanIOReaderWriterTest.step1">
        <chunk item-count="1000000">
            <reader ref="beanIOItemReader">
                <properties>
                    <property name="resource" value="#{jobParameters['resource']}"/>
                    <property name="start" value="1"/>
                    <property name="end" value="1000"/>
                    <property name="streamName" value="persons"/>
                    <property name="streamMapping" value="person-beanio-mapping.xml"/>
                    <property name="mappingProperties" value="zipCodeFieldName=zipCode, zipCodeFieldType=string"/>
                    <property name="errorHandler" value="org.beanio.BeanReaderErrorHandlerSupport"/>
                    <!--<property name="locale" value="en_US"/>-->
                    <property name="charset" value="UTF-8"/>
                </properties>
            </reader>
            
            <writer ref="beanIOItemWriter">
                <properties>
                    <property name="resource" value="#{jobParameters['writeResource']}"/>
                    <property name="streamName" value="persons"/>
                    <property name="streamMapping" value="person-beanio-mapping.xml"/>
                    <property name="mappingProperties" value="zipCodeFieldName=zipCode, zipCodeFieldType=string"/>
                    <property name="charset" value="UTF-8"/>
                </properties>
            </writer>
        </chunk>
    </step>
</job>
```
The following is a sample fixed length flat file (downloaded from http://star.cde.ca.gov/star2013/research_fixfileformat.aspx), and its BeanIO mapping file, referenced in job xml as `streamMapping` property:

```
000000000000000000201304State of California
010000000000000000201305Alameda
011001700000000000201306Alameda                                           Alameda County Office of Education
011001701098350728201309Alameda                                           FAME Public Charter                               FAME Public Charter                               94560
011001701126070811201309Alameda                                           Envision Academy for Arts & T                     Envision Academy for Arts & T                     94612
011001701184891049201309Alameda                                           Aspire California College Prep                    Aspire California College Prep                    94606
```
```xml
<beanio xmlns="http://www.beanio.org/2012/03"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.beanio.org/2012/03 http://www.beanio.org/2012/03/mapping.xsd">
    <stream name="star-entities" format="fixedlength">
        <record name="star-entity" class="org.jberet.support.io.CaliforniaStarEntity">
            <field name="countyCode" length="2"/>
            <field name="districtCode" length="5"/>
            <field name="schoolCode" length="7"/>
            <field name="charterNumber" length="4"/>
            <field name="testYear" length="4"/>
            <field name="typeId" length="2"/>
            <field name="countyName" length="50" />
            <field name="districtName" length="50"/>
            <field name="schoolName" length="50"/>
            <field name="${zipCodeFieldName}" type="${zipCodeFieldType}" length="5"/>
        </record>
    </stream>
</beanio>
```

Another sample BeanIO mapping file for person data file with "|" as the delimiter:
```
Number|Gender|Title|GivenName|MiddleInitial|Surname|StreetAddress|City|State|ZipCode|Country|CountryFull|EmailAddress|Username|Password|TelephoneNumber|MothersMaiden|Birthday|CCType|CCNumber|CVV2|CCExpires|NationalID|UPS|Color|Occupation|Company|Vehicle|Domain|BloodType|Pounds|Kilograms|FeetInches|Centimeters|GUID|Latitude|Longitude
1|female|Mrs.|Susannah|R|Pemberton|1585 Court Street|Maryland Heights|MO|63141|US|United States|SusannahRPemberton@rhyta.com|Turs1956|eiGufib3th|636-898-6411|Andrews|11/5/1956|MasterCard|5112913274177177|156|9/2015|490-10-6886|1Z 4Y0 424 93 3839 606 7|Purple|Sampler|Forth & Towne|2002 MCC Smart|RentHoldings.com|O+|218.7|99.4|5' 1"|156|c738a629-5108-47b1-a168-8ab6d1438e50|38.728008|-90.37838
2|male|Mr.|Anthony|J|Quirion|4928 Carson Street|San Diego|CA|92103|US|United States|AnthonyJQuirion@rhyta.com|Quoinep|ew2Keewoh|858-856-8403|Ferguson|7/5/1979|MasterCard|5533216847446447|542|1/2015|622-80-6651|1Z W28 086 52 9104 825 3|Orange|Shampooer|JumboSports|1993 Mazda 121|PoliticalPlanners.com|B+|140.8|64.0|5' 11"|180|79b9a58b-ada9-4fab-8093-d0f17c541cf1|32.831002|-117.113794
3|male|Mr.|Shaun|P|Schwartz|1246 Kyle Street|Grand Island|NE|68801|US|United States|ShaunPSchwartz@fleckens.hu|Sivend88|saeKooca8r|308-615-8773|Guevara|10/23/1988|MasterCard|5562020815274730|454|10/2019|507-90-6312|1Z 96V V28 41 7598 869 9|White|Lease operator|Grass Roots Yard Services|2000 BMW 528|StationCard.com|O+|191.4|87.0|5' 10"|179|ebaa641c-0344-45aa-9649-b5ae134e9e61|40.897922|-98.182374
4|male|Mr.|Michael|B|Petrey|3875 Jehovah Drive|Spotsylvania|VA|22553|US|United States|MichaelBPetrey@superrito.com|Thibust|ooho3Ocoh|540-507-6775|Weaver|12/26/1972|MasterCard|5420696909140416|637|5/2016|230-80-7928|1Z 8V1 528 75 9778 835 3|Red|Telecommunications specialist|Great American Music|1995 Ford Ranger|EmploymentCars.com|B+|165.9|75.4|6' 0"|182|dbc2c6d6-d967-4908-be39-2f4b22ba2c17|38.118885|-77.706684
```
```xml
<beanio xmlns="http://www.beanio.org/2012/03"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.beanio.org/2012/03 http://www.beanio.org/2012/03/mapping.xsd">
    
    <stream name="persons" format="delimited" ignoreUnidentifiedRecords="true">
        <parser>
            <property name="delimiter" value="|"/>
            <property name="comments" value="#"/>
        </parser>
        <record name="person" class="org.jberet.support.io.Person" occurs="0+">
            <field name="number"/>
            <field name="gender" />
            <field name="title" />
            <field name="givenName" />
            <field name="middleInitial" />
            <field name="surname" />
            <field name="streetAddress" />
            <field name="city" />
            <field name="state" />
            <field name="zipCode" />
            <field name="country" />
            <field name="countryFull" />
            <field name="emailAddress" />
            <field name="username" />
            <field name="password" />
            <field name="telephoneNumber" />
            <field name="mothersMaiden" />
            <field name="birthday" format="M/d/YYYY"/>
            <field name="CCType" />
            <field name="CCNumber" />
            <field name="CVV2" />
            <field name="CCExpires" />
            <field name="nationalID" />
            <field name="UPS" />
            <field name="color" />
            <field name="occupation" />
            <field name="company" />
            <field name="vehicle" />
            <field name="domain" />
            <field name="bloodType" />
            <field name="pounds" />
            <field name="kilograms" />
            <field name="feetInches" />
            <field name="centimeters" />
            <field name="GUID" />
            <field name="latitude" />
            <field name="longitude" />
        </record>
    </stream>
</beanio>
```

###Batch Properties for Both `beanIOItemReader` and `beanIOItemWriter`
####resource
The resource to read from (for batch readers), or write to (for batch writers). Some reader or writer implementations may choose to ignore this property and instead use other properties that are more appropritate.

####streamName
Name of the BeanIO stream defined in BeanIO mapping file. It corresponds to the batch job property.

####streamMapping
Location of the BeanIO mapping file, which can be a file path, a URL, or a resource loadable by the current class loader.

####streamFactoryLookup
JNDI name for looking up `org.beanio.StreamFactory` when running in application server. When `streamFactoryLookup` property is specified in job xml and hence injected into batch reader or writer class, `org.beanio.StreamFactory` will be looked up with JNDI, and `streamMapping` and `mappingProperties` will be ignored.

####mappingProperties
`java.util.Map`

User properties that can be used for property substitution in BeanIO mapping file. When used with batch job JSL properties, they provide dynamic BeanIO mapping attributes. For example,

1. In batch job client class, set the properties values as comma-separated key-value pairs:
```java
params.setProperty("mappingProperties", "zipCodeFieldName=zipCode, zipCodeFieldType=string");
```

2. In job xml file, make the properties available for `beanIOItemReader` or `beanIOItemWriter` via `@BatchProperty` injection:
```xml
<property name="mappingProperties" value="#{jobParameters['mappingProperties']}"/>
```

3. In BeanIO mapping file, reference the properties defined above:
```xml
<field name="${zipCodeFieldName}" type="${zipCodeFieldType}" length="5"/> 
```

####charset
The name of the character set to be used for reading and writing data, e.g., UTF-8. This property is optional, and if not set, the platform default charset is used.



###Batch Properties for `beanIOItemReader` Only
In addition to the above common properties, `beanIOItemReader` also supports the following batch properties in job xml:

####skipBeanValidation
`boolean`

Indicates whether the current batch reader will invoke Bean Validation API to validate the incoming data POJO. Optional property and defaults to `false`, i.e., the reader will validate data POJO bean where appropriate.

####start
`int`

A positive integer indicating the start position in the input resource. It is optional and defaults to `1` (starting from the 1st data item).

####end
`int`

A positive integer indicating the end position in the input resource. It is optional and defaults to `Integer.MAX_VALUE`.

####errorHandler
`java.lang.Class`

A class implementing `BeanReaderErrorHandler` for handling exceptions thrown by a `BeanReader`.

####locale
The locale name for this `beanIOItemReader`


###Batch Properties for `beanIOItemWriter` Only
None
 
