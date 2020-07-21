# Duplicating device twins from IoT Hub to Cosmos DB using Azure Functions

## Scenario
**IoT Hub** is a PaaS on Azure which can be used to ingest and manage all the communication from and to your devices. For every device that is connected to your IoT Hub, a **device twin** is created. Device twins are JSON documents that store device state information including metadata, configurations, and conditions.  


## Architecture
![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/architecture.png?raw=true)


## Context
Since the IoT Hub imposes throttling limits on reading the device twins, we replicate the twins to a Cosmos DB container. Once a device twin is updated in IoT Hub, the message routing feature for twin change events in IoT Hub sends an event to an Event Hub, which then triggers this function to update the twin in a Cosmos DB container. This is also done for lifecycle events, sending an event to Event Hub when a device is deleted or created, triggering this function to replicate that operation on the device twin in the Cosmos DB container.*

**Advantages of storing Device Twins in Cosmos DB**
* Auto-scaling of Request Units
* No throttling limits on reads/writes
* More freedom in building applications on top of your device twins

## Configuring IoT Hub to route Device Twin changes towards Event Hubs
*add picture of configuration*


## Creating Event Hub-triggered Azure Functions
*add picture from VS code/portal?*

## Creating or Deleting a Device Twin in Cosmos DB using the LifecycleUpdates function
The code for this function can be found [here][https://github.com/machteldbogels/devicetwinsync/blob/master/LifecycleUpdates/index.js]

*The environment variables that point to the endpoint and key of your Cosmos DB instance can be stored either locally or within an Azure Key Vault for example.*


## Updating a Device Twin in Cosmos DB using the TwinChanges function
The code for this function can be found [here][https://github.com/machteldbogels/devicetwinsync/blob/master/TwinChanges/index.js]

The schema of the event consists of properties as well as the message body:

*add schema example*

**Partial Updates**
At the moment, the Cosmos DB SQL API does not support partial updates, so therefore the db has to be queried in order to retrieve the existing




