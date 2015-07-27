# Programmatic Job Definition with Java

In addition to defining batch jobs with job XML, one can also define a batch job programmatically using builder classes in `org.jberet.job.model` package:

* `org.jberet.job.model.JobBuilder`
* `org.jberet.job.model.StepBuilder`
* `org.jberet.job.model.FlowBuilder`
* `org.jberet.job.model.SplitBuilder`
* `org.jberet.job.model.DecisionBuilder`

##`JobBuilder`

Builder class for building a single job. After the job is built, the same `JobBuilder` instance should not be reused to build another job.

Jobs built programmatically with this builder and other related builder classes do not support JSL inheritance.

This class does not support multi-threaded access or modification. 

Usage example,
```java
Job job = new JobBuilder(jobName)
      .restartable(false)
      .property("jobk1", "J")
      .property("jobk2", "J")
      .listener("jobListener1", new String[]{"jobListenerk1", "#{jobParameters['jobListenerPropVal']}"},
              new String[]{"jobListenerk2", "#{jobParameters['jobListenerPropVal']}"})

      .step(new StepBuilder(stepName)
              .properties(new String[]{"stepk1", "S"}, new String[]{"stepk2", "S"})
              .batchlet(batchlet1Name, new String[]{"batchletk1", "B"}, new String[]{"batchletk2", "B"})
              .listener("stepListener1", stepListenerProps)
              .stopOn("STOP").restartFrom(stepName).exitStatus()
              .endOn("END").exitStatus("new status for end")
              .failOn("FAIL").exitStatus()
              .nextOn("*").to(step2Name)
              .build())

      .step(new StepBuilder(step2Name)
              .batchlet(batchlet1Name)
              .build())

              .build();
 ```

##`StepBuilder`

Builder class for building a single step. After the step is built, the same `StepBuilder` instance should not be reused to build another step.

This class tries to model the `jsl:Step` element in job XML, while keeping a somewhat flattened structure. Methods for setting step sub-elements resides directly under `StepBuilder` where possible, avoiding the need for drilling down to sub-elements and popping back to parent context. For instance,

* batchlet-specific method, `batchlet(String, java.util.Properties)` and `batchlet(String, java.util.Properties)` are directly in `StepBuilder`;
    
* chunk-specific method, `reader(String, java.util.Properties)`, `processor(String, java.util.Properties)`, `writer(String, String[]...)`, etc are directly in `StepBuilder`, with no intermediary chunk builder;
    
* partition-specific method, `partitionPlan(int, int, List)`, `partitionMapper(String, String[]...)`, `partitionReducer(String, java.util.Properties)`, `partitionCollector(String, String[]...)`, etc are directly in `StepBuilder`, with no intermediary partition builder. 

However, transition methods, such as `endOn(String)`, `stopOn(String)`, `failOn(String)`, and `nextOn(String)` will drill down to `Transition.End`, `Transition.Stop`, `Transition.Fail`, and `Transition.Next` respectively. These classes all contain a terminating method, which pops the context back to the current `StepBuilder`.

This class does not support multi-threaded access or modification. 

Usage example,
```java
Step step1 = new StepBuilder(step1Name).batchlet(batchlet1Name).build();

Step step2 = new StepBuilder(step2Name)
      .properties(new String[]{"stepk1", "S"}, new String[]{"stepk2", "S"})
      .batchlet(batchlet1Name, new String[]{"batchletk1", "B"}, new String[]{"batchletk2", "B"})
      .listener("stepListener1", stepListenerProps)
      .stopOn("STOP").restartFrom(step1Name).exitStatus()
      .endOn("END").exitStatus("new status for end")
      .failOn("FAIL").exitStatus()
      .nextOn("*").to(step3Name)
      .build());

Step step3 = new StepBuilder(step3Name)
      .reader("integerArrayReader", new String[]{"data.count", "30"})
      .writer("integerArrayWriter", new String[]{"fail.on.values", "-1"}, new String[]{"writer.sleep.time", "0"})
      .processor("integerProcessor")
      .checkpointPolicy("item")
      .listener("chunkListener1", new String[]{"stepExitStatus", stepExitStatusExpected})
      .itemCount(10)
      .allowStartIfComplete()
      .startLimit(2)
      .skipLimit(8)
      .timeLimit(2, TimeUnit.MINUTES)
      .build());

Step step4 = new StepBuilder(stepName)
      .reader("integerArrayReader", new String[]{"data.count", "30"},
              new String[]{"partition.start", "#{partitionPlan['partition.start']}"},
              new String[]{"partition.end", "#{partitionPlan['partition.end']}"})
      .writer("integerArrayWriter", new String[]{"fail.on.values", "-1"}, new String[]{"writer.sleep.time", "0"})
      .partitionMapper("partitionMapper1", new String[]{"partitionCount", String.valueOf(partitionCount)})
      .partitionCollector("partitionCollector1")
      .partitionAnalyzer("partitionAnalyzer1")
      .partitionReducer("partitionReducer1")
      .build())
```

##`DecisionBuilder`

Builder class for building a single `Decision`. After the decision is built, the same `DecisionBuilder` instance should not be reused to build another decision.

This class does not support multi-threaded access or modification. 

Usage example:
```java
Decision decision = new DecisionBuilder("decision1", "decider1")
  .failOn("FAIL").exitStatus()
  .stopOn("STOP").restartFrom(stepName).exitStatus()
  .nextOn("NEXT").to(stepName)
  .endOn("*").exitStatus(stepName)
  .build()
```

##`FlowBuilder`

Builder class for building a single `Flow`. After the flow is built, the same `FlowBuilder` instance should not be reused to build another flow.

This class does not support multi-threaded access or modification. 

Usage example:
```java
Flow flow = new FlowBuilder(flowName)
      .step(new StepBuilder(stepName).batchlet(batchlet1Name)
              .next(step2Name)
              .build())
      .step(new StepBuilder(step2Name).batchlet(batchlet1Name)
              .build())
      .build())
```

##`SplitBuilder`

Builder class for building a single `Split`. After the split is built, the same `SplitBuilder` instance should not be reused to build another split.

This class does not support multi-threaded access or modification. 

Usage example:
```java
Split split = new SplitBuilder(splitName)
      .flow(new FlowBuilder(flowName)
              .step(new StepBuilder(stepName).batchlet(batchlet1Name).build())
              .build())
      .flow(new FlowBuilder(flow2Name)
              .step(new StepBuilder(step2Name).batchlet(batchlet1Name).build())
              .build())
      .next(step3Name)
      .build())
```
