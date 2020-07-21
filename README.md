# Duplicating device twins from IoT Hub to Cosmos DB using Azure Functions

## Scenario
**IoT Hub** is a PaaS on Azure which can be used to ingest and manage all the communication from and to your devices. For every device that is connected to your IoT Hub, a **device twin** is created. Device twins are JSON documents that store device state information including metadata, configurations, and conditions.  


## Architecture
![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/architecture.png?raw=true)


## Advantages of storing your device twins in Cosmos DB
* Auto-scaling of Request Units
* No throttling limits on reads/writes
* More freedom in building applications on top of your device twins

## Configuring IoT Hub to route device twin changes to Event Hubs


## Creating Event Hub-triggered Azure Functions


## Retrieving existing device twins from Cosmos DB and update where necessary




*Since the IoT Hub imposes throttling limits on reading the device twins, we replicate the twins to a Cosmos DB container. Once a device twin is updated in IoT Hub, the message routing feature for twin change events in IoT Hub sends an event to an Event Hub, which then triggers this function to update the twin in a Cosmos DB container. This function is fed by the same Event Hub with twin change events as the above Downlink function.*

*Similar to the TwinReplication function, the message routing feature in IoT Hub is used for lifecycle events, sending an event to Event Hub when a device is deleted or created, triggering this function to replicate that operation on the device twin in the Cosmos DB container.*


