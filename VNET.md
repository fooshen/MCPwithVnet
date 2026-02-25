# Part 2: Setting up and connecting Power Platform to VNet

Now that we have built and deployed an MCP Server in Azure Function and connected to it in Copilot Studio, and subsequently disabled public network access - we will now set up our Environment to connect over Vnet so that Copilot Studio can continue to use this MCP Server over a private end point.

## Set up Azure Virtual Network ##
1. In your Azure Subscription, create a new Virtual Network resource.
   
   <img width="200" alt="image" src="https://github.com/user-attachments/assets/9d2300b6-6a40-4df1-8d88-518e0d1989d4" />

3. Give it a name and select the region that corresponds to your Power Platform Environment - see [here](README.md#before-you-start) (eg mydemo-vnet-australiasoutheast). You may leave everything else as default - for a bare minimal example, we don't need Azure Bastion, Firewall, etc. If your Power Platform geography has more than one region, you need to create another vnet - one for each region. Refer [here](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview#supported-regions) to see which Azure region your Power Platform geography is mapped with. In my example, as Environments in Australia are mapped to **australiaeast** and **australiasoutheast**, I will need to create two VNet - one for each region.
   
   <img width="900" alt="image" src="https://github.com/user-attachments/assets/918a9948-4ff0-4b14-be14-a8a9973f5b8f" />


5. In the VNet resources, you will need to create a subnet that will be delegated for Power Platform. In the primary Vnet (the one that is on the same region as your Power Platform Environment - this my example, this is in "australiasoutheast"), we need at least **two** subnets. We will need Subnet1 (in this example below: "fsdemomcp-subnet") for our MCP Server in Azure Function, and another Subnet2 ("pp-vnet") for Power Platform to use.
   
   <img width="808" height="370" alt="image" src="https://github.com/user-attachments/assets/26ba89b9-77d9-445f-bcb4-964ea05e56cb" />


> [!IMPORTANT]
> Subnet size matters! Refer [here](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview#estimating-subnet-size-for-power-platform-environments) for a guide on estimating subnet sizes for your vnet. Rule of thumb: Plan for 25-30 IP addresses for typical production environment. Do not share the Production Environment's Vnet policy with other environments.
>

5. This is a very important step - for the Power Platform delegated subnet, it must not be shared or used by other purposes. In the second subnet "pp-vnet", set **"Subnet Delegation"** to **"Microsoft.PowerPlatform/enterprisePolicies"**. This allows Power Platform to manage this subnet - and at runtime, it will run containers in this delegated subnet, which in turn will be able to connect to resources within the same vnet.
   
   <img width="800" alt="image" src="https://github.com/user-attachments/assets/1d37ba8b-1639-4150-bd8d-d23d3c5a3893" />

7. Next, we head back into our HelloWorldMCPDemo Azure Function app. Under Settings -> Network, click on the private endpoints to create one.
   
   <img width="1001" height="520" alt="image" src="https://github.com/user-attachments/assets/b6290a29-ea57-4cf9-a55a-77cd19836303" />

9. Click "+ Add", you may use either Express or Advanced method - give it a friendly name, and select the Vnet that we have created in Step 3, and the subnet created in Step 5. **DO NOT** use the subnet that has been delegated for Power Platform.
    
   <img width="788" height="288" alt="image" src="https://github.com/user-attachments/assets/8edcb20a-848d-4fe5-88cb-7a6081f81eb2" />

11. Our MCP Server function app now has a private end point inside this vnet. Click on the Private endpoint name.
    
   <img width="719" height="303" alt="image" src="https://github.com/user-attachments/assets/b2b2c74b-32cd-4e64-9aef-18a04332b12c" />

13. In the Private endpoint settings, go to **"DNS configuration"**. Note that we now have a Private DNS zone created. Click on the private DNS Zone ("privatelink.azurewebsites.net").
    
    <img width="1038" height="556" alt="image" src="https://github.com/user-attachments/assets/4090c734-bf70-49e3-b20b-91f7f7f0229d" />

15. In the Private DNS Zone, under **DNS Management** click on **"Virtual Network Links"**. We will need to link the vnet in our other region to this zone. Click **"+ Add"**.
    
    <img width="999" height="341" alt="image" src="https://github.com/user-attachments/assets/69c679f0-f8cb-4845-80a7-b35d7748d028" />

17. Provide a friendly name for the link, and select the other subnet - recall that in this example my Power Platform Environment is in Australia geography, which has two Azure region - australiaeast and australiasoutheast. I have created two vnets in each region - the function app and its private end point, the private DNS Zone is in southeast (matching my Environment's region). We will link the subnet in australiaeast to this Private DNS Zone. This is referenced [here](https://learn.microsoft.com/en-us/troubleshoot/power-platform/administration/virtual-network?toc=%2Fpower-platform%2Fadmin%2Ftoc.json&bc=%2Fpower-platform%2Fbreadcrumb%2Ftoc.json#azure-resource-with-a-private-endpoint).
    
    <img width="800" height="668" alt="image" src="https://github.com/user-attachments/assets/4145ee69-9e59-4544-9f06-394c95aa6f06" />

18. Now we have a private end point enabled for our MCP Server in Azure Function, we will be able to reach this within the same VNet. We can test this quickly by using a VM or a Container in the same VNet and use nslookup against your Function app domain. Example below shows an nslookup from a Container within the same vnet. (Note that Server:Unknown here simply means we don't have a PTR record, and this is using Azure DNS). We can see the nslookup resolves to 10.2.2.7 in my example here, which corresponds to the private end point for the MCP Server.
    
    <img width="640" height="493" alt="image" src="https://github.com/user-attachments/assets/509fc363-ea81-4079-a1a5-95ed8d6aecd8" />
    <img width="656" height="344" alt="image" src="https://github.com/user-attachments/assets/e5071f7f-2c11-4f43-a37f-c0c9c1fdae6c" />

19. Next step is to create an Enteprise Policy and associate our Power Platform to this Policy. For this step, you will need to grab your Azure Subscription ID (GUID), the resource group name where your VNet is created in, and the Resource ID for all the Vnets (one per each region). In the Vnet resource, you can copy the Subscription ID and click on JSON View to copy the Resource ID.
    
    <img width="900" height="220" alt="image" src="https://github.com/user-attachments/assets/a7e0741b-6fdb-411b-a6e0-e15c00b8038e" />
    <img width="785" height="237" alt="image" src="https://github.com/user-attachments/assets/024a3f54-5e77-49da-8ef4-564637b5d494" />

20. Use PowerShell and run the following command:
```PowerShell
Install-Module Microsoft.PowerPatform.EnterprisePolicies
Import-Module Microsoft.PowerPlatform.EnterprisePolicies
New-SubnetInjectionEnterprisePolicy -SubscriptionId "YourAzureSubscriptionId" -ResourceGroupName "YourAzureResourceGroupName" -PolicyName "giveThePolicyAName" -PolicyLocation "australia" -VirtualNetworkId "resourceIdForVNet1" -SubnetName "pp-vnet" -VirtualNetworkId2 "ResourceIdForVNet2" -SubnetName2 "pp-vnet"
```
> [!NOTE]
> On the parameters for the PowerShell command **New-SubnetInjectionEnterprisePolicy"**
>
> - SubscriptionId - this will be a GUID value as copied in Step 13 above.
> - ResourceGroupName - this will be the Azure Resource Group that house your Vnet, it will be a string value of its display name.
> - PolicyName - this will be a string value, a given name for your policy (eg PowerPlatformVNetPolicyTest).
> - VirtualNetworkId, VirutalNetworkId2 - this will be the resourceId for your VNet (the first one would be the VNet that is in the same region as your Power Platform Environment).
> - SubnetName, SubnetName2 - the name of the subnet that has been delegated for Power Platform from step 5 above in each VNet that corresponds to your regions.
> - (Optional) - if you have multiple subscriptions with multiple logins like me, you will need to add **-ForceAuth** parameter to force a picker. 

15. Run **Get-SubnetInjectionEnterprisePolicy -SubscriptionId "YourSubscriptionId" (optional -ForceAuth) to check if everything looks good. You can copy the ResourceId for your Enterprise Policy.
    
    <img width="856" height="235" alt="image" src="https://github.com/user-attachments/assets/91632321-c2c8-412b-8bf0-3f92b944eee8" />

If you need to remove the Enterprise Policy, run Remove-SubnetInjectionPolicy
```PowerShell
Remove-SubnetInjectionPolicy -PolicyResourceId "yourEnterprisePolicyResourceId"
```

16. Once the policy has been created, we can add an Environment into this policy. We can do this either through PowerShell or via the Power Platform Admin Center.
    If using PowerShell - use **Enable-SubnetInjection** to add the Environment to the policy.
```PowerShell
Enable-SubnetInjection -EnvironmentId "YourEnvironmentId" -PolicyArmId "yourEnterprisePolicyResourceId"
```

To remove the environment, use **Disable-SubnetInjection**
```PowerShell
Disable-SubnetInjection -ENvironmentId "YourEnvironmentId"
```

If using PPAC - navigate to "Security" -> "Data and privacy" -> "Azure Virtual Network polices".
<img width="900" height="416" alt="image" src="https://github.com/user-attachments/assets/889b106e-6596-4f73-9bf2-b423ef3f1f01" />

Select the desired Power Platform and click "Next". We should now see the Enterprise Policy name we have just created available to be assigned.

<img width="900" height="421" alt="image" src="https://github.com/user-attachments/assets/c8983add-2ad9-4028-8b6a-428123424b80" />

Give it a few seconds. Once the Environment has been assigned successfully, refresh the page and you should see the Policy name associated to the Environment.

<img width="689" height="250" alt="image" src="https://github.com/user-attachments/assets/ab4bc987-383f-40d7-b487-25c927653540" />

17. If your MCP tool was created ~before~ the Enterprise policy was assigned to the Environment, you may need to re-save the underlying custom connector. Go to your Custom Connector, edit and click "Update custom connector".
    
    <img width="818" height="225" alt="image" src="https://github.com/user-attachments/assets/487f1663-07eb-4696-bfeb-81b6b693cfa5" />

18. Now we are ready to re-try in Copilot Studio. If we refresh the tool list, we should now see that it is able to connect and resolve the tool list. Likewise, a prompt in the test pane should also demonstrate that we are getting response from the MCP tool. Note that your connection will likely been stale at this point, you will be prompted to open connection manager to reselect the connnection and retry.
    
    <img width="800" height="505" alt="image" src="https://github.com/user-attachments/assets/48a6e9fe-4b90-4278-b0bb-f29f35c6c654" />

19. If you head back into the Function App for the MCP Server, we can confirm this has public network access disabled, and it is using a private end point!
    
    <img width="900" height="404" alt="image" src="https://github.com/user-attachments/assets/d5e362a6-8801-434c-a7b4-52f53036efe9" />

20. Be aware that once you have assigned your Power Platform Environment to an Enteprrise Policy, all supported connectors will be using the delegated VNet. If you need to connect to internet resources as well using the same set of connectors in the same environment, you will need to configurea additional resources for your Vnet - including your Network Security Group and NAT Gateway. Please refer to the [Virtual Network support white paper](https://learn.microsoft.com/en-us/power-platform/admin/virtual-network-support-whitepaper#configuration-considerations) for more information.

References:

    
 
    




