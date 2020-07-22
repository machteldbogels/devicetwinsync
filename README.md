# Duplicating device twins from IoT Hub to Cosmos DB using Azure Functions

### Background
IoT Hub is a service on Azure which can be used as a platform to ingest and manage all the communication from and to your devices. For every device that is connected to your IoT Hub, a Device Twin is created. Device Twins are JSON documents that store device state information including metadata, configurations, and conditions.  

### Scenario
Since the IoT Hub imposes throttling limits on reading the device twins, one option could be to replicate the Device Twins to an external database, in this example a Cosmos DB is used. The advantage of this solution is that you can scale up the external storage without having to scale the whole instance of IoT Hub. In this way you won't run into throttling limits when you are building an application on top of this database with frequent queries because you can scale dynamically based on your requirements. 

### Architecture
![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/architecture.png?raw=true)

Once a change has occurred to an existing Device Twin in IoT Hub, the message routing feature for twin change events in IoT Hub sends an event to an Event Hub, which then triggers an Azure function to update the twin in a Cosmos DB container. This is also done for lifecycle events, sending an event to Event Hub when a device is deleted or created, triggering another Azure function to replicate that operation on the device twin in the Cosmos DB container.

### Configuring IoT Hub to route Device Twin changes and Lifecycle events towards Event Hubs
*add picture of configuration*

The schema of the events consist of properties as well as the message body:

```
*add schema example*
```

### Creating Event Hub-triggered Azure Functions
*add picture from VS code/portal?*

### Creating or Deleting a Device Twin in Cosmos DB using the LifecycleUpdates function
The code for this function can be found [here](https://github.com/machteldbogels/devicetwinsync/blob/master/LifecycleUpdates/index.js)

*The environment variables that point to the endpoint and key of your Cosmos DB instance can be stored either locally or within an Azure Key Vault for example.*


### Updating a Device Twin in Cosmos DB using the TwinChanges function
The code for this function can be found [here](https://github.com/machteldbogels/devicetwinsync/blob/master/TwinChanges/index.js)


By default only the message body is written to Cosmos DB. Since we also want to know the DeviceId as well as the Event Type, two properties from the event are added to the message body:

```
        // Update event message with id, deviceid and eventType
        var updatedDeviceDetails = myEventHubMessage[i];      
        updatedDeviceDetails["deviceid"] = deviceid;
        updatedDeviceDetails["eventType"] = eventType;
```

At the moment, the Cosmos DB SQL API does not support partial updates, so therefore the db has to be queried in order to retrieve the existing




