---
title: How to integrate a Benchmark
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: benchmark_integration.html
folder: docs
---

This article shows how to use the provided abstract classes to implement the different benchmark components using the interfaces and classes from the [core library](https://github.com/hobbit-project/core) of the HOBBIT platform. The role of the different components is described in the [overview article](/overview.html) If you haven't took a look at the [general development of a component in java](/java_components.html) you should do this before reading this article.

* [Data Generator](#data-generator)
* [Task Generator](#task-generator)
* [Evaluation Module](#evaluation-module)
* [Benchmark Controller](#benchmark-controller)
* [Adaptations](#adaptations)
    * [Sequential tasks](#sequential-tasks)

## Data Generator
The data generator is the component that creates the dataset the benchmark is using.

```java
public class ExampleDataGenrator extends AbstractDataGenerator {

    @Override
    public void init() throws Exception {
        // Always init the super class first!
        super.init();
        
        // Your initialization code comes here...
    }
    
    @Override
    protected void generateData() throws Exception {
        // Create your data inside this method. You might want to use the
        // id of this data generator and the number of all data generators
        // running in parallel.
        int dataGeneratorId = getGeneratorId();
        int numberOfGenerators = getNumberOfGenerators();
    
        byte[] data;
        while(notEnoughDataGenerated) {
            // Create your data here
            data = ...
            
            // the data can be sent to the task generator(s) ...
            sendDataToTaskGenerator(data);
            // ... and/or to the system
            sendDataToSystemAdapter(data);
        }
    }

    @Override
    public void close() throws IOException {
        // Free the resources you requested here
        ...
        
        // Always close the super class after yours!
        super.close();
    }

}
```

## Task Generator
The task generator is the component that creates tasks based on the incoming data.

```java
public class ExampleTaskGenrator extends AbstractTaskGenerator {

    @Override
    public void init() throws Exception {
        // Always init the super class first!
        super.init();
        
        // Your initialization code comes here...
    }
    
    @Override
    protected void generateTask(byte[] data) throws Exception {
        // Create tasks based on the incoming data inside this method.
        // You might want to use the id of this task generator and the
        // number of all task generators running in parallel.
        int dataGeneratorId = getGeneratorId();
        int numberOfGenerators = getNumberOfGenerators();
        
        // Create an ID for the task
        String taskId = getNextTaskId();
        
        // Create the task and the expected answer
        byte[] taskData = ...;
        byte[] expectedAnswerData = ...;
    
        // Send the task to the system (and store the timestamp)
        long timestamp = System.currentTimeMillis();
        sendTaskToSystemAdapter(taskId, taskData);
        
        // Send the expected answer to the evaluation store
        sendTaskToEvalStorage(taskId, timestamp, expectedAnswerData);
    }

    @Override
    public void close() throws IOException {
        // Free the resources you requested here
        ...
        
        // Always close the super class after yours!
        super.close();
    }

}
```
Note, that the `generateTask(byte[] data)` method is called with the data that has been received in one single message coming from a data generator.


## Evaluation Module
The evaluation module is the component that evaluates the results that have been received from the benchmarked system. It operates in two phases. During the first phase, the module iterates over the single tasks and their expected answers (created by the task generator(s)) as well as the answers coming from the benchmarked system. For every pair, the method `evaluateResponse` is called.

After all available pairs have been seen, the `summarizeEvaluation` method is called and should return an RDF model containing the values of the KPIs of the experiment.

```java
public class ExampleEvaluationModule extends AbstractEvaluationModule {

    @Override
    public void init() throws Exception {
        // Always init the super class first!
        super.init();
        
        // Your initialization code comes here...
    }
    
    @Override
    protected void evaluateResponse(byte[] expectedData, byte[] receivedData, long taskSentTimestamp, long responseReceivedTimestamp) throws Exception {
        // evaluate the given response and store the result, e.g., increment internal counters
        ...
    }
    
    @Override
    protected Model summarizeEvaluation() throws Exception {
        // All tasks/responsens have been evaluated. Summarize the results,
        // write them into a Jena model and send it to the benchmark controller.
        Model model = createDefaultModel();
        Resource experimentResource = mode.getResource(experimentUri);
        model.add(experimentResource , RDF.type, HOBBIT.Experiment);
        ...
        return model;
    }

    @Override
    public void close() throws IOException {
        // Free the resources you requested here
        ...
        
        // Always close the super class after yours!
        super.close();
    }

}
```

The following information do not have to be added to the result RDF model since they will be added by the platform.
* The platform version
* The time at which the execution of the experiment started
* The RDF model of the involved benchmark
* The values of the configured benchmark parameters
* The RDF model of the involved system instance and its system (if a meta system is defined)

## Benchmark Controller
The benchmark controller is used to create and orchestrate all other components needed to execute the benchmark.

Note that for the creation of benchmark components it is recommended to use the methods that are offered for the different types of components instead of using the more general `createContainer` method. This is caused by internal states of the `AbstractBenchmarkController` class that for example tracks the number of data generators that it has created.

```java
public class ExampleBenchmarkController extends AbstractBenchmarkController {


    @Override
    public void init() throws Exception {
        super.init();

        // Your initialization code comes here...

        // You might want to load parameters from the benchmarks parameter model
        NodeIterator iterator = benchmarkParamModel.listObjectsOfProperty(benchmarkParamModel
                    .getProperty("http://example.org/myParameter"));
        
        // Create the other components

        // Create data generators
        String dataGeneratorImageName = "example-data-generator";
        int numberOfDataGenerators = ...;
        String[] envVariables = new String[]{"key1=value1", ...};
        createDataGenerators(dataGeneratorImageName, numberOfDataGenerators, envVariables);
        
        // Create task generators
        String taskGeneratorImageName = "example-task-generator";
        int numberOfTaskGenerators = ...;
        envVariables = new String[]{"key1=value1", ...};
        createTaskGenerators(taskGeneratorImageName, numberOfTaskGenerators, envVariables);
        
        // Create evaluation storage
        createEvaluationStorage();
        
        // Wait for all components to finish their initialization
        waitForComponents();
    }

    @Override
    protected void executeBenchmark() throws Exception {
        // give the start signals
        sendToCmdQueue(Commands.TASK_GENERATOR_START_SIGNAL);
        sendToCmdQueue(Commands.DATA_GENERATOR_START_SIGNAL);

        // wait for the data generators to finish their work
        waitForDataGenToFinish();

        // wait for the task generators to finish their work
        waitForTaskGenToFinish();

        // wait for the system to terminate. Note that you can also use 
        // the method waitForSystemToFinish(maxTime) where maxTime is
        // a long value defining the maximum amount of time the benchmark 
        // will wait for the system to terminate.
        waitForSystemToFinish();

        // Create the evaluation module
        String evalModuleImageName = "example-eval-module";
        String[] envVariables = new String[]{"key1=value1", ...};
        createEvaluationModule(evalModuleImageName, envVariables);

        // wait for the evaluation to finish
        waitForEvalComponentsToFinish();

        // the evaluation module should have sent an RDF model containing the
        // results. We should add the configuration of the benchmark to this
        // model.
        // this.resultModel.add(...);

        // Send the resultModul to the platform controller and terminate
        sendResultModel(resultModel);
    }

    @Override
    public void close() throws IOException {
        // Free the resources you requested here
        ...
        
        // Always close the super class after yours!
        super.close();
    }

}
```

## Adaptations
The workflow described above can be further adapted to different scenarios.

### Sequential tasks
For some use cases it is important that the task generator sends the tasks sequentially, waiting for the current task to be finished before the next task can be started. To achieve such a behaviour the task generator and the evaluation storage have to be synchronized. This can be done by 
1. extending the `AbstractSequencingTaskGenerator` for implementing the task generator and
2. starting the evaluation storage with the environmental variable `"ACKNOWLEDGEMENT_FLAG=true"`
