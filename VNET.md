# Part 2: Setting up and connecting Power Platform to VNet

Now that we have built and deployed an MCP Server in Azure Function and connected to it in Copilot Studio, and subsequently disabled public network access - we will now set up our Environment to connect over Vnet so that Copilot Studio can continue to use this MCP Server over a private end point.

## Set up Azure Virtual Network ##
1. In your Azure Subscription, create a new Virtual Network resource.
   
   <img width="200" alt="image" src="https://github.com/user-attachments/assets/9d2300b6-6a40-4df1-8d88-518e0d1989d4" />

3. Give it a name and select the region that corresponds to your Power Platform Environment - see [here](README.md#before-you-start) (eg mydemo-vnet-australiasoutheast). You may leave everything else as default - for a bare minimal example, we don't need Azure Bastion, Firewall, etc. If your Power Platform geography has more than one region, you need to create another vnet - one for each region. Refer [here](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview#supported-regions) to see which Azure region your Power Platform geography is mapped with. In my example, as Environments in Australia are mapped to **australiaeast** and **australiasoutheast**, I will need to create two VNet - one for each region.
   <img width="900" alt="image" src="https://github.com/user-attachments/assets/918a9948-4ff0-4b14-be14-a8a9973f5b8f" />


5. In the VNet resource for the same region as your Power Platform Environment (in my example, Australia southeast), you will need to create at least **two** subnets. We will need Subnet1 (in this example below: "fsdemomcp-subnet") for our MCP Server in Azure Function, and another Subnet2 ("pp-vnet") for Power Platform to use.
   
   <img width="808" height="370" alt="image" src="https://github.com/user-attachments/assets/26ba89b9-77d9-445f-bcb4-964ea05e56cb" />


> [!IMPORTANT]
> Subnet size matters! Refer [here](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview#estimating-subnet-size-for-power-platform-environments) for a guide on estimating subnet sizes for your vnet. Rule of thumb: Plan for 25-30 IP addresses for typical production environment. Do not share the Production Environment's Vnet policy with other environments.
>

5. This is a very important step - for the Power Platform delegated subnet, it must not be shared or used by other purposes. In the second subnet "pp-vnet", set **"Subnet Delegation"** to **"Microsoft.PowerPlatform/enterprisePolicies"**. This allows Power Platform to manage this subnet - and at runtime, it will run containers in this delegated subnet, which in turn will be able to connect to resources within the same vnet.
   <img width="800" alt="image" src="https://github.com/user-attachments/assets/1d37ba8b-1639-4150-bd8d-d23d3c5a3893" />

6. Next, we head back into our HelloWorldMCPDemo Azure Function app. Under Settings -> Network, click on the private endpoints to create one. 
   <img width="1001" height="520" alt="image" src="https://github.com/user-attachments/assets/b6290a29-ea57-4cf9-a55a-77cd19836303" />
