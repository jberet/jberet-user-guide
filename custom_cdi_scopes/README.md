# Custom CDI Scopes

JBeret supports 3 custom CDI scopes with the corresponding annotations:
* `org.jberet.cdi.JobScoped`
* `org.jberet.cdi.StepScoped`
* `org.jberet.cdi.PartitionScoped`

These annotations can be applied to CDI beans in a batch application to specify their scopes within the batch application. For instance, a `@JobScoped` singleton bean can be used to share and exchange application state between various artifacts within the same job execution.

Note that they should not be applied to any standard batch artifact types such as batchlet, item reader/writer/processor, batch listener, partition reducer/collector/analyzer/mapper, etc.

## `JobScoped`
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


