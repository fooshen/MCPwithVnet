# Deploying a sample MCP server
In this example, we will build and deploy a very basic, no-frills HelloWorld MCP server using C# into Azure Function using Visual Studio Code. 
You can skip this part if you already have an MCP server to work with, or are using other samples - this is not about writing an MCP server, but this is intended for an easy follow through if you don't already have one and wanted to quickly get to the VNet part.
For examples on writing MCP Servers on Azure Functions, you may refer to this article: [Tutorial: Host an MCP server on Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-mcp-tutorial?tabs=mcp-extension&pivots=programming-language-csharp). View more examples [here](https://github.com/microsoft/mcp-dotnet-samples).


## You will need
- Visual Studio Code
- Azure subscription

### Create a simple Hello World MCP server using C#
1. [Install Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) for Visual Studio Code.
2. In Visual Studio Code, press Ctrl+Shift+P or F1, and select Azure Functions: Create New Project
   <img width="1063" height="201" alt="image" src="https://github.com/user-attachments/assets/5d0278b7-8a95-4b3d-b91b-8ad8a96f288f" />
3. Create a new folder "HelloWorldMCPServer".
4. Select "C#", then ".NET 8.0 Isolated LTS" for runtime, and "McpToolTrigger" for project template.
   <img width="1361" height="261" alt="image" src="https://github.com/user-attachments/assets/eb3be78a-13de-4721-b540-7e524db9db55" />
   
> [!NOTE]
> This uses the Azure Functions MCP Extension project scaffolding that sets up everything we need to create an MCP tool endpoint. Read more about this [here](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-mcp?pivots=programming-language-csharp). 
   
5. Provide a function name - let's go with "SayHello". Give it a namespace - I am using fsdemo here.
6. Visual Studio will generate the necessary files for a basic, no-frills MCP server. In Explorer view, select "host.json" file and let's change "webhookAuthorizationLevel" to "Anonymous" to simplify our testing. Update the instructions and serverName property for our HelloWorldMCPServer.
```CSharp
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
    "mcp": {
      "instructions": "Greet the user with a simple 'Hello, World!' message.",
      "serverName": "HelloWorldMCPServer",
      "serverVersion": "2.0.0",
      "encryptClientState": true,
      "messageOptions": {
        "useAbsoluteUriForEndpoint": false
      },
      "system": {
        "webhookAuthorizationLevel": "Anonymous"
      }
    }
  }
}
```
7. This no-frills sample only has one tool - SayHello. It should just simply greet the user with the message "Hello {user}! This is an MCP Tool." Open the file SayHello.cs - I like to add a time stamp in its response.
```CSharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Extensions.Mcp;
using Microsoft.Extensions.Logging;

namespace fsdemo;

public class SayHello
{
    private ILogger<SayHello> _logger;

    public SayHello(ILogger<SayHello> logger)
    {
        _logger = logger;
    }

    [Function(nameof(SayHello))]
    public object Run(
        [McpToolTrigger(nameof(SayHello), "Responds to the user with a hello message.")] ToolInvocationContext context,
        [McpToolProperty(nameof(name), "The name of the person to greet.")] string? name
    )
    {
        _logger.LogInformation("C# MCP tool trigger function processed a request.");
        return new
        {
            content = new[]
            {
                new
                {
                    type = "text",
                    text =  $"Hello, {name ?? "world"}! This is an MCP Tool! Time now is {DateTime.Now}"
                }
            }
        };       
       
    }
}
```
> [!NOTE]
> The default function returns a raw string. We need to change this to return a json object instead. Copilot Studio uses diff-style renderer for raw strings. If you ever see MCP Tool responses in Copilot Studio with strikethrough format like "Hello, this is MCP tool. ~Time now is {now}~", it is because the tool is returning raw string. We should avoid returning raw strings in our MCP tools. 

8. We are now ready for a quick, local test - Hit F5. When prompted, select **"Use Local Emulator"** - this will set up Azurite as a local emulator for Azure Blob Storage that our Function app will use.
<img width="570" height="123" alt="image" src="https://github.com/user-attachments/assets/8d299c61-20a8-4cdd-ae9d-accd003b1123" />
If you get a "Failed to verify 'AzureWebJobStorage' connection specified in "local.Settings.json", click **"Run Emulator"** to start it. Wait for the build to complete.
9. You should now see your local server running:
<img width="752" height="58" alt="image" src="https://github.com/user-attachments/assets/018d78da-7238-4f00-852a-3c3fc1c28ba0" />
10. We can now do some simple testing locally. In Visual Studio Code's GitHub Copilot chat, type **use #SayHello** and pick the SayHello tool, and add a parameter as the user name.
<img width="452" height="169" alt="image" src="https://github.com/user-attachments/assets/fb89bc72-4f3b-46ef-a23d-5682839bcb90" />
When prompted, select **"Allow"**
<img width="304" height="310" alt="image" src="https://github.com/user-attachments/assets/782acba3-80be-4de1-951c-cd2f1e7d1abe" />
11. Now let's deploy into Azure Function. 





    
 
