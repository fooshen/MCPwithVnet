# Connecting Copilot Studio to an MCP Server over vnet
This guide walks through how Copilot Studio can securely connect to an MCP server that sits behind a private endpoint inside an Azure Virtual Network. 
Microsoft Copilot Studio is built directly on the Microsoft Power Platform, which means it inherits the same enterprise‑grade security, governance, compliance, and networking controls that global organizations have already validated at scale. When Copilot Studio connects to MCP servers, it does so through Power Platform’s Custom Connector framework. MCP defines the protocol and tool semantics, but the transport, authentication, and governance all run through the connector layer. As a result, every MCP tool benefits from the same hardened controls enterprises rely on today: secure authentication flows, network isolation, DLP enforcement, ALM pipelines, and centralized admin governance. While this guide illustrates connecting to an MCP server, the same steps applies when configuring connection to any supported connectors in Power Platform, and applies equally to Power Apps and Power Automate as well.

## Reference documentations
- [Power Platform Virtual Network support Overview](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview)
- [Power Platform Virtual Network support Whitepaper](https://learn.microsoft.com/en-us/power-platform/admin/virtual-network-support-whitepaper)
- [Power Platform Virtual Network support Set Up Guide](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-setup-configure)
- [Troubleshooting tips](https://learn.microsoft.com/en-us/troubleshoot/power-platform/administration/virtual-network)

## Why This Matters
Modern agents increasingly need access to internal systems—inventory, finance, operations, knowledge bases, line‑of‑business APIs. But these systems often live inside VNets,  behind private endpoints. Exposing them publicly is not an option.

By placing the MCP server behind a private endpoint and using Power Platform’s VNet support, we get:

1. Zero Public Exposure
- Your MCP server never touches the public internet.
- Only Power Platform’s managed runtime—via your delegated subnet—can reach it.
 
2. Enterprise‑Grade Network Isolation
Traffic flows entirely through:
- Private endpoints
- Private DNS zones
- VNet‑to‑VNet routing (if needed)

## What is in this guide

- Deploying a sample MCP server in Azure Functions
- Securing it with a private endpoint
- Enabling Power Platform to reach it through a delegated VNet

## Pre-requisites / What you will need
- Visual Studio Code (optional, to create and deploy our sample MCP server)
- A Power Platform Environment
- An Azure Subscription in the same tenant as your Power Platform
- PowerShell (optional - to help troubleshoot)

## Before you start
1. [Create a Power Platform Environment](https://learn.microsoft.com/en-us/power-platform/admin/create-environment) if you don't already have one. You can create Production, Sandbox or Developer environments. Vnet is **not supported** for Trial environments.
2. Enable Managed Environment feature for your environment.

> [!IMPORTANT]
> Before you start, make sure you have identified which Azure Region your Power Platform Environment is in.
> We can do this using PowerShell [Get-EnvironmentRegion](https://learn.microsoft.com/en-us/powershell/module/microsoft.powerplatform.enterprisepolicies/get-environmentregion)
> Alternately, you can go to the [maker portal](https://make.powerapps.com), navigate to "Azure Synapse Link" (or click on "More", "Discover All" and look for "Azyre Synapse Link" if it is not pinned on your navigation bar). Click on "New Link" and this will show your current Azure region.
> <img width="875" height="145" alt="image" src="https://github.com/user-attachments/assets/8fc59206-c2e3-40d0-af2c-1fb0ed6d66d3" />
>
> In this example, my Environment is in Australia Southeast.
> Understand that Power Platform Environment can be in a **geography** that is mapped to one or more **Azure Regions**. For example, if your Environment is in Australia geo, your region can be either in Australia East or Australia Southeast. See [here](https://learn.microsoft.com/en-us/power-platform/admin/vnet-support-overview#supported-regions) for a list of mapped regions. If you ever need to move your existing Environment to another region, contact your Microsoft Support for assistance.
>
> If you need to specify the region when creating a new Environment, you may do so using PowerShell [New-AdminPowerAppEnvironment](https://learn.microsoft.com/en-us/powershell/module/microsoft.powerapps.administration.powershell/new-adminpowerappenvironment) with the [RegionName](https://learn.microsoft.com/en-us/powershell/module/microsoft.powerapps.administration.powershell/new-adminpowerappenvironment?view=pa-ps-latest#-regionname) parameter or using [Power Platform API](https://learn.microsoft.com/en-us/rest/api/power-platform/).
 

## Content
- [Introduction (this page)](readme.md)
- [Part 1 :Create and deploy a simple MCP server on Azure Function and testing in Copilot Studio](MCPServer.md)
