---
title: Tutorial: Creating a simple benchmark
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: tutorial_example_benchmark.html
folder: docs
---

The goal of this tutorial is to create a basic, first benchmark from scratch. This tutorial relies on the two tutorials that describe the [general integration of benchmarks](https://hobbit-project.github.io/benchmark_integration.html) and [systems](https://hobbit-project.github.io/system_integration.html) into the HOBBIT platform but show the practical steps using an example scenario.

Throughout this tutorial, we will use emojis to mark parts of the tutorial. 📝 marks important details, that we have to remember later on. Other emojis might be used randomly 😉

# Prerequisites

TODO

1. prepare hobbit platform
2. have an IDE to develop java programs
3. have Docker installed (which version?)
4. either build the project locally (maven + java) or within the container
5. download necessary data https://archive.ics.uci.edu/dataset/186/wine+quality

# Scenario and Benchmark Design

The target of this tutorial is to implement a benchmark for a classification. As example, we use the [🍷 wine quality dataset](https://archive.ics.uci.edu/dataset/186/wine+quality) of Cortez et al. In the following, we will briefly look at the data, define the task that a benchmarked system should fulfill and design the benchmark that we are going to implement.

## Data 

The dataset comprises two files. One file is for red, the second is for white wine. In the remainder of the tutorial, we will handle these two files as two distinct datasets. However, both of them have the same structure. They come as CSV files in the following form:
```
"fixed acidity";"volatile acidity";"citric acid";"residual sugar";"chlorides";"free sulfur dioxide";"total sulfur dioxide";"density";"pH";"sulphates";"alcohol";"quality"
7;0.27;0.36;20.7;0.045;45;170;1.001;3;0.45;8.8;6
6.3;0.3;0.34;1.6;0.049;14;132;0.994;3.3;0.49;9.5;6
8.1;0.28;0.4;6.9;0.05;30;97;0.9951;3.26;0.44;10.1;6
7.2;0.23;0.32;8.5;0.058;47;186;0.9956;3.19;0.4;9.9;6
```
The structure of the file will be important later on. At the moment, it is sufficient to recognize that the file contains several features in a tabular format and that the last column contains the `"quality"`. This is the target class, that a system should predict.

## Task

We define the task that a system, which will be evaluated with our benchmark, should fulfill is defined as multi-class single-label classification. The classes are the quality levels 0 (very bad) to 10 (excellent). Note that the data itself only contains labels ranging from 3 to 9 and that the classes are very unbalanced, i.e., the most wines belong to the quality class 5, 6 or 7.

We further want to enable the usage of supervised machine learning. Hence, our benchmark has to ensure that the data is split into training and test data. The training data has to be made available for the system to train an algorithm. Further, we define that the split should be done randomly and that 90% of the data should be used for training.

The benchmark should measure the effectiveness and efficiency of a system. While the effectiveness can be measured with a large number of different measures, we focus on the usage of Micro Precision, Recall and F1-measure. The efficiency should be measured in form of the time that the system needs to classify the single examples.

## Benchmark Design

A major part of the work on a benchmark is to define its internal workflow and the API that the system has to implement to be benchmarked by the benchmark.

We rely on the [suggested workflow of a benchmark](https://hobbit-project.github.io/experiment_workflow.html). Hence, our benchmark implementation will comprise a Data Generator, a Task Generator, an Evaluation Storage, an Evaluation Module, and a Benchmark Controller. We will make use of a default implementation for the Evaluation Storage. All other components will be implemented within this tutorial.

In addition to the [general API of the HOBBIT platform](https://hobbit-project.github.io/system_integration_api.html) that has to be implemented by the system, the benchark's API comprises the following parts:
1. The task definition above already stated that the data has to be split to get training and test data. We define that the training data is sent to the system at the beginning as a CSV file of the form above, without the headline.
2. The system should have the time to train an internal algorithm using the training data. Hence, the evaluation of the system should only start after the system signalled that the internal training has finished and that it is ready to be benchmarked.
3. The benchmark will sent the single tasks (i.e., the instances of the test set) as CSV lines as above (without the quality column).

TODO Benchmark parameters:
* dataset name
* seed

# Project Setup

TODO pom file, etc.

# Data Generator

We create a class named `org.example.hobbit.benchmark.DataGenerator`. We copy the parts of the benchmark tutorial for the Data Generator into this class. After that, the class looks as follows:
```java
package org.example.hobbit.benchmark;

import java.io.IOException;

import org.hobbit.core.components.AbstractDataGenerator;

public class DataGenerator extends AbstractDataGenerator {

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
            data = null; //...
            
            // the data can be sent to the task generator(s) ...
            sendDataToTaskGenerator(data);
            // ... and/or to the system
            sendDataToSystemAdapter(data);
        }
    }

    @Override
    public void close() throws IOException {
        // Free the resources you requested here
        // ...
        
        // Always close the super class after yours!
        super.close();
    }

}
```
The compiler will show an error. However, that is not a problem since we will replace the faulty parts in the following. we can also ignore the `close` method. We won't need it within this tutorial. Instead, we focus on the implementation of the `init` and `generateData` methods. Within these methods, we want to load the data, split it into training and test data, send the training data as one message to the system and the test data to the Task Generator. 

We first start by loading the data from the file. It is worth noting that we assume that we already have the data in the container of the Data Generator 📝. However, we have two datasets—one for red and a second for white wine. As defined above, we assume that the user chose one of the datasets when configuring the benchmark. Hence, the Benchmark Controller (which will later on create our Data Generator) will have to forward this information. Within this tutorial, we will implement this as an environment variable, which we call `DATASET_FILE_NAME`. The Benchmark Controller will have to set this value when creating the Data Generator container 📝. Similarly, we will expect to receive a seed value that we use for the random split of training and test data as environment variable 📝. We implement this part of the Data Generator as follows:
```diff

+import java.io.File;
import java.io.IOException;
+import java.nio.charset.StandardCharsets;
+import java.util.List;

+import org.apache.commons.io.FileUtils;
import org.hobbit.core.components.AbstractDataGenerator;
+import org.slf4j.Logger;
+import org.slf4j.LoggerFactory;

public class DataGenerator extends AbstractDataGenerator {
+
+   private static final Logger LOGGER = LoggerFactory.getLogger(DataGenerator.class);
+
+   public static final String FILE_NAME_ENV_KEY = "DATASET_FILE_NAME";
+   public static final String SEED_ENV_KEY = "SEED_FOR_RNG";
+   public static final String DATASET_FOLDER = "/data/";
+
+   protected String fileName;
+   protected long seed;

    @Override
    public void init() throws Exception {
        // Always init the super class first!
        super.init();

        // Your initialization code comes here...
+       fileName = this.getConfiguration().getString(FILE_NAME_ENV_KEY, LOGGER);
+       seed = this.getConfiguration().getLong(SEED_ENV_KEY, LOGGER);
    }

    @Override
    protected void generateData() throws Exception {
-       // Create your data inside this method. You might want to use the
-       // id of this data generator and the number of all data generators
-       // running in parallel.
-       int dataGeneratorId = getGeneratorId();
-       int numberOfGenerators = getNumberOfGenerators();
+       // Read data from file
+       List<String> dataset = FileUtils.readLines(new File(DATASET_FOLDER + fileName), StandardCharsets.UTF_8);
```
The implementation above defines a `LOGGER` which is used within the `init` method to log errors in case one of the environmental variables cannot be read. The three static final fields define the name of the two environmental variables and the directory to the data within the Data Generator's container. The latter is important when creating the container image later on 📝.

Within the `init` method the file name and the seed value are gathered and stored in class attributes. We can remove the first lines at the beginning of the `generateData` method. In this tutorial, we won't use several Data Generators. Hence, the generator ID and the number of generators are not important. However, we add a method that reads the lines from the dataset file.

The next step is to randomly generate the 


The latter data will be sent in several messages. Each of them comprises a single 




![Configuration of a benchmarking experiment. (a) Select the benchmark.](/images/10_Benchmark.png)


