# Replicating Device Twins outside your IoT Hub Gateway
Using IoT Hub, Cosmos DB, Event Hub and Azure Functions


### Context
Cloud gateway services provide the ability to ingest and manage all communication from and to your devices. Within those services, typically every physical device has a digital representation inside a registry where information is stored about the device, such as its device id, connection status and other properties that describe its latest known information. This device data is relevant to end users as it shows important device health information which in many cases needs to be handled upon. Therefore, it is important to have the freedom to build applications that consume this data without being constrained by scalability and accessing underlying storage according to your development needs. 

### Scenario
In this scenario, Azure IoT Hub is used to exemplify the context in which such a solution could be useful. IoT Hub is a platform service on Azure which enables bidirectional communication to IoT devices using AMQP, MQTT or HTTP protocols (natively). For every device that is connected to an IoT Hub, a Device Twin is created. Device Twins are basically JSON documents that store device state information including metadata, configurations, and conditions. Each IoT Hub instance imposes throttling limits based on the service tier that is used, imposing a maximum number of reads and writes on the database that stores your Device Twins ([documentation](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-quotas-throttling)). 

This means that if you want build an application, dashboard or any other solution that often queries the device twin storage, you would need to scale up the whole IoT Hub instance, whereas in reality you might only use the storage part of all the higher tiered services. A solution for this would be to replicate the Device Twins to an external database which can store JSON documents, such as Cosmos DB. In that way, you can easily scale just your storage solution rather than paying for services you don't use. 

### Drivers
Building a solution that circumvents the current scenario will improve its scalability and independence. Since each element is instantiated separately, the solution becomes more modular and can easily be redesigned to adapt to a different scenario in the future. 


### Solution
![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/azure.png?raw=true)

Whenever a Device Twin is created in IoT Hub, the message routing feature for Lifecycle 
Once a change has occurred to an existing Device Twin in IoT Hub, the message routing feature for twin change events in IoT Hub sends an event to an Event Hub, which then triggers an Azure function to update the twin in Cosmos DB. This is also done for lifecycle events, sending an event to Event Hub when a device is deleted or created, triggering another Azure function to replicate that operation on the device twin in the Cosmos DB container.


#### Configuring IoT Hub to route Device Twin changes and Lifecycle events towards Event Hubs
To set up the message routing to send events to an Event Hub whenever changes occur to your Device Twins, go to `Message routing` under the *Messaging* tab and select `Add`:

![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/messagerouting1.png?raw=true)

There you would need to `Add an endpoint` which points to the Event Hub you would like to use for this scenario, and make sure you select `Device Lifecycle Events` from the dropdown list: 

![](https://github.com/machteldbogels/devicetwinsync/blob/master/images/messagerouting2.png?raw=true)

Repeat this process to create another Message routing for the `Device Twin Changes Events` which points to another Event Hub instance (which can be within the same Event Hub Namespace as the previous one).

Under the `Test` tab, visible at the bottom of the page when creating a new Message routing, you can see an example of the schema of  each event, including the `System Properties`, `Application Properties`, the `Message Body` and the `Device Twin`. 


#### Creating Event Hub-triggered Azure Functions
First, a local project needs to be created inside an IDE to develop your code. In this example I'm using VS Code, since it has an Azure Functions extension allowing you to easily create Functions and deploy your code without having to go to the Azure Portal.
When creating a local function in VS code, the first prompt you'll receive is to `Select a Language`, for which in this case `JavaScript` was used. The second prompt is to `Select a Template`, for which you'll have to select `Azure Event Hub Trigger`. After picking a name, you will need to `Create new local app setting` to set up the connection to the Event Hub where either one of your Message routes is sending events to. 


You can select a connection policy either at the namespace level, such as the `RootManageSharedAccessKey` or a policy that applies to a specific Event Hub. 
After that, you'll select the policy that is set up between your IoT Hub and Event Hub, 


The last prompt you'll receive is to set the consumer group of the Event Hub that you're listening to. If you haven't specified any particular consumer group within the configuration of the Event Hub then you can leave the `$Default` setting.

#### Creating or Deleting a Device Twin in Cosmos DB using the LifecycleUpdates function
The code for this function can be found [here](https://github.com/machteldbogels/devicetwinsync/blob/master/LifecycleUpdates/index.js)

The environment variables that point to the endpoint and key of your Cosmos DB instance can be stored either locally or within an Azure Key Vault for example.


#### Updating a Device Twin in Cosmos DB using the TwinChanges function
The code for this function can be found [here](https://github.com/machteldbogels/devicetwinsync/blob/master/TwinChanges/index.js)

Out of the four components which are visible during creation of the Route, namely the `System Properties`, `Application Properties`, `Message Body` and the `Device Twin`, by default only the `Message Body` is part of the event message that is written to Cosmos DB. Since we also want to know the `DeviceId` as well as the type of event (update/creation/deletion), visible as the `OpType` property within the `Application Properties`, these two values are added to the message body.

At the moment, the Cosmos DB SQL API does not support partial updates, so therefore the db has to be queried in order to retrieve the existing Device Twins and update only the part of the JSON document for which an update has occurred, either for the reported or desired properties.


### Related Patterns
This example was developed as part of a larger solution, for which the code and explanation can be found [here.](https://github.com/jessevl/azure-iot-durable-patterns)

