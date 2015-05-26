# Bean Validation API for Data POJO
Batch applications can leverage Bean Validation API to perform data validation, by embedding validation constraints in data POJO class. It is supported in all `ItemReader` implementation classes in jberet-support module.

In each `ItemReader` class, if the configured `beanType` for the incoming data is a custom POJO (i.e., not `java.util.Map`, or `java.util.List`, for example, `Person`, `StockTrade`, `Company`), the reader class will invoke `javax.validation.Validator#validate` to perform the validation.

Any validation constraint failure will cause the reader to throw `javax.validation.ConstraintViolationException`, a subclass of `java.lang.RuntimeException` and `javax.validation.ValidationException`.

The following POJO class, `StockTrade`, demonstrates certain usage of Bean Validation constraints:
```java
public class StockTrade implements Serializable {

    private static final long serialVersionUID = -55209987994863611L;
    
    @NotNull
    @Past
    Date date;
    
    @NotNull
    @Size(min = 3, max = 5)
    @Pattern(regexp = "^([0-9]|0[0-9]|1[0-9]|2[0-3]):[0-5][0-9]$")
    String time;
    
    @NotNull
    @Max(1000)
    @Min(1)
    double open;
    
    @Max(1000)
    @Min(1)
    double high;
    
    @Max(1000)
    @Min(1)
    double low;
    
    @Max(1000)
    @Min(1)
    double close;
    
    @NotNull
    @DecimalMin("100")
    @DecimalMax("9999999999")
    double volume;

    ...
```

## Dependencies
All `ItemReader` implementation classes in jberet-support module has a compile-time dependency on Bean Validation API:
```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>${version.javax.validation}</version>
</dependency>
```

If a batch application triggers bean validation at runtime, then the following runtime dependencies are also required:
```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>${version.org.hibernate.validator}</version>
</dependency>

<dependency>
    <groupId>org.jboss.spec.javax.el</groupId>
    <artifactId>jboss-el-api_3.0_spec</artifactId>
    <version>${version.org.jboss.spec.javax.el.jboss-el-api_3.0_spec}</version>
</dependency>

<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>javax.el</artifactId>
    <version>${version.org.glassfish.javax.el}</version>
</dependency>
```
Bean Validation may not be triggered when running a batch application, because
* item reader `beanType` property is not a custom POJO type; 
* bean validation is disabled in job xml for a particular `ItemReader` class.
 
## Disable Bean Validation
Bean validation is enabled by default for all `ItemReader` classes in jberet-support. It can be turned off by setting batch property `skipBeanValidation` to true on `reader` element in job.xml (its default value is false, which means bean validation is on):
```xml
<property name="skipBeanValidation" value="true"/>
```

Performing bean validation in your batch application has its performance overhead. So if processing speed and performance is high priority, consider disabling bean validation on `ItemReader` artifacts, or set item reader `beanType` property to a non-POJO type.

