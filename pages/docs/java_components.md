---
title: Java Components
keywords: HOBBIT Documentation
sidebar: main_sidebar
toc: false
permalink: java_components.html
folder: docs
---

This tutorial explains how a component (of a benchmark or a system adapter) can be created using the interfaces and classes from the [core library](https://github.com/hobbit-project/core) of the HOBBIT platform.
This article describes the general development.
It is assumed that the component is programmed in Java using Maven.
After reading it you might want to take a look at the [development of a system adapter](Develop-a-system-adapter-in-Java) or of a [benchmark component](Develop-a-benchmark-component-in-Java).

## Project creation
At first, a Java Maven project has to be created. In the pom file of this project, the following lines should be added.
```xml
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
		...
	</repositories>

	<dependencies>
		<dependency>
			<groupId>org.hobbit</groupId>
			<artifactId>core</artifactId>
			<version>1.0.5</version>
		</dependency>
		<!-- Add a slf4j log binding here -->
		...
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
						<transformer
							implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
							<manifestEntries>
								<X-Compile-Source-JDK>${maven.compile.source}</X-Compile-Source-JDK>
								<X-Compile-Target-JDK>${maven.compile.target}</X-Compile-Target-JDK>
							</manifestEntries>
						</transformer>
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
			...
		</plugins>
		...
	</build>
```

The lines add the AKSW repository containing the core library. Additionally, the shaded plugin is added which is used to create a jar file that contains the code of the project as well as all its dependencies.

## Components (in general)

### The Component interface

A single benchmark comprises several components (as described in the [platform overview](/overview.html)) and a system adapter can be seen as a single component as well. All these components should implement the `org.hobbit.core.components.Component` interface. It defines the following methods.
```java
	public void init() throws Exception;

	public void run() throws Exception;

	public void close() throws IOException;
```
The typical work flow of these components is straight forward and can be seen by inspecting the `ComponentStarter ` class that is used to execute the components.

```java
public class ComponentStarter {

	public static void main(String[] args) {
	    ...
		Component component = null;
		boolean success = true;
		try {
			component = createComponentInstance(args[0]);
			// initialize the component
			component.init();
			// run the component
			component.run();
		} catch (Throwable t) {
			LOGGER.error("Exception while executing component. Exiting with error code.", t);
			success = false;
		} finally {
			IOUtils.closeQuietly(component);
		}

		if (!success) {
			System.exit(ERROR_EXIT_CODE);
		}
    }

	...
}
```
After its creation, the component instance is initialized. Then, its `run` method is called before its `close` method is called inside the `finally` block.

Apart from this work flow, the example shows the following important features of a component
* The component can be instantiated by using only its class name. Thus, it has to offer a constructor that needs no parameters.
* If a component encounters a severe error (in the code above such an error is indicated by an uncatched `Throwable` or `Exception`) the process is exited with an error code. However, some components might have more than one thread. If another thread than the main thread needs to be able to close the complete process by calling `System.exit(ERROR_EXIT_CODE)` this thread should call the `close` method of the component before to make sure that connections that a component might have end up in an undefined state.

The implementation of a typical component might extend an already existing abstract component implementation. However, in most cases it looks like the following:

```java
public class ExampleComponent extends SomeAbstractExampleComponent {

	public ExampleComponent() {
		/* In most cases the constructor is empty. It is better to execute
		 * the needed commands for the class initialization inside the
		 * init method since the super classes should be initialized before
		 * your class is initializing itself (and might use functionalities
		 * of a super class).
		 */
	}

    @Override
	public void init() throws Exception {
        // Always init the super class first!
        super.init();

		// Your initialization code comes here
		...
    }

    @Override
	public void run() throws Exception {
		// Your component execution code comes here
		...
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

### Using the command queue

The abstract classes offer an easy usage of the [command queue](/command_queue.html). In most cases, a direct usage of the command queue shouldn't be necessary. However, the abstract classes offer the following methods to send a message to the command queue:

```java
protected void sendToCmdQueue(byte command) throws IOException;

protected void sendToCmdQueue(byte command, byte data[]) throws IOException;

protected void sendToCmdQueue(byte command, byte data[], BasicProperties props) throws IOException;
```

It can be seen that the methods take the byte of the command that should be send as well as the optional data that can be added to the message. The last method accepts a `BasicProperties` instance with which the communication can be configured in more detail. However, in nearly all cases these properties are not needed.

Commands that are received by a component can be handled by overriding the `receiveCommand` method.
```java
@Override
public void receiveCommand(byte command, byte[] data) {
    // handle command

    // call method of super class!
    super.receiveCommand(command, data);
}
```
It is highly recommended to call the super class at the end of the method to make sure that the abstract classes are working as expected.

### Creating and stopping containers

As described in the [general hints on components](https://github.com/hobbit-project/platform/wiki/General-hints-on-components#creating-containers) the platform offers the creation of additional Docker containers. This functionality is used by the benchmark controller to create the other benchmark components. In the same way, it can be used by the system adapter to create multiple instances of the system or by any other component to create containers if they are required. The abstract classes for the components ease this by offering the following methods.

```java
protected String createContainer(String imageName, String[] envVariables);

protected void stopContainer(String containerName);
```
Note that the `createContainer` method returns the name of the created container that serves as the host name in the virtual network.

## Execution
For the creation of Docker images docker files have to be created. Let's assume that the example project has been compiled and the shaded plugin generated the file `target/example-1.0.0.jar`.
```
FROM java

ADD target/example-1.0.0.jar /example/example-1.0.0.jar

WORKDIR /example

CMD java -cp example.jar org.hobbit.core.run.ComponentStarter org.example.ExampleDataGenerator
```
This docker file tells Docker
* that the image extends a container in which a Java program can be run,
* that the generated jar file should be copied to the `example` directory,
* that this directory is our working directory and
* that it should execute the `ComponentStarter` that will load our component (in this example it is the `ExampleDataGenerator`)

## Logging
The offered abstract classes use the [slf4j](https://www.slf4j.org/) logging library with a [binding to log4j](https://www.slf4j.org/manual.html#swapping). However, using the abstract classes without additional configuration of the logging will lead to the following Warnings instead of log messages:
```
log4j:WARN No appenders could be found for logger (org.hobbit.benchmark.platform.Temp).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
```

An easy way to configure a log4j appender is to add a `log4j.properties` file to the project (e.g., to `src/main/resources` in a maven project) that could have the following content:
```
# Direct log messages to stdout
log4j.rootLogger=INFO,stdout

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - <%m>%n
```
This file defines a simple log appender that writes all log messages which have at least the log level `INFO` to the console. For more information on configuring log4j, please have a look at its documentation at http://logging.apache.org/log4j/1.2/index.html

If a different logging should be used and a bridge from slf4j to this logging is available, the log4j binding has to be excluded in your projects pom file using.
```
	<dependencies>
		<!-- Hobbit core -->
		<dependency>
			<groupId>org.hobbit</groupId>
			<artifactId>core</artifactId>
			<version>1.0.0</version>
			<exclusions>
				<exclusion>
					<artifactId>slf4j-log4j12</artifactId>
					<groupId>org.slf4j</groupId>
				</exclusion>
			</exclusions>
		</dependency>
		...
	</dependencies>
```
