# Connecting Copilot Studio to an MCP Server over vnet
This guide walks through how to build, secure, and integrate a Model Context Protocol (MCP) server hosted on Azure Functions with Copilot Studio—specifically when the function app is placed behind a private endpoint and accessed through Power Platform’s VNet connectivity. It provides a practical, end‑to‑end blueprint for teams that want to expose internal APIs as secure, governed tools inside Copilot Studio.

## What is in this guide

- Deploying a sample MCP server in Azure Functions
- Securing it with a private endpoint
- Enabling Power Platform to reach it through a delegated VNet

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

## Pre-requisites / What you will need
- Visual Studio Code (optional, to create and deploy our sample MCP server)
- A Power Platform Environment
- An Azure Subscription in the same tenant as your Power Platform
- PowerShell (optional - to help troubleshoot)

## Content

