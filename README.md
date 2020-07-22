# Duplicating device twins from IoT Hub to Cosmos DB using Azure Functions

### Background
IoT Hub is a service on Azure which can be used as a platform to ingest and manage all the communication from and to your devices. For every device that is connected to your IoT Hub, a Device Twin is created. Device Twins are JSON documents that store device state information including metadata, configurations, and conditions.  

### Scenario
Each IoT Hub instance imposes throttling limits based on the service tier you are using. This implies that there is a maximum number of reads and writes on the database that stores your Device Twins, for which the documentation can be found [here](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-quotas-throttling). To work around that, an option would be to replicate the Device Twins from IoT Hub to an external database which can store JSON files such as Cosmos DB. The advantage of this solution is that, in order to overcome the throttling limits, you only need to scale up the external storage rather than the whole instance of IoT Hub (which includes many services besides storage) In this way you won't run into performance issues when building an application that frequently reads from the database because you can scale dynamically based on your requirements. 

### Architecture
![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/architecture.png?raw=true)

Whenever a Device Twin is created in IoT Hub, the message routing feature for Lifecycle 
Once a change has occurred to an existing Device Twin in IoT Hub, the message routing feature for twin change events in IoT Hub sends an event to an Event Hub, which then triggers an Azure function to update the twin in Cosmos DB. This is also done for lifecycle events, sending an event to Event Hub when a device is deleted or created, triggering another Azure function to replicate that operation on the device twin in the Cosmos DB container.

### Configuring IoT Hub to route Device Twin changes and Lifecycle events towards Event Hubs
To set up the message routing to send events to an Event Hub whenever changes occur to your Device Twins, go to `Message routing` under the *Messaging* tab and select `Add`:

![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/messagerouting1.png?raw=true)

There you would need to `Add an endpoint` which points to the Event Hub you would like to use for this scenario, and make sure you select `Device Lifecycle Events` from the dropdown list: 

![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/messagerouting2.png?raw=true)

Repeat this process to create another Message routing for the `Device Twin Changes Events` which points to another Event Hub instance (which can be within the same Event Hub Namespace as the previous one).


The schema of the events that are being sent to these Event Hubs consists of some properties as well as the message body containing the changes that occurred, for example:

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




