# Creating the HTTP-Triggered Azure Function

In this section, we will create a new Azure Function project and implement an HTTP-triggered function that will handle incoming requests. This function will act as the entry point for your serverless application.

## Create a New Azure Function Project

First, we need to create a new Azure Function project using the Azure Functions Core Tools.

Using Azure Functions Core Tools

- Open a Terminal (Command Prompt, PowerShell, or terminal for macOS/Linux).
- Navigate to Your Project Directory:
  ```bash
  cd /path/to/your/project/directory
  ```
- Create a New Azure Function Project: Run the following command to create a new Azure Functions project in .NET 8:
  ```bash
  func init MyServerlessApp --dotnet
  cd MyServerlessApp
  ```
- Add a New HTTP-Triggered Function: After initializing the project, create an HTTP-triggered function by running:
  ```bash
  func new
  ```
  - Choose `HttpTrigger` from the list of available templates.
  - Name the function (e.g., `HttpTriggerFunction`).
  - Open the Project in Your IDE (e.g., Visual Studio Code):
    ```bash
    code .
    ```

## Implement the HTTP Trigger to Handle Incoming Requests

Once the Azure Function project is created, we can modify the generated HTTP-triggered function to handle incoming requests.

The default HTTP-triggered function will be located in a file named HttpTriggerFunction.cs or similarly named based on your chosen function name.

Example Code for HttpTriggerFunction.cs:

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
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)]
                HttpRequest req,
            ILogger log
        )
        {
            log.LogInformation("HTTP trigger function received a request.");

            // Read the request body (for POST requests)
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

            // Get query string parameters (for GET requests)
            string name = req.Query["name"];

            // If no name in query, check the request body
            if (string.IsNullOrEmpty(name))
            {
                dynamic data = System.Text.Json.JsonSerializer.Deserialize<dynamic>(requestBody);
                name = name ?? data?.name;
            }

            // Return a simple greeting
            return name != null
                ? (ActionResult)new OkObjectResult($"Hello, {name}!")
                : new BadRequestObjectResult(
                    "Please pass a name on the query string or in the request body."
                );
        }
    }
}
```

Breakdown of the Code:

- FunctionName attribute: This defines the name of the Azure Function. The function is called HttpTriggerFunction.
- HTTP Trigger Binding:
  - The [HttpTrigger] attribute binds the function to an HTTP trigger.
  - The AuthorizationLevel.Anonymous means no authentication is required.
  - The function listens for both GET and POST requests ("get", "post").
- Reading the Request:
  - The code reads query string parameters (req.Query["name"]) for GET requests.
  - It reads the request body for POST requests.
- Returning a Response:
  - If a name is found in the query string or request body, the function returns a greeting.
  - If no name is provided, the function returns a 400 Bad Request response with an error message.

## Test the HTTP-Triggered Azure Function Locally

You can now test the function locally using Azure Functions Core Tools.

- Run the Function Locally:
  - Open a terminal in the project directory and run the following command:
  ```bash
  func start
  ```
- Send a Test HTTP Request:

  - Open a browser or use a tool like curl or Postman to send an HTTP request to the function.

    For example, use curl:

    ```bash
    curl "http://localhost:7071/api/HttpTriggerFunction?name=Azure"
    ```

    This should return:

    ```bash
    Hello, Azure!
    ```

- Test POST Request: You can also test POST requests with JSON data:

  ```bash
  curl -X POST "http://localhost:7071/api/HttpTriggerFunction" -H "Content-Type: application/json" -d '{"name": "Azure"}'
  ```

  Expected output:

  ```
  Hello, Azure!
  ```

## Summary

At this point, you have created and tested a basic HTTP-triggered Azure Function that handles both GET and POST requests. This function will act as the entry point for the serverless application, receiving data from the client and triggering subsequent event-driven processes.

In the next steps, we will connect this function to Azure Service Bus for event-driven communication and implement retry policies and dead-letter queue handling.
