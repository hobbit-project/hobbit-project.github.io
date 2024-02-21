---
title: Tutorial for creating a simple benchmark
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: true
permalink: tutorial_example_benchmark.html
folder: docs
---

The goal of this tutorial is to create a basic, first benchmark from scratch. This tutorial relies on the two tutorials that describe the [general integration of benchmarks](benchmark_integration.html) and [systems](system_integration.html) into the HOBBIT platform but show the practical steps using an example scenario.

Throughout this tutorial, we will use emojis to mark parts of the tutorial. 📝 marks important details, that we have to remember later on. ↪️ marks the places at which we should remember these details. The two icons should be linked to each other, i.e., if you click on the notepad or arrow icon you should jump to the place where something is used later on or has been defined before, respectively. Other emojis might be used randomly 😉

## Prerequisites

TODO

1. prepare hobbit platform
2. have an IDE to develop java programs
3. have Docker installed (which version?)
micha@mprec:~/workspace/enexa-documentation$ docker -v
Docker version 25.0.2, build 29cf629
4. either build the project locally (maven + java) or within the container
micha@mprec:~/workspace/enexa-documentation$ mvn -v
Apache Maven 3.6.3
Maven home: /usr/share/maven
Java version: 17.0.9
5. download necessary data https://archive.ics.uci.edu/dataset/186/wine+quality

## Scenario and Benchmark Design

The target of this tutorial is to implement a benchmark for a regression task. As example, we use the [🍷 wine quality dataset](https://archive.ics.uci.edu/dataset/186/wine+quality) of Cortez et al. In the following, we will briefly look at the data, define the task that a benchmarked system should fulfill and design the benchmark that we are going to implement.

### Data 

The dataset comprises two files. One file is for red, the second is for white wine. In the remainder of the tutorial, we will handle these two files as two distinct datasets. However, both of them have the same structure. They come as CSV files in the following form:
```
"fixed acidity";"volatile acidity";"citric acid";"residual sugar";"chlorides";"free sulfur dioxide";"total sulfur dioxide";"density";"pH";"sulphates";"alcohol";"quality"
7;0.27;0.36;20.7;0.045;45;170;1.001;3;0.45;8.8;6
6.3;0.3;0.34;1.6;0.049;14;132;0.994;3.3;0.49;9.5;6
8.1;0.28;0.4;6.9;0.05;30;97;0.9951;3.26;0.44;10.1;6
7.2;0.23;0.32;8.5;0.058;47;186;0.9956;3.19;0.4;9.9;6
```
The structure of the file will be important later on. At the moment, it is sufficient to recognize that the file contains several features in a tabular format and that the last column contains the `"quality"`. This is the target value, that a system should predict.

### Task

We define the task that a system, which will be evaluated with our benchmark, should fulfill is defined as a regression. The target value are the quality levels 0 (very bad) to 10 (excellent). Note that the data itself only contains labels ranging from 3 to 9 and that the data is unbalanced, i.e., the most wines belong to the quality levels 5, 6 or 7.

We further want to enable the usage of supervised machine learning. Hence, our benchmark has to ensure that the data is split into training and test data. The training data has to be made available for the system to train an algorithm. Further, we define that the split should be done randomly and that 90% of the data should be used for training.

The benchmark should measure the effectiveness and efficiency of a system. While the effectiveness can be measured with a large number of different measures, we focus on the usage of Micro Precision, Recall and F1-measure. The efficiency should be measured in form of the time that the system needs to classify the single examples.

### Benchmark Design

A major part of the work on a benchmark is to define its internal workflow and the API that the system has to implement to be evaluated by the benchmark. We rely on the [suggested workflow of a benchmark](experiment_workflow.html). Hence, our benchmark implementation will comprise a Data Generator, a Task Generator, an Evaluation Storage, an Evaluation Module, and a Benchmark Controller. We will make use of a default implementation for the Evaluation Storage. All other components will be implemented within this tutorial.

In addition to the [general API of the HOBBIT platform](system_integration_api.html) that has to be implemented by the system, the benchark's API comprises the following parts:
1. The task definition above already stated that the data has to be split to get training and test data. We define that 10% of the dataset are randomly chosen as test dataset. The remaining 90% are used as training data and are sent to the system at the beginning as a CSV file of the form above, without the headline.
2. The system should have the time to train an internal algorithm using the training data. Hence, the evaluation of the system should only start after the system signalled that the internal training has finished and that it is ready to be benchmarked.
3. The benchmark will sent the single tasks (i.e., the instances of the test set) as CSV lines as above (without the quality column).

Our benchmark will have several parameters:
* **Dataset**: There are two wine datasets. One for red and a second for white wine. The user should be able to choose which dataset they want to use.
* **Seed**: The split into train and test data will be done randomly. However, the user should be able to define a seed value to ensure that an experiment is repeatable.
* **Test dataset size**: The benchmark should report the size of the randomly created test dataset.

The benchmark should provide the following evaluation results (also called key performance indicators (KPIs)):
* **MSE**: The mean squared error (MSE) that the benchmarked system achieves.
* **Runtime**: The average runtime that the system needs to answer a request (including its standard deviation).
* **Faulty responses**: The number of faulty responses that the system may have produced. This avoids to include them into the error calculation and allows the benchmark to report that the system did not always create correctly formed answers.

The figure below gives an overview of the benchmark and its components as well as the type of data that they send to each other.
In this tutorial, we will implement all components, except the Evaluation Storage for which we will reuse an existing implementation.

<p align="center">
  <img src="/images/Components-diagram-wine-benchmark.svg" />
</p>

## Project Setup

We create a directory (e.g., `wine-benchmark`) in which we are going to work. We start by extracting the downloaded wine dataset into the directory. We should have the following three files in the directory:
* `winequality.names` (this file is not really needed)
* `winequality-red.csv`
* `winequality-white.csv`

### Benchmark Meta Data

Our benchmark has to provide meta data about itself to enable the HOBBIT platform to match it with systems that implement the benchmark's API and to run the benchmark's controller. Hence, we create the file `benchmark.ttl` in our project and open it with an editor. In the following, we will add the meta data for our benchmark as [RDF](https://www.w3.org/TR/rdf11-primer/) written using the [Turtle](https://www.w3.org/TR/2014/REC-turtle-20140225/) serialization. Note that for this tutorial, it is not necessary to be an RDF expert. We will point out the important lines, explain their content and emphasize which parts may have to be adapted for your own benchmark implementation.

We start with the prefixes. They simply define namespaces and work similar to XML namespaces. This eases the handling of all the IRIs in the file and make it more readable.
```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix : <http://example.org/wine-example/> .

```
We define prefixes for the two standard vocabularies [RDFS](https://www.w3.org/TR/rdf12-schema/) and [XSD](https://www.w3.org/TR/xmlschema11-1/). The first is mainly used to define labels and descriptions, which are important for a user-friendly representation of the benchmark parameters and results in HOBBIT's user interface. The latter defines datatypes that we want to use within the meta data file. Apart from these two standard vocabularies, the list of prefixes also contains the `hobbit` prefix for the vocabulary used by the HOBBIT platform. The last prefix, which is simply written as `:` defines the namespace of the elements of our benchmark. We choose `http://example.org/wine-example/` as namespace for our benchmark. 

⚠️ **Note**: It is important that you define **your own namespace** for your benchmark, e.g., by replacing `wine-example` or by adding an additional path element to form a longer namespace, e.g., `http://example.org/max-power/wine-example/`. The IRI (i.e., the identifier) of our benchmark relies on the namespace that we define and the HOBBIT platform relies on the basic assumption of RDF that these identifiers are global. Two benchmarks that have the same IRI will be handled as a single benchmark by the platform, which may lead to the situation that you create a benchmark but are not able to run it within the platform.

In this tutorial, we define our benchmark as an entity within our meta data as follows:
```turtle
:Benchmark a hobbit:Benchmark;
  rdfs:label "Wine Example Benchmark"@en;
  rdfs:comment "This is an example benchmark based on a wine quality prediction scenario suggested by Cortez et al."@en;
  hobbit:imageName "hobbit-wine-example-controller";
  hobbit:usesImage
    "hobbit-wine-example-data-generator",
    "hobbit-wine-example-task-generator",
    "hobbit-wine-example-evaluation-module";
  hobbit:hasParameter
    :dataset,
    :seed,
    :testDataSize;
  hobbit:measuresKPI
    :mse,
    :runtime,
    :runtimeStdDev,
    :faultyResponses;
  hobbit:hasAPI :Api .
```
The first line states that our benchmark (with the IRI `http://example.org/wine-example/Benchmark`) is a HOBBIT benchmark. The next two lines define it's human-readable label and give a short description.

The next five lines define names of Docker images that our benchmark relies on. With `hobbit:imageName`, we define the Docker image name of our benchmark controller (here, we use `hobbit-wine-example-controller` <span id="controller-image-defined">[📝](#controller-image-used)</span>). With `hobbit:usesImage`, we can inform the HOBBIT platform about additional Docker images that our benchmark is going to use. This is helpful in case we use large images that need time to be downloaded. In this tutorial, we only add these lines for completeness and define names for the containers of our Data Generator <span id="data-gen-image-defined">[📝](#data-gen-image-used)</span>, Task Generator <span id="task-gen-image-defined">[📝](#task-gen-image-used)</span> and Evaluation Module <span id="eval-module-image-defined">[📝](#eval-module-image-used)</span>.

⚠️ **Note**: These Docker image names can be used for local development. To upload the images later on, the image name should contain the service to which they will be uploaded. See the [Docker web page](https://docs.docker.com/engine/reference/commandline/image_tag/#description) for a detailed explanation.

With `hobbit:hasParameter` and `hobbit:measuresKPI`, we define the three parameters and four KPIs, respectively. We will add more information about them in the following. Finally, we define the IRI `http://example.org/wine-example/Api` as the identifier for the API that our benchmark implements. Each system that can be evaluated by our benchmark should have list this IRI in it's metadata to express that it implements the API of our benchmark. All these systems will be listed by the HOBBIT platform when we choose our benchmark later on in its graphical user interface.

In the following lines, we define the details of the benchmark's parameters. We start with the dataset parameter:
```turtle
:dataset a hobbit:Parameter, hobbit:ConfigurableParameter;
  rdfs:label "Dataset"@en;
  rdfs:comment "The name of the dataset that should be used."@en;
  rdfs:domain hobbit:Experiment, hobbit:Challenge;
  rdfs:range :WineDataset;
  hobbit:defaultValue :CortezWhite .

:CortezRed a :WineDataset;
  rdfs:label "Red wine"@en;
  rdfs:comment "The red wine dataset proposed by Cortez et al."@en;
  :fileName "winequality-red.csv" .

:CortezWhite a :WineDataset;
  rdfs:label "White wine"@en;
  rdfs:comment "The white wine dataset proposed by Cortez et al."@en;
  :fileName "winequality-white.csv" .
```
The first part defines the `dataset` parameter (<span id="dataset-param-iri-defined">[📝](#dataset-param-iri-used)</span>). The definition comes with a name, a description, the class `WineDataset` as range (i.e., as set of possible values) and the `CortezWhite` dataset as default value.

The two parts below are the definitions of the two wine dataset—the red and the white wine datasets. Both have labels and descriptions so that a user can easily choose one of them later on. Both also have a value for the `fileName` property which we are going to use later on <span id="file-name-prop-defined">[📝](#file-name-prop-used)</span>. The values are simply the names of the two files that are located in the zip file of the [🍷 wine quality dataset](https://archive.ics.uci.edu/dataset/186/wine+quality) of Cortez et al.

Our benchmark has two more parameters that we define below:
```turtle
:seed a hobbit:Parameter, hobbit:ConfigurableParameter;
  rdfs:label "Seed"@en;
  rdfs:comment "A seed value for initialising random number generators is used to ensure the repeatability of experiments."@en;
  rdfs:domain hobbit:Experiment, hobbit:Challenge;
  rdfs:range xsd:long;
  hobbit:defaultValue "42"^^xsd:integer .

:testDataSize a hobbit:Parameter, hobbit:FeatureParameter ;
  rdfs:label "Test dataset size"@en;
  rdfs:comment "The number of instances in the test dataset."@en;
  rdfs:domain hobbit:Experiment, hobbit:Challenge;
  rdfs:range xsd:long .
```
Like the `dataset` parameter, the `seed` (<span id="seed-param-iri-defined">[📝](#seed-param-iri-used)</span>) is a `hobbit:ConfigurableParameter`. This means that the user will be able to set it later on in the user interface of the HOBBIT platform. It comes with a label, a description, the information that it expects long as datatype and a default value.

The `testDataSize` parameter is not configurable (<span id="test-size-param-iri-defined">[📝](#test-size-param-iri-used)</span>). It will be used to report the size of the test dataset created by the Data Generator.

Finally, we define the four KPIs that our benchmark should measure.
```turtle
:mse a hobbit:KPI ;
  rdfs:label "Mean squared error (MSE)"@en;
  rdfs:comment "The arithmetic average of the squared errors."@en;
  rdfs:domain hobbit:Experiment, hobbit:Challenge;
  rdfs:range xsd:double .

:avgRuntime a hobbit:KPI ;
  rdfs:label "Average runtime (in ms)"@en;
  rdfs:comment "The average runtime the system needed to predict the quality of a single wine in milliseconds."@en;
  rdfs:domain hobbit:Experiment, hobbit:Challenge;
  rdfs:range xsd:double .

:stdDevRuntime a hobbit:KPI ;
  rdfs:label "Runtime standard deviation"@en;
  rdfs:comment "The standard deviation of the runtime."@en;
  rdfs:domain hobbit:Experiment, hobbit:Challenge;
  rdfs:range xsd:double .

:faultyResponses a hobbit:KPI ;
  rdfs:label "Number of faulty responses"@en;
  rdfs:comment "The number of responses provided by the system that couldn't be parsed by the benchmark."@en;
  rdfs:domain hobbit:Experiment, hobbit:Challenge;
  rdfs:range xsd:long .
```
We define the `mse`, `avgRuntime`, `stdDevRuntime`, and `faultyResponses` IRIs for the KPIs (<span id="kpi-iris-defined">[📝](#kpi-iris-used)</span>).
We define labels and descriptions for all four KPIs. Four of them will have a floating point number value while the last one will have an long value.

**Hint**: you may want to check the validity of the file, e.g., by using a [Turtle validator](http://ttl.summerofcode.be/). ✅

### Maven Project

We implement our benchmark using [Maven](https://maven.apache.org/) as build automation tool. However, using Maven is not mandatory and you can use any other tool that helps you to build the project at the end. When you use Maven, you either have an IDE that generates the directory structure or you create it manually. We will do the latter by creating two directories:
* `src/main/java` is the directory in which we locate all our java classes.
* `src/main/resources` is the directory for additional resources that we will need.

Further, we create the [`pom.xml`](https://maven.apache.org/pom.html) file in our work directory, which is the XML representation of our project and its meta data needed by Maven. Our file looks like follows:
```
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.dice-research</groupId>
    <artifactId>hobbit.example.wine.benchmark</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Wine example benchmark</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.target>17</maven.compiler.target>
        <maven.compiler.source>17</maven.compiler.source>
    </properties>

    <repositories>
        <repository>
            <id>maven.aksw.internal</id>
            <name>University Leipzig, AKSW Maven2 Repository</name>
            <url>https://maven.aksw.org/repository/internal</url>
        </repository>
        <repository>
            <id>maven.aksw.snapshots</id>
            <name>University Leipzig, AKSW Maven2 Repository</name>
            <url>https://maven.aksw.org/repository/snapshots</url>
        </repository>
    </repositories>

    <dependencies>
        <!-- HOBBIT core library -->
        <dependency>
            <groupId>org.hobbit</groupId>
            <artifactId>core</artifactId>
            <version>1.0.23</version>
        </dependency>
        <!-- ~~~~~~~~~~~~~~~~~~~ Logging ~~~~~~~~~~~~~~~~~~~~~~ -->
        <!-- slf4j: Logging API -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.26</version>
        </dependency>
        <!-- slf4j: Logging Binding -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.26</version>
        </dependency>
        <!-- ~~~~~~~~~~~~~~~~~~~ End Logging ~~~~~~~~~~~~~~~~~~~~~~ -->
    </dependencies>

    <build>
        <finalName>wine-benchmark</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.target}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.3</version>
                <configuration>
                    <!-- filter all the META-INF files of other artifacts -->
                    <filters>
                        <filter>
                            <artifact>*:*</artifact>
                            <excludes>
                                <exclude>META-INF/*.SF</exclude>
                                <exclude>META-INF/*.DSA</exclude>
                                <exclude>META-INF/*.RSA</exclude>
                            </excludes>
                        </filter>
                    </filters>
                    <transformers>
                        <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                            <manifestEntries>
                                <X-Compile-Source-JDK>${maven.compile.source}</X-Compile-Source-JDK>
                                <X-Compile-Target-JDK>${maven.compile.target}</X-Compile-Target-JDK>
                            </manifestEntries>
                        </transformer>
                        <transformer
                            implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
                    </transformers>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```
The pom file defines the group ID, artifact ID and name of our project. Feel free to adapt them to your needs. The `repositories` section contains the definition of a repo of University Leipzig hosting the `hobbit.core` library which is listed in the `dependencies` section. This library already comes with a lot of basic functionalities related to HOBBIT that we want to use. As additional dependencies, we define the slf4j logging framework and its bridge to log4j. In the `build` section, we define the final name of the artifact that Maven should create (<span id="artifact-name-defined">[📝](#artifact-name-used)</span>). Further, we use two plugins. The compiler plugin allows us to define the version of the Java compiler while the shade plugin generates a single runnable jar, which we will need for the creation of our Docker images.

### Logging

We are going to use [log4j](https://logging.apache.org/log4j/2.x/) as logging framework. We create the log4j configuration file `src/main/resources/log4j.properties` with the following content:
```properties
log4j.rootLogger=info, stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
```
This file defines that all log messages should be printed to the standard output. This is helpful as it allows us to get log messages from the Docker containers in case we need them.

## Data Generator

The Data Generator has to load the dataset, split the data into training and test data. After that, the training data should be sent to the benchmarked system while the test data is sent to the Task Generator (line by line).

<p align="center">
  <img style="max-width:400px;height:auto;width:auto;" src="/images/Components-diagram-wine-benchmark-Data-Generator.svg" />
</p>

### Java Class

We create a class named `org.example.hobbit.wine.benchmark.DataGenerator`. We copy the parts of the [benchmark tutorial](benchmark_integration.html) for the Data Generator into this class. After that, the class looks as follows:
```java
package org.example.hobbit.wine.benchmark;

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
The compiler will show an error. However, that is not a problem since we will replace the faulty parts in the following. We can also ignore the `close` method. We won't need it within this tutorial. Instead, we focus on the implementation of the `init` and `generateData` methods. Within these methods, we want to load the data, split it into training and test data, send the training data as one message to the system and the test data to the Task Generator. 

We first start by loading the data from the file. It is worth noting that we assume that we already have the data inside the container of the Data Generator 📝. However, we have two datasets—one for red and a second for white wine. As defined above, we assume that the user chose one of the datasets when configuring the benchmark. Hence, the Benchmark Controller (which will later on create our Data Generator) will have to forward this information. Within this tutorial, we will implement this as an environment variable, which we call `DATASET_FILE_NAME`. The Benchmark Controller will have to set this value when creating the Data Generator container 📝. Similarly, we will expect to receive a seed value that we use for the random split of training and test data as environment variable named `SEED_FOR_RNG` 📝. We implement this part of the Data Generator as follows:
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

Within the `init` method the file name and the seed value are gathered and stored in class attributes. The main work of the Data Generator is done in the `generateData` method. In this tutorial, we won't use several Data Generators. Hence, the generator ID and the number of generators are not important and we can remove the first lines at the beginning of the method. However, we add a line that reads the content of the dataset file separated by line breaks.

The next step is to randomly select lines of the dataset that are either part of the training or test data. We can do that with adding the following pieces of code at the beginning of the class and to the `generateData` method, respectively:
```diff
+import java.util.ArrayList;
import java.util.List;
+import java.util.Random;
```
```diff
         // Read data from file
         List<String> dataset = FileUtils.readLines(new File(DATASET_FOLDER + fileName), StandardCharsets.UTF_8);

+        // Randomly select 10% test data
+        Random random = new Random(seed);
+        List<String> train = new ArrayList<>();
+        List<String> test = new ArrayList<>();
+        for (String instance : dataset.subList(1, dataset.size())) {
+            if (random.nextDouble() < 0.1) {
+                test.add(instance);
+            } else {
+                train.add(instance);
+            }
+        }
```
We randomly select 10% of the data as test data and keep the remaining 90% as training data. Note that we use the given seed to initialize the random number generator. We also skip the first line of the dataset by creating a sublist since this line contains the column headers.

Finally, we can send the training data to the system and the test data to the Task Generator. We send the training data in the same CSV format (except the header line). The single lines of the test data will be the tasks that our Task Generator. Hence, we send them one by one. The changes to the class look as follows:
```diff
                train.add(instance);
            }
        }

+        // Send training data to system
+        sendDataToSystemAdapter(transformToBytes(train));
+
+        // Send test data to task generator (one by one)
+        for (String instance : test) {
+            sendDataToTaskGenerator(instance.getBytes(StandardCharsets.UTF_8));
+        }
-        byte[] data;
-        while(notEnoughDataGenerated) {
-            // Create your data here
-            data = null; //...
-            
-            // the data can be sent to the task generator(s) ...
-            sendDataToTaskGenerator(data);
-            // ... and/or to the system
-            sendDataToSystemAdapter(data);
-        }  
    }

+    private byte[] transformToBytes(List<String> data) {
+        StringBuilder builder = new StringBuilder();
+        for (String d : data) {
+            builder.append(d);
+            builder.append('\n');
+        }
+        return builder.toString().getBytes(StandardCharsets.UTF_8);
+    }
```
We call the `sendDataToSystemAdapter` and prepare a single message containing all training data with the additional `transformToBytes` method. After that, we iterate the lines of the test data and send each one using the `sendDataToTaskGenerator` method. We also remove the example lines that we copied from the [benchmark tutorial](benchmark_integration.html) and that we do not need.

TODO add link to the final Data Generator file.

### Docker Image

TODO define Dockerfile
<span id="#artifact-name-used">[↪️](artifact-name-defined)</span>

<span id="data-gen-image-used">[↪️](#data-gen-image-defined)</span>.

## Task Generator

TODO AbstractSequencingTaskGenerator

The Task Generator receives the test data. Each line that the Task Generator receives will be split into the feature values and the expected quality value. The features should be sent to the benchmarked system while the expected quality value should be sent to the evaluation storage.

<p align="center">
  <img style="max-width:500px;height:auto;width:auto;" src="/images/Components-diagram-wine-benchmark-Task-Generator.svg" />
</p>

### Java Class

Again, we start our implementation by creating a new class named `org.example.hobbit.wine.benchmark.TaskGenerator`. We copy the parts of the [benchmark tutorial](benchmark_integration.html) for the Task Generator into our new class. After that, the class should look as follows:
```java
package org.example.hobbit.wine.benchmark;

import java.io.IOException;

import org.hobbit.core.components.AbstractTaskGenerator;

public class TaskGenerator extends AbstractTaskGenerator {

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
Our Task Generator is quite simple and doesn't need a special initialization. Hence, we can ignore the `init` method. Similarly, we only remove the unnecessary lines from the close method and ignore it after that as well:
```diff
     @Override
     public void close() throws IOException {
-        // Free the resources you requested here
-        ...
-        
         // Always close the super class after yours!
         super.close();
     }
```
Our main focus is on the `generateTask` method. This method receives the data of the Data Generator and should create a task based on that data. We create a task by separating the expected value (i.e., the human rating of the wine) from the features that the system will receive as input. The expecteed answer is the value in the last column of the CSV file line that we get from our Data Generator. Hence, we simply can search for the last column separator.
```diff
import java.io.IOException;
+import java.nio.charset.StandardCharsets;
```
```diff
     @Override
     protected void generateTask(byte[] data) throws Exception {
+        // Create tasks based on the incoming data inside this method.
+        String line = new String(data, StandardCharsets.UTF_8);
+        // Separate the answer from the feature data
+        int pos = line.lastIndexOf(';');
+        if (pos < 0) {
+            // Something is wrong with the data. Let's ignore it.
+            return;
+        }
-        // Create tasks based on the incoming data inside this method.
-        // You might want to use the id of this task generator and the
-        // number of all task generators running in parallel.
-        int dataGeneratorId = getGeneratorId();
-        int numberOfGenerators = getNumberOfGenerators();
 
         // Create an ID for the task
         String taskId = getNextTaskId();
```
In this tutorial, we only use a single Task Generator in our set up. Hence, we can remove the ID of the generator and the number of generators that are used.

After we know where we can separate the expected answer from the input data for the system, we can send the input data to the system and the expected answer to the evaluation storage.
```diff
         // Create an ID for the task
         String taskId = getNextTaskId();

         // Create the task and the expected answer
+        byte[] taskData = line.substring(0, pos).getBytes(StandardCharsets.UTF_8);
+        byte[] expectedAnswerData = line.substring(pos + 1).getBytes(StandardCharsets.UTF_8);
-        byte[] taskData = ...;
-        byte[] expectedAnswerData = ...;

         // Send the task to the system (and store the timestamp)
         long timestamp = System.currentTimeMillis();
         sendTaskToSystemAdapter(taskId, taskData);

         // Send the expected answer to the evaluation store
         sendTaskToEvalStorage(taskId, timestamp, expectedAnswerData);
     }
```

TODO add link to the final Task Generator file.

### Docker Image

TODO define Dockerfile

<span id="task-gen-image-used">[↪️](#task-gen-image-defined)</span>.

## Evaluation Module

<p align="center">
  <img style="max-width:500px;height:auto;width:auto;" src="/images/Components-diagram-wine-benchmark-Eval-Module.svg" />
</p>

### Java Class

We start our implementation by creating a new class named `org.example.hobbit.wine.benchmark.EvaluationModule` and copy the parts of the [benchmark tutorial](benchmark_integration.html) for the Evaluation Module into this new class. After that, our class should look as follows:
```java
package org.example.hobbit.wine.benchmark;

import java.io.IOException;

import org.apache.jena.rdf.model.Model;
import org.apache.jena.rdf.model.Resource;
import org.apache.jena.vocabulary.RDF;
import org.hobbit.core.components.AbstractEvaluationModule;
import org.hobbit.vocab.HOBBIT;

public class EvaluationModule extends AbstractEvaluationModule {

    @Override
    public void init() throws Exception {
        // Always init the super class first!
        super.init();
        
        // Your initialization code comes here...
    }

    @Override
    protected void evaluateResponse(byte[] expectedData, byte[] receivedData, long taskSentTimestamp,
            long responseReceivedTimestamp) throws Exception {
        // evaluate the given response and store the result, e.g., increment internal counters
        ...
    }
    
    @Override
    protected Model summarizeEvaluation() throws Exception {
        // All tasks/responsens have been evaluated. Summarize the results,
        // write them into a Jena model and send it to the benchmark controller.
        Model model = createDefaultModel();
        Resource experimentResource = model.getResource(experimentUri);
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
The Evaluation Module has two phases. First, it compares single responses of the benchmarked system and the expected responses sent by the Task Generator. This is done within the `evaluateResponse` method. In the second phase, it summarizes the evaluation results and returns the summary as RDF model. This phase is implemented within the `summarizeEvaluation` method.

We implement the first phase, i.e., the comparison of single responses as follows:
```diff
import java.io.IOException;
+import java.nio.charset.StandardCharsets;
+import java.util.ArrayList;
+import java.util.List;

import org.apache.jena.rdf.model.Model;
import org.apache.jena.rdf.model.Resource;
import org.apache.jena.vocabulary.RDF;
import org.hobbit.core.components.AbstractEvaluationModule;
import org.hobbit.vocab.HOBBIT;
+import org.slf4j.Logger;
+import org.slf4j.LoggerFactory;

public class EvaluationModule extends AbstractEvaluationModule {
+
+    private static final Logger LOGGER = LoggerFactory.getLogger(EvaluationModule.class);
+
+    protected List<Double> squaredErrors = new ArrayList<>();
+    protected List<Long> runtimes = new ArrayList<>();
+    protected int faultyResponses = 0;

     @Override
     public void init() throws Exception {
         // Always init the super class first!
         super.init();
-        
-        // Your initialization code comes here...
     }

     @Override
     protected void evaluateResponse(byte[] expectedData, byte[] receivedData, long taskSentTimestamp,
             long responseReceivedTimestamp) throws Exception {
-        // evaluate the given response and store the result, e.g., increment internal counters
-        ...
+        // read data as string
+        double expectedAnswer;
+        double receivedAnswer;
+        String expectedAnswerString = null;
+        String receivedAnswerString = null;
+        try {
+            expectedAnswerString = new String(expectedData, StandardCharsets.UTF_8);
+            expectedAnswer = Double.parseDouble(expectedAnswerString);
+        } catch (Exception e) {
+            LOGGER.error("Couldn't parse expected answer \"" + expectedAnswerString + "\". It will be ignored.", e);
+            return;
+        }
+        try {
+            receivedAnswerString = new String(receivedData, StandardCharsets.UTF_8);
+            receivedAnswer = Double.parseDouble(receivedAnswerString);
+        } catch (Exception e) {
+            LOGGER.error("Couldn't parse expected answer \"" + expectedAnswerString + "\". It will be ignored.", e);
+            ++faultyResponses;
+            return;
+        }
+        squaredErrors.add(Math.pow(expectedAnswer - receivedAnswer, 2));
+        runtimes.add(responseReceivedTimestamp - taskSentTimestamp);
+    }
```
First, we transform the expected and received data into Strings. After that, we parse the numbers that we expect in these Strings. After that, we calculate the squared error of the parsed numbers and add it to an internal list of squared errors that we define as class attribute. Similarly, we store the runtime of the task, i.e., the difference between the timestamp at which the task was sent and the timestamp at which the answer was received. In case there the expected answer cannot be parsed, an error is printed and the received data is ignored. If the answer of the benchmarked system cannot be parsed, the counter for faulty responses is increased.

In the second phase, i.e., in the `summarizeEvaluation` method, we calculated the arithmetic average and standard deviation of the squared error and runtime values, respectively.
```diff
     @Override
     protected Model summarizeEvaluation() throws Exception {
-        // All tasks/responsens have been evaluated. Summarize the results,
-        // write them into a Jena model and send it to the benchmark controller.
+        // Calculate averages and standard deviations
+        double avgError = squaredErrors.stream().mapToDouble(l -> l).average().orElse(Double.NaN);
+        double stdDevError = Math
+                .sqrt(squaredErrors.stream().mapToDouble(l -> Math.pow(avgError - l, 2)).average().orElse(Double.NaN));
+        double avgRuntime = runtimes.stream().mapToLong(l -> l).average().orElse(Double.NaN);
+        double stdDevRuntime = Math
+                .sqrt(runtimes.stream().mapToDouble(l -> Math.pow(avgRuntime - l, 2)).average().orElse(Double.NaN));
+
+        // Create RDF model
         Model model = createDefaultModel();
         Resource experimentResource = model.getResource(experimentUri);
         model.add(experimentResource, RDF.type, HOBBIT.Experiment);
-        ...
+
+        model.addLiteral(experimentResource, model.createProperty("http://example.org/wine-example/mse"), avgError);
+        model.addLiteral(experimentResource, model.createProperty("http://example.org/wine-example/avgRuntime"), avgRuntime);
+        model.addLiteral(experimentResource, model.createProperty("http://example.org/wine-example/stdDevRuntime"), stdDevRuntime);
+        model.addLiteral(experimentResource, model.createProperty("http://example.org/wine-example/faultyResponses"), faultyResponses);
+        model.addLiteral(experimentResource, model.createProperty("http://example.org/wine-example/testDataSize"), faultyResponses + squaredErrors.size());
+
         return model;
     }

     @Override
     public void close() throws IOException {
-        // Free the resources you requested here
-        ...
-        
         // Always close the super class after yours!
         super.close();
     }
```

<span id="test-size-param-iri-defined">[📝](#test-size-param-iri-used)</span>

<span id="kpi-iris-used">[↪️](#kpi-iris-defined)</span>
The calculated values are added to the an RDF knowledge graph using the IRIs that we defined above (TODO Link). Finally, we remove the unnecessary lines from the `close` method.

### Docker Image

TODO define Dockerfile

<span id="eval-module-image-used">[↪️](#eval-module-image-defined)</span>.

## Benchmark Controller

<p align="center">
  <img src="/images/Components-diagram-wine-benchmark-Benchmark-Controller.svg" />
</p>

### Java Class

<span id="data-gen-image-used-bc">[↪️](#data-gen-image-defined)</span>.
<span id="task-gen-image-used-bc">[↪️](#task-gen-image-defined)</span>.
<span id="eval-module-image-used-bc">[↪️](#eval-module-image-defined)</span>.

<span id="file-name-prop-defined">[📝](#file-name-prop-used)</span>

<span id="dataset-param-iri-defined">[📝](#dataset-param-iri-used)</span>
<span id="seed-param-iri-defined">[📝](#seed-param-iri-used)</span>


### Docker Image

TODO define Dockerfile

TODO build command of the image <span id="controller-image-used">[↪️](#controller-image-defined)</span>.

docker build -f baseline-system.Dockerfile -t hobbit-wine-example-baseline-system .
docker build -f benchmark-controller.Dockerfile -t hobbit-wine-example-controller .
docker build -f data-generator.Dockerfile -t hobbit-wine-example-data-generator .
docker build -f evaluation-module.Dockerfile -t hobbit-wine-example-task-generator .
docker build -f task-generator.Dockerfile -t hobbit-wine-example-evaluation-module .

## System

TODO

## Execution on HOBBIT

TODO

### Local HOBBIT instance

TODO

### Public HOBBIT instance

TODO
