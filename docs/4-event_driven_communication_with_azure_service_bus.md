## Event-Driven Communication with Azure Service Bus

This section will cover using Azure Service Bus for messaging in our serverless .NET 8 application. We'll explore the messaging options, choose between Service Bus queues or topics, create the necessary Service Bus infrastructure using Bicep, and define function bindings for sending and receiving messages. Lastly, we'll implement retry policies and exponential backoff to ensure resilience.

## Overview of Azure Service Bus for Messaging

Azure Service Bus is a fully managed enterprise messaging service that allows for reliable message delivery between different components of an application. It provides decoupling between the sender (publisher) and the receiver (subscriber), making it ideal for event-driven architectures where components should operate independently.

- Key Benefits of Azure Service Bus:
  - Decoupling: Publishers and subscribers don’t need to know about each other.
  - Reliable Delivery: Guarantees message delivery with multiple modes (At-Least-Once, At-Most-Once).
  - Resilience: Built-in dead-letter queues (DLQ) and support for retries and error handling.
  - High Throughput: Can handle large volumes of messages with automatic scaling.
  - FIFO (First-In-First-Out): Ensures that messages are processed in order when needed (using sessions).

Azure Service Bus supports two key messaging models:

- Queues: Simple point-to-point communication.
- Topics: Publish-subscribe communication where multiple receivers (subscribers) can receive the same message.

## Choose Between Service Bus Queues or Topics for Event Communication

Depending on your application's needs, you will choose between queues or topics for message distribution.

- Service Bus Queues (Point-to-Point):

  - Use Case: Ideal when a single consumer (or worker) processes messages from a queue.
  - How it works: Messages are sent to a queue, and one receiver processes each message. Once the message is processed, it is removed from the queue.
  - Benefits: Simple to set up, guarantees at-least-once delivery, and supports dead-lettering for failed messages.

- Service Bus Topics (Publish-Subscribe):
  - Use Case: Useful when you want multiple subscribers to receive a copy of each message.
  - How it works: A message is sent to a topic, and multiple subscribers (subscriptions) can receive the same message. Each subscription can also define filters to receive only certain types of messages.
  - Benefits: Scales to support many subscribers, useful for broadcasting messages to multiple services.
    For this guide, we'll use a Service Bus Queue for point-to-point messaging.
- Creating a Service Bus Namespace and Queue Using Bicep
  - In this step, we will use Bicep (an Azure-native Infrastructure-as-Code tool) to create a Service Bus Namespace and a Queue.

Bicep Template: servicebus.bicep

```bicep
resource serviceBusNamespace 'Microsoft.ServiceBus/namespaces@2021-11-01' = {
  name: 'myServiceBusNamespace'
  location: resourceGroup().location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
}

resource serviceBusQueue 'Microsoft.ServiceBus/namespaces/queues@2021-11-01' = {
  name: 'myServiceBusQueue'
  parent: serviceBusNamespace
  properties: {
    enablePartitioning: true
    requiresDuplicateDetection: false
  }
}
```

Deploying the Bicep Template:

- Save the Bicep template as servicebus.bicep.
- Use the Azure CLI to deploy the template:

  ```bash
  az group create --name <your-resource-group> --location <your-location>
  az deployment group create --resource-group <your-resource-group> --template-file ./servicebus.bicep
  ```

  > You should only create a resource group once. But you can deploy the Bicep template multiple times to update the existing resources.

- This will create the Service Bus Namespace and Queue in your Azure resource group.

## Define Function Bindings to Send/Receive Messages from Service Bus

Next, we'll define bindings in the Azure Function to integrate it with the Service Bus Queue. Azure Functions bindings make it easy to send and receive messages without directly handling the Service Bus SDK.

### Sending Messages to the Service Bus Queue

To send messages from the HTTP-triggered function to the Service Bus Queue, add an output binding in the function's attributes.

Update `HttpTriggerFunction.cs` to include a Service Bus output binding:

```csharp
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;

namespace MyServerlessApp
{
    public static class HttpTriggerFunction
    {
        [FunctionName("HttpTriggerFunction")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = null)] HttpRequest req,
            [ServiceBus("myServiceBusQueue", Connection = "ServiceBusConnectionString")]
                IAsyncCollector<string> messageCollector,
            ILogger log
        )
        {
            log.LogInformation("HTTP trigger function received a request.");

            // Read the request body
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

            // Send the message to the Service Bus Queue
            await messageCollector.AddAsync(requestBody);

            return new OkObjectResult("Message sent to Service Bus Queue.");
        }
    }
}

```

Explanation:

- ServiceBus attribute: The [ServiceBus] attribute binds the function to send messages to the specified Service Bus Queue (myServiceBusQueue).

- Connection string: The ServiceBusConnectionString is stored in your application’s settings. You’ll need to configure this in local.settings.json for local development:

  ```json
  {
    "IsEncrypted": false,
    "Values": {
      "AzureWebJobsStorage": "UseDevelopmentStorage=true",
      "FUNCTIONS_WORKER_RUNTIME": "dotnet",
      "ServiceBusConnectionString": "Endpoint=sb://<your-servicebus-namespace>.servicebus.windows.net/;SharedAccessKeyName=<key-name>;SharedAccessKey=<key>"
    }
  }
  ```

  - Generating the ServiceBusConnectionString using az command:

  ```bash
  az servicebus namespace authorization-rule keys list --resource-group <your-resource-group> --namespace-name <your-servicebus-namespace> --name RootManageSharedAccessKey --query primaryConnectionString --output tsv
  ```

  This should output the connection string that you can use in your local.settings.json file.

## Receiving Messages from the Service Bus Queue

Create another Azure Function that will receive messages from the Service Bus Queue:

Create an Event-Triggered Function
In the same project you created earlier, generate a new function that will be triggered by Service Bus messages.

In the terminal, run the following command to create a new Service Bus-triggered Azure Function:

```bash
func new --template "Service Bus Queue trigger" --name ServiceBusQueueTriggerFunction
```

This creates a function template named ServiceBusQueueTriggerFunction that listens to messages from the specified Service Bus queue. Also install the required NuGet packages from where the ServiceBusTrigger attribute is defined.

Now, update the ServiceBusQueueTriggerFunction function to read messages from the queue and process them. Open the ServiceBusQueueTriggerFunction.cs file and modify it as follows:

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

namespace MyServerlessApp
{
    public static class ServiceBusQueueTriggerFunction
    {
        [FunctionName("ServiceBusQueueTriggerFunction")]
        public static void Run(
            [ServiceBusTrigger("myServiceBusQueue", Connection = "ServiceBusConnectionString")]
                string queueMessage,
            ILogger log
        )
        {
            log.LogInformation($"Received message: {queueMessage}");
        }
    }
}

```

This function listens to the Service Bus Queue and logs each message it receives.

### Handling Resilience: Retry Policies and Exponential Backoff

Azure Functions and Service Bus have built-in support for retry policies to handle transient failures. You can configure retries and exponential backoff in your host.json file.

`host.json` Configuration:

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "excludedTypes": "Request"
      },
      "enableLiveMetricsFilters": true
    }
  },
  "extensions": {
    "serviceBus": {
      "messageHandlerOptions": {
        "maxConcurrentCalls": 16,
        "autoComplete": true,
        "maxAutoRenewDuration": "00:05:00"
      }
    }
  },
  "retry": {
    "strategy": "exponentialBackoff",
    "maxRetryCount": 5,
    "minimumInterval": "00:00:02",
    "maximumInterval": "00:01:00"
  }
}
```

Explanation:

- Retry policy: Configures exponential backoff with retries starting at 2 seconds and a maximum of 1 minute between attempts.

### Testing the Event-Driven Communication Locally

To test the event-driven communication locally, run the Azure Functions project using the Azure Functions Core Tools:

```bash
func start
```

This will start the Azure Functions runtime locally and allow you to test sending messages to the Service Bus Queue and receiving them in the triggered function.

Your local functions will listen the events on the Service Bus Queue previously created in cloud using the Bicep template, using the Connection String provided in the local.settings.json file.

Example: Send a message to the HTTP-triggered function:

```bash
curl -X POST http://localhost:7071/api/HttpTriggerFunction -d "Hello, Azure!"
```

### Summary

In this section, we configured the Azure Service Bus for event-driven communication by creating a namespace and queue using Bicep. We integrated Service Bus with Azure Functions using bindings for sending and receiving messages. Finally, we implemented resilience with retry policies and exponential backoff to handle transient failures and dead-lettering.

In the next section, we will test and debug the event-driven architecture locally and start adding the database integration.
