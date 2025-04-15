---
title: 'How to use Custom Bing Search with Azure AI Agent Service'
titleSuffix: Azure OpenAI
description: Learn how to ground Azure AI Agents using Custom Bing Search results.
services: cognitive-services
manager: nitinme
ms.service: azure-ai-agent-service
ms.topic: how-to
ms.date: 04/15/2025
author: aahill
ms.author: aahi
ms.custom: azure-ai-agents
---

# Grounding with Bing Custom Search 

Grounding with Bing Custom Search tool allows your Azure AI Agents to search within a configurable set of public web domains. It allows you to define the parts of the web you want to draw from so users only see relevant results from the domains and subdomains of your choosing. You need to first create a Grounding with Bing Custom Search resource in Azure Portal. 

Once the resource is deployed, go to **Configurations** in the Azure AI Foundry portal, and create a configuration instance to include content only from websites that your users care about. Instructions about how to create your custom configuration can be found in the [setup section](#set-up).

Once the tool is configured, you can connect it to your Azure AI Agent. When a user sends a query, the agentw will decide if Grounding with Bing Custom Search should be leveraged or not. If so, it will use Bing to search over the domains you specify and return relevant chunks. Lastly, Azure AI Agents will use the returned chunks to generate a response to return to the user 

> [!IMPORTANT]
>  
> * Your usage of Grounding with Bing Custom Search can incur costs. See the pricing page for details. 
> * By creating and using a Grounding with Bing Custom Search resource through code-first experience, such as Azure CLI, or deploying through deployment template, you agree to be bound by and comply with the [terms of use](https://www.microsoft.com/bing/apis/grounding-legal), which may be updated periodically. 
> * When you use Grounding with Bing Custom Search, your customer data is transferred outside of the Azure compliance boundary to the Grounding with Bing Custom Search service. Grounding with Bing Custom Search is not subject to the same data processing terms (including location of processing) and does not have the same compliance standards and certifications as the Azure AI Agent Service, as described in the Grounding with Bing Custom Search Terms of Use. It is your responsibility to assess whether use of Grounding with Bing Custom Search in your agent meets your needs and requirements. 

## How Grounding with Bing Custom Search works 

The user query is the message that an end user sends to an agent, such as "what water bottle is available?" (assuming you create an instance to Contoso product websites). Instructions are the system message a developer can provide to share context and provide instructions to the AI model on how to use various tools or behave. 

When a user sends a query, the customer's AI model deployment first processes it (using the provided instructions) to later perform a Bing Custom Search query (which is visible to developers). Grounding with Bing Custom Search returns relevant search results to the customer's model deployment, which then generates the final output. 

> [!NOTE]
> When using Grounding with Bing Custom Search, only the Bing Custom Search query, the configuration instance, and your resource key are sent to Bing, and no end user-specific information is included. Your resource key is sent to Bing solely for billing and rate limiting purposes. 

The authorization will happen between Grounding with Bing Custom Search service and Azure AI Agent service. Any Bing Custom Search query that is generated and sent to Bing for the purposes of grounding is transferred, along with the resource key and configuration instance, outside of the Azure compliance boundary to the Grounding with Bing Custom Search service. Grounding with Bing Custom Search is subject to Bing's terms and do not have the same compliance standards and certifications as the Azure AI Agent Service, as described in the Grounding with Bing Custom Search Terms of Use. It is your responsibility to assess whether the use of Grounding with Bing Custom Search in your agent meets your needs and requirements. 

Developers and end users don't have access to raw content returned from Grounding with Bing Custom Search. The model response, however, includes citations with links to the websites used to generate the response and is allowed to be stored using the mechanisms provided by the Agents Service. You can retrieve the model response by accessing the data in the thread that was created. These references must be retained and displayed in the exact form provided by Microsoft, as per Grounding with Bing Custom Search's Use and Display Requirements. 

 

## Setup 

1. Create an Azure AI Agent by following the steps in the [quickstart](../../quickstart.md). 

1. Create a Grounding with Bing Custom Search resource in the [Azure portal](https://portal.azure.com/#create/Microsoft.BingGroundingCustomSearch). You need to have owner or contributor role in your subscription or resource group to create it. Make sure you create this Grounding with Bing Custom Search resource in the same resource group as your Azure AI Agent, AI Project, and other resources. 

1. After you have created a Grounding with Bing Custom Search resource, you can create a configuration instance.  

    1. Go to the Grounding with Bing Custom Search resource created 

    1. Select **Resource Management** on the left panel. 

    1. Select **Configurations**. 

    1. Select **Create a new configuration**. 

When you create or update a configuration, enter the following information: 

1. The name of your configuration. This can be changed at any time. 

1. Include URLs to define your search slices in at least one of the following fields: 

    > [!NOTE]
    > Grounding with Bing Custom Search only returns results for domains and webpages that are public and have been indexed by Bing. 
    > * Domain (for example, `https://www.microsoft.com`) 
    > * Domain and path (for example, `https://www.microsoft.com/surface`) 
    > * Webpage (for example, `https://www.microsoft.com/en-us/p/surface-earbuds/8r9cpq146064`) 

    1. Allowed domains to search against. 

    1. Blocked domains to exclude from the search space. 

    1. Pinned domains to prioritize when searching on multiple domains, for specific search queries. 


1. Determine if you want to include subpages. A subpage slice specifies a domain path. Bing searches all content found at and below the path. You may specify a maximum of two sub-folders in the path. For example, `www.microsoft.com/windows/`. 

1. Adjust ranking if needed 

1. Go to the [Azure AI Foundry portal](https://ai.azure.com/) and use the Grounding with Bing Custom Search tool in your agent. 

    1. Click the agent you created. 

    1. Click to add a knowledge tool. 

    1. Click the **Grounding with Bing Custom Search** tool  

    1. Click to create a new connection or use an existing connection 

    1. For new connection, select your Grounding with Bing Custom Search resource. 

    1. Once you have connected to a resource, select the configuration name. 

    1. Save the tool and start chatting with your agent. 

### Custom Search configuration capabilities 

Grounding with Bing Custom Search is a powerful tool that allows you to select a subspace of the web to limit your Agent’s grounding knowledge. Here you can find a few tips for how to take the maximum advantage of this capability: 

1. If you own a public site that you want to include in the search but Bing hasn’t indexed, see the [Bing webmaster documentation](https://www.bing.com/webmaster/help/webmaster-guidelines-30fba23a) for details about getting your site indexed. The webmaster documentation also provides details about getting Bing to crawl your site if the index is out of date. 

1. Any pinned domain is allowed by default and prioritized in the search slice. 

1. You can only block certain domains and perform a search against the rest of the Web (a competitor’s site, for example). 

 