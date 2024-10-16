# Overview of the application

In this guide, we will develop a serverless .NET 8 application using Azure Functions to handle HTTP requests and orchestrate a series of subsequent processes via event-driven communication. The application will be designed to ensure resilience through retry policies with exponential backoff, and it will handle failed messages using a Dead-Letter Queue (DLQ). Initially, the application will focus on event-driven communication without database integration. In the second part, we will integrate a MariaDB SQL database for storage and enhance the overall application.

This guide will use Azure’s serverless and messaging services, alongside infrastructure as code (IaC) with Bicep, to ensure scalable and automated deployments.

## Overview of the Application

The application will expose an HTTP endpoint that accepts incoming requests, processes the data, and then triggers a chain of events. These events will be handled by different Azure Functions communicating asynchronously through Azure Service Bus (queues or topics). The focus is on designing a loosely coupled system with high availability, scalability, and resilience.

The key goals of the application are:

- **Serverless Architecture**: Using Azure Functions to reduce infrastructure management.
- **Event-Driven Communication**: Utilize Azure Service Bus to trigger subsequent processes based on events.
- **Resilience**: Implement retry policies with exponential backoff to handle transient failures and ensure message delivery.
- **Dead-Letter Queue (DLQ)**: Capture failed messages for later inspection or reprocessing.

## High-Level Architecture

The architecture follows an event-driven pattern, broken down into the following components:

- HTTP Trigger (Azure Function):
  - An Azure Function exposes an HTTP endpoint that receives requests (e.g., new data or events).
  - After receiving a request, it sends a message to an Azure Service Bus queue or topic for further processing.
- Service Bus Queue/Topic:
  - The Service Bus queue/topic will act as the central messaging hub, decoupling the HTTP-triggered Azure Function from the subsequent processes. Messages will be distributed to appropriate consumers.
- Event Processing Functions:
  - A series of Azure Functions will be triggered by the messages in the Service Bus queue/topic. Each function can perform specific tasks, such as logging, validation, or further processing.
- Retry and Resilience:
  - Each Azure Function will have retry policies with exponential backoff to handle transient failures. In case of repeated failures, messages will be moved to a Dead-Letter Queue (DLQ) for later analysis.
- Dead-Letter Queue (DLQ):
  - Messages that cannot be processed after multiple retries will be automatically transferred to the DLQ. The system can be configured to reprocess or inspect these messages for troubleshooting.

Later, we will introduce MariaDB as the persistence layer, which will store processed data coming from the event-driven workflow.

## Technologies and Services Used

- .NET 8: The core technology for building Azure Functions and handling HTTP requests and responses.
- Azure Functions: A serverless compute service used to run event-driven code in response to HTTP requests and Service Bus messages.
- Azure Service Bus: A fully managed enterprise message broker for decoupling applications through queues and topics, used for event-driven communication.
- Azure Functions Bindings for Service Bus: Simplifies integration between Azure Functions and Service Bus, allowing functions to easily send and receive messages.
- Dead-Letter Queues (DLQ): A sub-queue in Azure Service Bus for handling messages that cannot be delivered or processed.
- Retry Policies and Exponential Backoff: To implement resilience and robustness by retrying failed operations.
- Bicep: An Azure-native infrastructure-as-code (IaC) tool to manage and deploy Azure resources programmatically.
- MariaDB (to be added later): A SQL-based relational database used for data storage once the event-driven flow is complete.
