# Duplicating your device twins from IoT Hub to Cosmos DB using Azure Functions

## Scenario
Let's say you want to use **IoT Hub** to ingest and manage all the communication from and to your devices. For every device that is connected to your IoT Hub, a **device twin** is created. Device twins are JSON documents that store device state information including metadata, configurations, and conditions.  

## Advantages of storing your device twins in Cosmos DB
* Auto-scaling of Request Units
* No throttling limits on reads/writes
* More freedom in building applications on top of your device twins


*Since the IoT Hub imposes throttling limits on reading the device twins, we replicate the twins to a Cosmos DB container. Once a device twin is updated in IoT Hub, the message routing feature for twin change events in IoT Hub sends an event to an Event Hub, which then triggers this function to update the twin in a Cosmos DB container. This function is fed by the same Event Hub with twin change events as the above Downlink function.*

*Similar to the TwinReplication function, the message routing feature in IoT Hub is used for lifecycle events, sending an event to Event Hub when a device is deleted or created, triggering this function to replicate that operation on the device twin in the Cosmos DB container.*
