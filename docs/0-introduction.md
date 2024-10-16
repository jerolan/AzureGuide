# Initial Application without Database Integration

## Introduction

### Overview of the application

- High-level architecture
- Technologies and services used

### Setting Up the Development Environment

- Install .NET 8 SDK
- Install Azure Developer CLI or Azure Functions Core Tools
- Setting up Visual Studio/VS Code for Azure Functions and Bicep

### Creating the HTTP-Triggered Azure Function

- Create a new Azure Function project
- Implement the HTTP trigger to handle incoming requests

### Defining Event-Driven Communication

- Overview of Azure Service Bus or Event Grid for messaging
- Choose between Service Bus queues or topics for event communication
- Creating a Service Bus Namespace and Queue/Topic using Bicep
- Define Function bindings to send/receive messages from Service Bus

### Handling Resilience: Retry Policies and Exponential Backoff

- Set up retry policies with exponential backoff in Function App
- Using Service Bus message dead-letter queues (DLQ) for error handling

### Bicep Template for Infrastructure

- Define Bicep template for deploying Azure Function and Service Bus
- Deploying infrastructure using Azure CLI

### Testing and Debugging the Application Locally

- Running the Azure Function locally
- Sending test HTTP requests and validating message processing through Service Bus
