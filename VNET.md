# Part 2: Setting up and connecting Power Platform to VNet

Now that we have built and deployed an MCP Server in Azure Function and connected to it in Copilot Studio, and subsequently disabled public network access - we will now set up our Environment to connect over Vnet so that Copilot Studio can continue to use this MCP Server over a private end point.

## Set up Azure Virtual Network ##
1. In your Azure Subscription, create a new Virtual Network resource.
   <img width="320" height="499" alt="image" src="https://github.com/user-attachments/assets/9d2300b6-6a40-4df1-8d88-518e0d1989d4" />

2. Give it a name and select the region that corresponds to your Power Platform Environment (see [here](README.md#) (eg mydemo-vnet-australiasoutheast) 
