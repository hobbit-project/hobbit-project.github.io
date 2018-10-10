---
title: Transferring Data
keywords: HOBBIT Documentation
sidebar: main_sidebar
permalink: transferring_data.html
folder: docs
---

## Transferring files
The core library offers a simple way to transfer data (as files) from one container to another using RabbitMQ. The two classes `SimpleFileSender` and `SimpleFileReceiver` can be used to send files which will be stored in a given output folder.

### Sending data
The sender requires the name of the queue which should be used to send the data and a `RabbitQueueFactory` instance. The `AbstractComponent` class and all its extensions already offer two factories for incoming and outgoing connections (`this.incomingDataQueueFactory ` and `this.outgoingDataQueuefactory`).

```java
    // define a queue name, e.g., read it from the environment
    String queueName = System.getenv().get(Constants.DATA_QUEUE_NAME_KEY);

    // create the sender
    SimpleFileSender sender = SimpleFileSender.create(this.outgoingDataQueuefactory, queueName);

    InputStream is = null;
    try {
        // create input stream, e.g., by opening a file
        is = ...
        // send data
        sender.streamData(is, someFileName);
    } catch (Exception e) {
        // handle exception
    } finally {
        IOUtils.closeQuietly(is);
    }

    // close the sender
    IOUtils.closeQuietly(sender);
```

It can be seen that the sender has to give a name to all given `InputStream` objects. This name will be used by the receiver as file name to store the data.

### Receiving data
The receiver requires the same data as the sender.

```java
    SimpleFileReceiver receiver = SimpleFileReceiver.create(this.incomingDataQueueFactory, queueName);
    String[] receivedFiles = receiver.receiveData(outputDir);
```

The `receiveData(outputDir)` returns a list of receiver (relative) file names of the files that have been written to the given output directory. Note that the method blocks until the receiver is terminated by calling its `terminate()` method.

After the `receiveData` method finished, the receiver can be asked for errors that occured by calling the `receiver.getErrorCount()` method. If the count is `>0` the received data might be faulty.

## Integration into the data generator
For integrating the file transferring into the platform the `DockerBasedMimickingAlg` class can be used. It takes a Docker image name of a mimicking algorithm and manages the generation of the data. It requires a `PlatformConnector` which is already implemented by the `AbstractDataGenerator` class.

```java
    DockerBasedMimickingAlg alg = new DockerBasedMimickingAlg(this, dockerImageName);
    alg.generateData(someOutputDirectory, envVariables);
```

The `generateData` method takes an output directory to which the data will be written and a String array containing environment variables of the form `"key1=value1", "key2=value2"` that will be given to the created mimicking algorithm container. The method will block until all files are received. If an error occurs, an exception is thrown.
The method implements the following workflow:

1. Create a queue with a random name to avoid overlapping with a another queue
2. Create a `SimpleFileReceiver` with the created queue
3. Create a mimicking algorithm container submitting the given environmental variables together with an additional variable which has a key defined by `Constants.DATA_QUEUE_NAME_KEY` and contains the name of the created queue. It is assumed that the mimicking algorithm will send its data using this queue.
4. The data will be received until the mimicking algorithm container terminates (The methods of the given `PlatformConnector` are used to receive this event). This event will be used to terminate the internal `SimpleFileReceiver`, close the queue and leave the method.
