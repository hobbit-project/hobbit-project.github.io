---
title: Integrating Your Own Code
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: system_integration.html
folder: docs
---

This tutorial shows how a system can be prepared to be benchmarked within the HOBBIT platform. The tutorial mainly covers all necessary steps including the development of a System Adapter in Java. Note that the adapter can be implemented in other languages as well. However, for Java we are offering a library which eases the implementation. For other languages, you may want to take a look into the [System API](/system_integration_api.html).

A system that should be benchmark within the HOBBIT platform needs to fulfill the following requirements:
1. Be encapsulated in a Docker image
1. Implement the API of the benchmark. This is something that can not be covered by this tutorial since it is depending on the benchmark.
1. Implement the API of the platform. This mainly includes
    1. Sending a message to the platform controller that your system is ready to be benchmarked your container has started,
    1. Sending requests and handling the responses correctly if your system does not comprise only one container but needs more than one.

For the requirements 2 and 3, a System Adapter can be developed which takes care of that. So your system does not have to be changed just to be benchmarked. In this tutorial, we will present two scenarios for that. First, the system adapter will be implemented as a wrapper which runs together with the benchmarked system in the same Docker container. The second approach is a separated System Adapter which runs in another Docker container in parallel to the benchmarked system.

## 1. Set up the System Adapter project

For this tutorial, we will use [Apache Maven](http://maven.apache.org/) to manage the dependencies and the build process of our project. However, you can use other tools if you prefer them.

We start by creating a Maven project called "my-system-adapter" with our preferred IDE. We open the projects pom file and adapt it by
1. Adding the AKSW repository for being able to include our libraries,
1. Adding the [`hobbit.core` library](https://github.com/hobbit-project/core) as dependency to be able to use the predefined classes,
1. Adding the slf4j-log4j binding since our classes will use [slf4j loggers](https://www.slf4j.org/index.html) for logging messages which won't work without a binding. Of course, you can use different bindings if you prefer them and
1. Configuring the [shade plugin](https://maven.apache.org/plugins/maven-shade-plugin/) which we will be using to create our final jar file containing all the classes and files necessary for the system adapter to run.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>my-system-adapter</artifactId>
    <version>1.0.0</version>

    <repositories>
        <repository>
            <id>maven.aksw.internal</id>
            <name>University Leipzig, AKSW Maven2 Repository</name>
            <url>http://maven.aksw.org/repository/internal</url>
        </repository>
        <repository>
            <id>maven.aksw.snapshots</id>
            <name>University Leipzig, AKSW Maven2 Repository</name>
            <url>http://maven.aksw.org/repository/snapshots</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.hobbit</groupId>
            <artifactId>core</artifactId>
            <version>1.0.11</version>
        </dependency>
        <!-- Add a slf4j log binding here -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.15</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
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
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer" />
                        <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer" />
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

In this project, you should find a `src/main/java` directory (your IDE should typically create it). In this directory (or a subdirectory if you want to use packages), we create the main class of our System Adapter in the next step.

## 2. Writing the System Adapter

The System Adapter comprises one main class that extends the [`org.hobbit.core.components.AbstractSystemAdapter` class](https://github.com/hobbit-project/core/blob/master/src/main/java/org/hobbit/core/components/AbstractSystemAdapter.java). We will build our System Adapter image to execute this class, in one of the later steps. However, our System Adapter class (let's call it `MySystemAdapter` in the `org.example` package) should look similar to the following piece of code:
```java
package org.example;

import java.io.IOException;
import org.hobbit.core.components.AbstractSystemAdapter;

public class MySystemAdapter extends AbstractSystemAdapter {

    @Override
    public void init() throws Exception {
        super.init();

        // Your initialization code comes here...

        // You can access the RDF model this.systemParamModel 
        // to retrieve meta data defined in your system.ttl file
    }

    @Override
    public void receiveGeneratedData(byte[] data) {
        // handle the incoming data as described in the benchmark description
    }

    @Override
    public void receiveGeneratedTask(String taskId, byte[] data) {
        // handle the incoming task and create a result
        byte[] result = ...;

        // Send the result to the evaluation storage
        sendResultToEvalStorage(taskId, result);
    }

    @Override
    public void close() throws IOException {
        // Free the resources you requested here

        // Always close the super class after yours!
        super.close();
    }

}
```

We will go through the different methods to clarify their meaning:
1. `init` is the method in which you should initialize your system. The first method that is called should be the `super.init()` method to make sure that all necessary resources are available. Note that after leaving the `init` method of your class, the benchmarking is allowed to start and your system should be ready to receive tasks.
1. `receiveGeneratedData` is depending on the benchmark. Some benchmarks will send data that should be stored or processed before the benchmarking starts or during the benchmarking. You will need to check the benchmarks API that you want to use to figure out whether the benchmark will make use of this feature and which format the data will have.
1. `receiveGeneratedTask` is the method which is called if your system receives a task that should be fulfilled. Based on the data that you receive, your system should generate the result the benchmark is assuming and send it back using the `sendResultToEvalStorage` method. Please check the benchmarks API for the format that the data of the task will have and which format the (serialized) result should have.
1. `close` is the method which will be called at the end after the benchmarking is done and your system is asked to shut down. Make sure that `super.close()` is the last method called this method.

Note that there are additional methods which could be overriden by your class if you want to make use of them. An overview of these methods can be found at [System API](/system_integration_api.html).

One of possibilities to manage communication with the system is to write a system adapter as a wrapper. Other options are described at [System API](/system_integration_api.html).

### 2.a. The System Adapter as wrapper

This solution will assume that your system is located within the same Docker container and your System Adapter is simply wrapping this system. This is a good approach if a) your project is implemented in Java and you want to call the systems class or method directly from the System Adapter or b) you think that it is easier to communicate within the Docker Container than with another container. For this tutorial, we will assume the first case.

To enable the System Adapter to work with your classes, you need to add them to the projects class path by adapting the pom file. There are several ways to do that and you are free to choose whatever way is best for you. For this tutorial, we will assume that 
* you are not familiar with Maven and 
* that you have your system available as a single, large `system.jar` file.

We will add your system using a [local repository](http://mark.koli.ch/maven-add-local-jar-dependency-to-classpath). Create a folder inside your project named `repository`. Choose a `groupId`, `artifactId` and a `version` for your system to represent it in Maven. For our example, we will use `org.example`, `my-system` and `1.0.0`. Execute the following command to install your system into the local repository and enable Maven to include it into your project.
```
mvn install:install-file \
  -Dfile=system.jar \
  -DgroupId=org.example \
  -DartifactId=my-system \
  -Dversion=1.0.0 \
  -Dpackaging=jar \
  -DlocalRepositoryPath=repository
```
Note that you may have to adapt the paths to the `system.jar` file or the `repository` directory.

After creating the local repository, the System Adapter projects pom file needs to be updated by
1. Adding the `repository` directory as local repository and
1. Adding the system to the dependencies

```xml
    <repositories>
        <repository>
            <id>local repository</id>
            <url>file://${project.basedir}/repository</url>
        </repository>
        ...
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>my-system</artifactId>
            <version>1.0.0</version>
        </dependency>
    ...
    </dependencies>
```

Now, the systems classes can be used by the System Adapter in the four methods described above as normal Java classes.

## 3. Create and push the Docker image

### Compiling the System Adapter project

For finally creating and pushing the Docker image to gitlab, the Maven project needs to be compiled. The following command will delete old compilings, compile the classes of the project and create a large jar file containing the classes defined in the project and the dependencies defined in the pom file.

```
mvn clean package
```

After that, the `target` folder of your project should contain a jar file with the name of your project. At the beginning, we named the project `my-system-adapter` and defined its version as `1.0.0` so we should find a file `my-system-adapter-1.0.0.jar` (by default Maven uses `<artifactId>-<version>`).

### Creating the Docker image

Create a `Dockerfile` in your System Adapter project with the following content:

```dockerfile
FROM java

ADD target/my-system-adapter-1.0.0.jar /system/my-system-adapter.jar

WORKDIR /system

CMD java -cp my-system-adapter.jar org.hobbit.core.run.ComponentStarter org.example.MySystemAdapter
```

The file defines that Docker will reuse the already existing `java` image which contains a JVM. It will add our jar file to a `system` directory. In this directory, the last line will be executed. This line executes the `ComponentStarter` class which will call the methods of our class.

Since we want to upload the image to the HOBBIT gitlab, the name of our image already has to contain the address of this gitlab as well as our gitlab user name and the name of the gitlab project. Let's assume that the name of our user is `maxpower` and that we own a project named `mysystemadapter` to which we will upload the image. The following command will create the Docker image and give it the name which is necessary to upload it in the next step.

```
docker build -t git.project-hobbit.eu:4567/maxpower/mysystemadapter .
```

Note that the letters should be lowercased even if the real user name and project name include upper case letters. Additionally, it might create problems to use names with non alphanumeric characters.

### Pushing the image

After the successful building of the image, it can be pushed to the git project.

```
docker login git.project-hobbit.eu:4567
docker push git.project-hobbit.eu:4567/maxpower/mysystemadapter
```

It is possible to upload multiple images. This can be necessary if the system is in a different container than the System Adapter. There are two ways to upload the additional images. Either you create a gitlab project for each image (note that every user has a maximum number of projects) or you upload several images to a single gitlab project. For that, it is necessary to name the images in a certain way. Assume that our users has created a gitlab project with the name `mysystem`. We could upload the system and system adapter images if we give them names like

```
docker push git.project-hobbit.eu:4567/maxpower/mysystem/system
docker push git.project-hobbit.eu:4567/maxpower/mysystem/adapter
```

Note that both names contain the complete project name and end with a different name to identify the image.

## 4. Writing the system.ttl file

The system meta data file comprises meta data about the uploaded system that is needed by the Hobbit platform.
The file contains the meta data as [RDF triples](https://www.w3.org/2001/sw/wiki/RDF) in the
[Turtle format](https://www.w3.org/TR/turtle/) and needs to be uploaded to the git instance of the platform, e.g., into the root directory of the project which hosts the system Docker image. This article gives guidance how this meta
data file can be created for a system called `"MyOwnSystem"`.

### Simple file

The simplest file defines only those information that are necessary to get the system running. First, we have to collect the necessary data:
* The system needs a unique identifier, i.e., a URI. In our example, we choose `http://www.example.org/exampleSystem/MyOwnSystem`.
* The system needs a name (`"MyOwnSystem"`) and a short description (`"This is my own example system..."`).
* The name of the uploaded docker image is needed (`"git.project-hobbit.eu:4567/maxpower/mysystemadapter"`).
* The URI of the benchmark API, the system implements (`http://benchmark.org/MyNewBenchmark/BenchmarkApi`, should be provided by the benchmark description page).

Our example file has the following content

```turtle
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .

<http://www.example.org/exampleSystem/MyOwnSystem> a  hobbit:SystemInstance;
	rdfs:label	"MyOwnSystem"@en;
	rdfs:comment	"This is my own system defined in a simple way"@en;
	hobbit:imageName "git.project-hobbit.eu:4567/maxpower/mysystemadapter";
	hobbit:implementsAPI <http://benchmark.org/MyNewBenchmark/BenchmarkApi> .
```
