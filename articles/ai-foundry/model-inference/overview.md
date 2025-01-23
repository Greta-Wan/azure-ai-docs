---
title: What is Azure AI model inference?
titleSuffix: Azure AI Foundry
description: Apply advanced language models to variety of use cases with Azure AI model inference.
manager: scottpolly
author: msakande
reviewer: santiagxf
ms.service: azure-ai-model-inference
ms.topic: concept-article
ms.date: 1/21/2025
ms.author: mopeakande
ms.reviewer: fasantia
ms.custom: generated
recommendations: false
---

# What is Azure AI model inference?

Azure AI model inference provides access to the most powerful models available in the Azure AI model catalog. The models come from key model providers in the industry, including OpenAI, Microsoft, Meta, Mistral, Cohere, G42, and AI21 Labs. These models can be integrated with software solutions to deliver a wide range of tasks that include content generation, summarization, image understanding, semantic search, and code generation.

Azure AI model inference provides a way to **consume models as APIs without hosting them on your infrastructure**. Models are hosted in a Microsoft-managed infrastructure, which enables API-based access to the model provider's model. API-based access can dramatically reduce the cost of accessing a model and simplify the provisioning experience.

Azure AI model inference is part of Azure AI Services and users can access the service through [REST APIs](../../ai-studio/reference/reference-model-inference-api.md), [SDKs in several languages](supported-languages.md) including Python, C#, JavaScript, and Java. It can also be used from [Azure AI Foundry by configuring a connection](how-to/configure-project-connection.md).

## Models

You can get access to the key model providers in the industry including OpenAI, Microsoft, Meta, Mistral, Cohere, G42, and AI21 Labs. Model providers define the license terms and set the price for use of their models. The following list shows all the models available:

> [!TIP]
> See the [Models](concepts/models.md) article for a detailed view of the models, capabilities, and details.

| Provider | Models |
| -------- | ------ |
| [AI21 Labs](concepts/models.md#ai21-labs) | - AI21-Jamba-1.5-Mini <br /> - AI21-Jamba-1.5-Large <br /> |
| [Azure OpenAI](concepts/models.md#azure-openai) | - o1 <br /> - gpt-4o <br /> - o1-preview <br /> - o1-mini <br /> - gpt-4o-mini <br /> - text-embedding-3-large <br /> - text-embedding-3-small <br /> |
| [Cohere](concepts/models.md#cohere) | - Cohere-embed-v3-english <br /> - Cohere-embed-v3-multilingual <br /> - Cohere-command-r-plus-08-2024 <br /> - Cohere-command-r-08-2024 <br /> - Cohere-command-r-plus <br /> - Cohere-command-r <br /> |
| [Core42](concepts/models.md#core42) | - jais-30b-chat <br /> |
| [Meta](concepts/models.md#meta) | - Llama-3.3-70B-Instruct <br /> - Llama-3.2-11B-Vision-Instruct <br /> - Llama-3.2-90B-Vision-Instruct <br /> - Meta-Llama-3.1-405B-Instruct <br /> - Meta-Llama-3-8B-Instruct <br /> - Meta-Llama-3.1-70B-Instruct <br /> - Meta-Llama-3.1-8B-Instruct <br /> - Meta-Llama-3-70B-Instruct <br /> |
| [Microsoft](concepts/models.md#microsoft) | - Phi-3-mini-128k-instruct <br /> - Phi-3-mini-4k-instruct <br /> - Phi-3-small-8k-instruct <br /> - Phi-3-medium-128k-instruct <br /> - Phi-3-medium-4k-instruct <br /> - Phi-3.5-vision-instruct <br /> - Phi-3.5-MoE-instruct <br /> - Phi-3-small-128k-instruct <br /> - Phi-3.5-mini-instruct <br /> - Phi-4 <br /> |
| [Mistral AI](concepts/models.md#mistral-ai) | - Ministral-3B <br /> - Mistral-large <br /> - Mistral-small <br /> - Mistral-Nemo <br /> - Mistral-large-2407 <br /> - Mistral-Large-2411 <br /> - Codestral-2501 <br /> |
| [NTT Data](concepts/models.md#ntt-data) | - Tsuzumi-7b |

## Pricing

Models that are offered by non-Microsoft providers (for example, Meta AI and Mistral models) are billed through Azure Marketplace. For such models, you're required to subscribe to the particular model offering in accordance with the [Microsoft Commercial Marketplace Terms of Use](/legal/marketplace/marketplace-terms). Users accept license terms for use of the models. Pricing information for consumption is provided during deployment.

Models that are offered by Microsoft (for example, Phi-3 models and Azure OpenAI models) don't have this requirement, and they're billed via Azure meters as First Party Consumption Services. As described in the [Product Terms](https://www.microsoft.com/licensing/terms/welcome/welcomepage), you purchase First Party Consumption Services by using Azure meters, but they aren't subject to Azure service terms.

> [!TIP]
> Learn how to [monitor and manage cost](how-to/manage-costs.md) in Azure AI model inference.

## Responsible AI

At Microsoft, we're committed to the advancement of AI driven by principles that put people first. Generative models such as the ones available in Azure AI models have significant potential benefits, but without careful design and thoughtful mitigations, such models have the potential to generate incorrect or even harmful content. 

Microsoft has made significant investments to help guard against abuse and unintended harm. These investments include:

- Incorporating Microsoft's [principles for responsible AI use](https://www.microsoft.com/ai/responsible-ai)
- Adopting a [code of conduct](/legal/cognitive-services/openai/code-of-conduct?context=/azure/ai-services/openai/context/context) for use of the service
- Building [content filters](/azure/ai-services/content-safety/overview) to support customers
- Providing responsible AI [information and guidance](/legal/cognitive-services/openai/transparency-note?context=%2Fazure%2Fai-services%2Fopenai%2Fcontext%2Fcontext&tabs=image) that customers should consider when using Azure OpenAI.

## Getting started

Azure AI Models is a new feature offering on Azure AI Services resources. You can get started with it the same way as any other Azure product where you [create and configure your resource for Azure AI model inference](how-to/quickstart-create-resources.md), or instance of the service, in your Azure Subscription. You can create as many resources as needed and configure them independently in case you have multiple teams with different requirements.

Once you create an Azure AI Services resource, you must deploy a model before you can start making API calls. By default, no models are available on it so you can control which ones to start from. See the tutorial [Create your first model deployment in Azure AI model inference](how-to/create-model-deployments.md).

## Next steps

- [Create your first model deployment in Azure AI model inference](how-to/create-model-deployments.md)
