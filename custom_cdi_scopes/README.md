# Custom CDI Scopes

JBeret supports 3 custom CDI scopes with the corresponding annotations:
* `org.jberet.cdi.JobScoped`
* `org.jberet.cdi.StepScoped`
* `org.jberet.cdi.PartitionScoped`

These annotations can be applied to CDI beans in a batch application to specify their scopes within the batch application. For instance, a `@JobScoped` singleton bean can be used to share and exchange application state between various artifacts within the same job execution. Or with a `@PartitionScoped` bean, different instances will be injected into targets in different partitions.

Note that they should not be applied to any standard batch artifact types such as batchlet, item reader/writer/processor, batch listener, partition reducer/collector/analyzer/mapper, etc.

## `JobScoped`
CDI beans with `@JobScoped` annotation are scoped to the current job execution. Injections of a bean type within the same job execution share the same bean instance. Injections of the same bean type across different job executions refers to different bean instances. `org.jberet.cdi.JobScoped` annotation is declared as follows:

```java
@Target({TYPE})
@Retention(RUNTIME)
@Documented
@NormalScope
@Inherited
public @interface JobScoped {}
```
To apply the annotation to CDI bean at class-level:
```java
@Named
@JobScoped
public class Foo {...}
```
To inject the above bean into a batch artifact:
```java
@Named
public class JobScopeBatchlet1 extends AbstractBatchlet {
    @Inject
    private Foo foo;
...
}
```

## `StepScoped`
Injections of a bean type within the same step execution share the same bean instance. Injections of the same bean type across different step executions refers to different bean instances. `org.jberet.cdi.StepScoped` annotation is declared as follows:

```java
@Target({TYPE})
@Retention(RUNTIME)
@Documented
@NormalScope
@Inherited
public @interface StepScoped {}
```
To apply the annotation to CDI bean at class-level:
```java
@Named
@StepScoped
public class Foo {...}
```
To inject the above bean into a batch artifact:
```java
@Named
public class StepScopedListener implements StepListener {
    @Inject
    private Foo foo;
...
}
```

## `PartitionScoped`
Injections of a bean type within the same partition execution share the same bean instance. Injections of the same bean type across different partition executions refers to different bean instances. `org.jberet.cdi.PartitionScoped` annotation is declared as follows:

```java
@Target({TYPE})
@Retention(RUNTIME)
@Documented
@NormalScope
@Inherited
public @interface PartitionScoped {}
```
To apply the annotation to CDI bean at class-level:
```java
@Named
@PartitionScoped
public class Foo {...}
```
To inject the above bean into a batch artifact:
```java
@Named
public class PartitionScopePartitionAnalyzer implements PartitionAnalyzer {
    @Inject
    private Foo foo;
...
}
```


