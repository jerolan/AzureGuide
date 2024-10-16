# Setting Up the Development Environment

Before building the serverless .NET 8 application, you need to set up the development environment. This section provides a step-by-step guide for installing the necessary tools and configuring your IDE for development with Azure Functions, Azure Service Bus, and Bicep.

## Install .NET 8 SDK

You need to install the .NET 8 SDK to develop Azure Functions using .NET 8.

### Steps:

- Download the .NET 8 SDK:
  - Go to the official .NET SDK download page.
  - Select .NET 8 SDK and download the appropriate version for your operating system.
- Install .NET 8 SDK:
  - Run the installer and follow the prompts to complete the installation.
- Verify Installation:
  - Open a terminal (Command Prompt, PowerShell, or terminal for macOS/Linux).
  - Run the following command to verify that .NET 8 is installed correctly:
  ```bash
  dotnet --version
  ```
  - The output should show a version number starting with 8.x.

## Install Azure CLI (az)

The Azure CLI (az) is necessary to manage Azure resources, and it is also a prerequisite for using Azure Functions Core Tools.

### Steps:

- Download and Install Azure CLI:
  - Visit the official Azure CLI installation page and follow the instructions based on your operating system.
- Verify Installation:
  - Open a terminal and run the following command to check if the Azure CLI is installed correctly:
  ```bash
  az version
  ```
  - This command should display the installed Azure CLI version and details about its components.
- Log in to Azure:
  - Run the following command to log in to your Azure account:
  ```bash
  az login
  ```

## Install Azure Functions Core Tools

The Azure Functions Core Tools is essential for creating, running, and debugging Azure Functions locally.

### Steps:

- Install Azure Functions Core Tools:

  - You can install Azure Functions Core Tools using npm, Homebrew (macOS), or Chocolatey (Windows).
  - Using npm:

  ```bash
  npm install -g azure-functions-core-tools@4 --unsafe-perm true
  ```

  - Using Homebrew (macOS):

  ```bash
  brew tap azure/functions
  brew install azure-functions-core-tools@4
  ```

  - Using Chocolatey (Windows):

  ```bash
  choco install azure-functions-core-tools-4
  ```

- Verify Installation:
  - Run the following command to ensure Azure Functions Core Tools is installed correctly:
  ```bash
  func --version
  ```
  - You should see a version like 4.x.x.

## Setting up Visual Studio/VS Code for Azure Functions and Bicep

You can choose to develop your project in either Visual Studio or Visual Studio Code. Both tools support Azure Functions development and infrastructure management using Bicep.

- Option 1: Visual Studio (Windows/Mac)
  - Install Visual Studio:
    - Download and install Visual Studio 2022.
  - Install Azure Functions Tools:
    - Open Visual Studio and go to Tools > Get Tools and Features.
    - Ensure that Azure Development is selected, which includes support for Azure Functions.
  - Install Bicep Extension:
    - In Visual Studio, go to Extensions > Manage Extensions.
    - Search for Bicep and install the Bicep Language Support extension.
- Option 2: Visual Studio Code (Cross-platform)
  - Install Visual Studio Code:
    - Download and install Visual Studio Code.
  - Install Azure Functions Extension:
    - Open VS Code and go to Extensions (Ctrl + Shift + X).
    - Search for Azure Functions and install the Azure Functions extension.
  - Install Bicep Extension:
    - In the Extensions view, search for Bicep and install the Bicep extension. This extension allows you to write and manage Bicep files directly from VS Code.
  - Install C# Extension (for .NET 8):
    - Install the C# extension for Visual Studio Code, which provides language support for .NET 8.
  - Install Azure CLI Extension (Optional):
    - You can also install the Azure CLI Tools extension to manage Azure resources directly from VS Code.

## Summary

By following these steps, you will have a complete development environment with the necessary tools to develop, test, and deploy Azure Functions. With Azure CLI and Azure Functions Core Tools, you can manage both the local development and deployment of your serverless application. You can now move forward to create your first Azure Function and implement event-driven communication using Azure Service Bus.
