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

7. Click "+ Add", you may use either Express or Advanced method - give it a friendly name, and select the Vnet that we have created in Step 3, and the subnet created in Step 5. **DO NOT** use the subnet that has been delegated for Power Platform.
   <img width="788" height="288" alt="image" src="https://github.com/user-attachments/assets/8edcb20a-848d-4fe5-88cb-7a6081f81eb2" />

8. Our MCP Server function app now has a private end point inside this vnet. Click on the Private endpoint name.
   <img width="719" height="303" alt="image" src="https://github.com/user-attachments/assets/b2b2c74b-32cd-4e64-9aef-18a04332b12c" />

9. In the Private endpoint settings, go to **"DNS configuration"**. Note that we now have a Private DNS zone created. Click on the private DNS Zone ("privatelink.azurewebsites.net").
    <img width="1038" height="556" alt="image" src="https://github.com/user-attachments/assets/4090c734-bf70-49e3-b20b-91f7f7f0229d" />

10. In the Private DNS Zone, under **DNS Management** click on **"Virtual Network Links"**. We will need to link the vnet in our other region to this zone. Click **"+ Add"**. 
    <img width="999" height="341" alt="image" src="https://github.com/user-attachments/assets/69c679f0-f8cb-4845-80a7-b35d7748d028" />

11. Provide a friendly name for the link, and select 


