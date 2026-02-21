# Deploying a sample MCP server
In this example, we will build and deploy a very basic HelloWorld MCP server using C# into Azure Function using Visual Studio Code. 
You can skip this part if you already have an MCP server to work with, or are using other samples - this is not a tutorial on writing MCP server.


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
   > This uses the Azure Functions MCP Extension project scaffolding that sets up everything we need to create an MCP tool endpoint.
   
5. Provide a function name - let's go with "SayHello". Give it a namespace - I am using fsdemo here. 

 
