# Create and deploy a simple MCP server on Azure Function and testing in Copilot Studio
In this example, we will build and deploy a very basic, no-frills HelloWorld MCP server using C# into Azure Function using Visual Studio Code. 
You can skip this part if you already have an MCP server to work with, or are using other samples - this is not about writing an MCP server, but this is intended for an easy follow through if you don't already have one and wanted to quickly get to the VNet part.
For examples on writing MCP Servers on Azure Functions, you may refer to this article: [Tutorial: Host an MCP server on Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-mcp-tutorial?tabs=mcp-extension&pivots=programming-language-csharp). View more examples [here](https://github.com/microsoft/mcp-dotnet-samples).


## You will need
- Visual Studio Code
- Azure subscription
- Power Platform Environment with Copilot Studio

### Create a simple Hello World MCP server using C#
1. [Install Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) for Visual Studio Code.
2. In Visual Studio Code, press Ctrl+Shift+P or F1, and select Azure Functions: Create New Project
   <img width="800" alt="image" src="https://github.com/user-attachments/assets/5d0278b7-8a95-4b3d-b91b-8ad8a96f288f" />
   
3. Create a new folder "HelloWorldMCPServer".
4. Select "C#", then ".NET 8.0 Isolated LTS" for runtime, and "McpToolTrigger" for project template.

   <img width="800" alt="image" src="https://github.com/user-attachments/assets/eb3be78a-13de-4721-b540-7e524db9db55" />
   
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

7. This no-frills sample only has one tool - SayHello. It should just greet the user with the message "Hello {user}! This is an MCP Tool." Open the file SayHello.cs - I like to add a time stamp in its response. The Azure Function MCP Extension scaffoling means we simply need to decorate using **McpToolTrigger** and **McpToolProperty** attributes. Note the tool name "Say Hello" specified in the attribute.

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
        [McpToolTrigger("Say Hello", "Responds to the user with a hello message.")] ToolInvocationContext context,
        [McpToolProperty("Name", "The name of the person to greet.")] string? name
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

8. We are now ready for a quick, local test - Hit F5. When prompted, select **"Use Local Emulator"** - this will set up Azurite as a local emulator for Azure Blob Storage that our Function app will use. If you get a "Failed to verify 'AzureWebJobStorage' connection specified in "local.Settings.json", click **"Run Emulator"** to start it. Wait for the build to complete.

   <img width="570" height="123" alt="image" src="https://github.com/user-attachments/assets/8d299c61-20a8-4cdd-ae9d-accd003b1123" />

9. You should now see your local server running:

   <img width="752" height="58" alt="image" src="https://github.com/user-attachments/assets/018d78da-7238-4f00-852a-3c3fc1c28ba0" />

10. We can now do some simple testing locally. In Visual Studio Code's GitHub Copilot chat, type **use #SayHello** and pick the SayHello tool, and add a parameter as the user name.

    <img width="452" height="169" alt="image" src="https://github.com/user-attachments/assets/fb89bc72-4f3b-46ef-a23d-5682839bcb90" />

12. When prompted, select **"Allow"**, otherwise the call is blocked.

    <img width="304" height="310" alt="image" src="https://github.com/user-attachments/assets/782acba3-80be-4de1-951c-cd2f1e7d1abe" />

## Deploy to Function Apps ##
11. Now let's deploy to Azure Function. Press CTRL+SHIFT+P or F1 (or simply click on the search/command bar at the top and click "Show and Run Commands"). Select **"Azure Functions: Deploy to Function Apps...".**
12. Select **"+ Create new function app..."** (or use existing one, but you must ensure the function app is in the [same region as your Power Platform Environment](README.md#before-you-start). Enter a name for your Function App (example, "HelloWorldMCPDemo"). If creating a new Function App, please select the same region that your Power Platform Environment is in. In my example, I'm on Australia Southeast.
<img width="400" alt="image" src="https://github.com/user-attachments/assets/92759ef4-e862-41d9-87b9-95dbc32296b1" />.

13. Select **".Net 8 Isolated"**, and **Secrets** for resource authentication type (it will need to communicate with Azure Blob Storage and optionally App Insights).

   > [!IMPORTANT]
   > Do not create Azure Function on Consumption plan - use Flex Consumption or any other plans. Consumption Plan does not support Private Link and will not be useful in our exercise here. See [here](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale#networking-features) on the networking features supported by the different plans.
   >

14. In your Azure portal, you can check the Function app is available and is running. Take note of the domain URL and the region.

    <img width="791" height="437" alt="image" src="https://github.com/user-attachments/assets/024a763b-812c-42ed-a7f3-5caa1579af85" />

## Test in Copilot Studio ##
15. Next, we will test the MCP tool in Copilot Studio. Make sure you are in the correct Environment - the Environment that we will set up VNet connectivity. In a test agent (create new one if you are not using an existing agent), scroll down to "Tools" and click **"+ Add Tool"**.

    <img width="651" height="226" alt="image" src="https://github.com/user-attachments/assets/efc5e60c-181f-4be3-a0e6-802769fad134" />

17. Select **"Model Context Protocol"** in the "Add Tool" dialog.

18. Enter a server name and description. For Server URL, enter the function app's URL as such: https://<functionapp>/runtime/webhooks/mcp. You can review the mcp.json file in Visual Studio code to see the url.

    <img width="500" alt="image" src="https://github.com/user-attachments/assets/9c4c5e36-5e2f-4fee-a77b-02375acc8991" />

   > [!NOTE]
   > The Azure Function MCP Extension project will always create this as http-streamable. SSE protocol has been deprecated for MCPs. Azure Functions will use the /mcp path for Streamable HTTP, and /mcp/sse for SSE.
   >

17. Next, you will be asked to create a connection. Click on **"Create new connection"**, and then **Add and configure**".

    <img width="500" alt="image" src="https://github.com/user-attachments/assets/28a2211d-20d2-464e-a37c-5774882fc604" />

19. By now, if you head into https://make.powerapps.com and navigate to **Custom Connectors**, you will see the MCP tool showing up as a Custom Connector.

    <img width="800" alt="image" src="https://github.com/user-attachments/assets/5d0b9d08-7f8a-40b8-a8e3-85c3cb47a651" />

21. Switching to Swagger view, we should see it looking like this - note the x-agentic-protocol attribute.

    <img width="500" alt="image" src="https://github.com/user-attachments/assets/5a3a8526-664d-411f-97f3-c9546bb08e49" />

23. Back in Copilot Studio, we can see the tool "Say Hello" discovered.

    <img width="500" alt="image" src="https://github.com/user-attachments/assets/c0d9c471-1288-4905-a6f3-95a64eea0e4e" />

25. Let's use the Test Pane and give it a prompt to greet a user name using the Say Hello tool. Note that you may get prompted to select the connection that has just been created, or if the connection is stale.

    <img width="400" alt="image" src="https://github.com/user-attachments/assets/9e5d0e7b-493e-4f07-a89f-4fc6ab27eb4c" />

27. We have now established that our MCP Sever is running and is connected to Copilot Studio. Now, we will remove the MCP server from public access.

28. Back in the Azure Function app, go to Settings > Networking. We can see that it has Public network access enabled.

    <img width="800" alt="image" src="https://github.com/user-attachments/assets/2099c205-d87d-455a-b689-62e222331f75" />

30. Click on the Public network access setting "Enabled with no access restrictions" to change this to **Disabled** and click **Save**. You will need to confirm via a checkbox to agree to the change. Now, our function app is no longer accessible over the public network.
   
    <img width="1597" height="505" alt="image" src="https://github.com/user-attachments/assets/ed202684-f3d8-4fab-84ae-0bbc44b3bc0f" />

31. We can now see the setting for Public network access as **"Disabled"**.

    <img width="1012" height="412" alt="image" src="https://github.com/user-attachments/assets/8b9a2fdd-5f43-424e-bf08-172965318ff2" />

32. Back in Copilot Studio, if we refresh the tool list, you will now see that Copilot Studio is unable to retrieve the tool list with an error "Connector request failed". Testing on the chat test pane will give a generic Hello rather than a response from the MCP tool. This proofs that we can no longer connect to the MCP server as we have disabled public network access. 

    <img width="903" height="269" alt="image" src="https://github.com/user-attachments/assets/3b0074d9-0ec3-4560-bcbb-05bb2fcf3baa" />








    
 
