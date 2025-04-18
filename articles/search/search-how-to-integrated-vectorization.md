---
title: Integrated Vectorization Using REST APIs or Python
titleSuffix: Azure AI Search
description: Learn how to use skills to automate data chunking and vectorization during indexing and query execution.
manager: nitinme
author: haileytap
ms.author: haileytapia
ms.service: azure-ai-search
ms.topic: how-to
ms.date: 04/18/2025
---

# Set up integrated vectorization in Azure AI Search using REST or Python

In this article, you learn how to use an indexer and a skillset to chunk, vectorize, and index content from a [supported data source](#supported-data-sources). The skillset calls the [Text Split skill](cognitive-search-skill-textsplit.md) or [Document Layout skill](cognitive-search-skill-document-intelligence-layout.md) for chunking and an embedding skill that's attached to a [supported embedding model](#supported-embedding-models) for chunk vectorization.

This article describes the end-to-end workflow for [integrated vectorization](vector-search-integrated-vectorization.md) using REST and Python. For portal-based instructions, see [Quickstart: Vectorize text and images in the Azure portal](search-get-started-portal-import-vectors.md).

## Prerequisites

+ An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

+ An [Azure AI Search service](search-create-service-portal.md). We recommend the Basic tier or higher.

+ A [supported data source](#supported-data-sources).

+ A [supported embedding model](#supported-embedding-models).

+ Completion of [Quickstart: Connect without keys](search-get-started-rbac.md) and [Configure a system-assigned managed identity](search-howto-managed-identities-data-sources.md#create-a-system-managed-identity). Although you can use key-based authentication for data plane operations, this article assumes [roles and managed identities](#role-based-access), which are more secure.

+ [Visual Studio Code](https://code.visualstudio.com/download) with a [REST client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) or the [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python) and [Jupyter package](https://pypi.org/project/jupyter/).

### Supported data sources

Integrated vectorization works with [all supported data sources](search-indexer-overview.md#supported-data-sources). However, this article focuses on the most commonly used data sources, which are described in the following table.

| Supported data source | Description |
|--|--|
| [Azure Blob Storage](search-howto-indexing-azure-blob-storage.md) | This data source works with blobs and tables. You must use a standard performance (general-purpose v2) account. Access tiers can be hot, cool, or cold. |
| [Azure Data Lake Storage (ADLS) Gen2](/azure/storage/blobs/create-data-lake-storage-account) | This is an Azure Storage account with a hierarchical namespace enabled. To confirm that you have Data Lake Storage, check the **Properties** tab on the **Overview** page.<br><br> :::image type="content" source="media/search-how-to-integrated-vectorization/data-lake-storage-account.png" alt-text="Screenshot of an Azure Data Lake Storage account in the Azure portal." border="true" lightbox="media/search-how-to-integrated-vectorization/data-lake-storage-account.png"::: |
| [OneLake](search-how-to-index-onelake-files.md) | This data source is currently in preview. For information about limitations and supported shortcuts, see [OneLake indexing](search-how-to-index-onelake-files.md). |

### Supported embedding models

Use an embedding model on an Azure AI platform in the [same region as Azure AI Search](search-create-service-portal.md#regions-with-the-most-overlap). For deployment instructions, see [Prepare your embedding model](#prepare-your-embedding-model).

| Provider | Supported models |
|--|--|
| [Azure OpenAI Service](https://aka.ms/oai/access) <sup>1, 2</sup> | text-embedding-ada-002<br>text-embedding-3-small<br>text-embedding-3-large |
| [Azure AI services multi-service resource](/azure/ai-services/multi-service-resource#azure-ai-services-resource-for-azure-ai-search-skills) <sup>3</sup> | For text and images:<br>[Azure AI Vision multimodal](/azure/ai-services/computer-vision/how-to/image-retrieval) (available in [select regions](/azure/ai-services/computer-vision/overview-image-analysis#region-availability))</li> |
| [Azure AI Foundry model catalog](/azure/ai-foundry/what-is-azure-ai-foundry) | For text:<br>Cohere-embed-v3-english<br>Cohere-embed-v3-multilingual<br><br>For images:<br>Facebook-DinoV2-Image-Embeddings-ViT-Base<br>Facebook-DinoV2-Image-Embeddings-ViT-Giant |

<sup>1</sup> If you're using Azure OpenAI Service, the endpoint must have a [custom subdomain](/azure/ai-services/cognitive-services-custom-subdomains), such as `https://my-unique-name.cognitiveservices.azure.com`. If you created your service in the [Azure portal](https://portal.azure.com/), this subdomain was automatically generated during service setup. Ensure that your service has a custom subdomain before you use it with the Azure AI Search integration.

<sup>2</sup> Azure OpenAI Service resources (with access to embedding models) that were created in the [Azure AI Foundry portal](https://ai.azure.com/) aren't supported. Only Azure OpenAI Service resources created in the Azure portal are compatible with the [Azure OpenAI Embedding skill](cognitive-search-skill-azure-openai-embedding.md) integration.

<sup>3</sup> For billing purposes, you must [attach your Azure AI services multi-service resource](cognitive-search-attach-cognitive-services.md) to the skillset in your Azure AI Search service. Unless you use a [keyless connection (preview)](cognitive-search-attach-cognitive-services.md#bill-through-a-keyless-connection) to create the skillset, both resources must be in the same region.

### Role-based access

You can use Microsoft Entra ID with role assignments or key-based authentication with full-access connection strings. For Azure AI Search service connections to other resources, we recommend role assignments.

To configure role-based access for integrated vectorization:

1. On your search service, [enable roles](search-security-enable-roles.md) and [configure a system-assigned managed identity](search-howto-managed-identities-data-sources.md#create-a-system-managed-identity).

1. On your data source platform and embedding model provider, create role assignments that allow your search service to access data and models. See [Prepare your data](#prepare-your-data) and [Prepare your embedding model](#prepare-your-embedding-model).

> [!NOTE]
> Free search services support role-based connections to Azure AI Search. However, they don't support managed identities on outbound connections to Azure Storage or Azure AI Vision. This lack of support requires key-based authentication on connections between free search services and other Azure resources.
>
> For more secure connections, use the Basic tier or higher. You can then enable roles and configure a managed identity for authorized access.

## Get connection information for Azure AI Search

In this section, you retrieve the endpoint and Microsoft Entra token for your Azure AI Search service. Both values are necessary to establish connections in REST and Python requests.

> [!TIP]
> The following steps assume that you're using [role-based access](#role-based-access) for local proof-of-concept testing. However, if you want to use integrated vectorization for app development, see [Connect your app to Azure AI Search using identities](keyless-connections.md).

1. Sign in to the [Azure portal](https://portal.azure.com/) and select your Azure AI Search service.

1. To obtain your search endpoint, copy the URL on the **Overview** page. An example search endpoint is `https://my-service.search.windows.net`.

1. To obtain your Microsoft Entra token, run the following command on your local system. This step requires completion of [Quickstart: Connect without keys](search-get-started-rbac.md).

   ```Azure CLI
   az account get-access-token --scope https://search.azure.com/.default --query accessToken --output tsv
   ```

## Prepare your data

In this section, you prepare your data for integrated vectorization by uploading files to a [supported data source](#supported-data-sources), assigning roles, and obtaining connection information.

### [Azure Blob Storage](#tab/prepare-data-storage)

1. Sign in to the [Azure portal](https://portal.azure.com/) and select your Azure Storage account.

1. From the left pane, select **Data Storage** > **Containers**.

1. Create a container or select an existing container, and then upload your files to the container.

1. To assign roles:

   1. From the left pane, select **Access Control (IAM)**.

   1. Select **Add** > **Add role assignment**.

   1. Under **Job function roles**, select **[Storage Blob Data Reader](search-howto-managed-identities-data-sources.md#assign-a-role)**, and then select **Next**.

   1. Under **Members**, select **Managed identity**, and then select **Select members**.

   1. Select your subscription and the managed identity of your search service.

1. To obtain a connection string:

   1. From the left pane, select **Security + networking** > **Access keys**.

   1. Copy either connection string, which you specify later in [Set variables](#set-variables).

1. (Optional) Synchronize deletions in your container with deletions in the search index. To configure your indexer for deletion detection:

   1. [Enable soft delete](/azure/storage/blobs/soft-delete-blob-enable?tabs=azure-portal#enable-blob-soft-delete-hierarchical-namespace) on your storage account. If you're using [native soft delete](search-howto-index-changed-deleted-blobs.md#native-blob-soft-delete), the next step isn't required.

   1. [Add custom metadata](search-howto-index-changed-deleted-blobs.md#soft-delete-strategy-using-custom-metadata) that an indexer can scan to determine which blobs are marked for deletion. Give your custom property a descriptive name. For example, you can name the property "IsDeleted" and set it to false. Repeat this step for every blob in the container. When you want to delete the blob, change the property to true. For more information, see [Change and delete detection when indexing from Azure Storage](search-howto-index-changed-deleted-blobs.md).

### [ADLS Gen2](#tab/prepare-data-adlsgen2)

1. Sign in to the [Azure portal](https://portal.azure.com/) and select your Azure Storage account.

1. From the left pane, select **Data Storage** > **Containers**.

1. Create a container or select an existing container, and then upload your files to the container.

1. To assign roles:

   1. From the left pane, select **Access Control (IAM)**.

   1. Select **Add** > **Add role assignment**.

   1. Under **Job function roles**, select **[Storage Blob Data Reader](search-howto-managed-identities-data-sources.md#assign-a-role)**, and then select **Next**.

   1. Under **Members**, select **Managed identity**, and then select **Select members**.

   1. Select your subscription and the managed identity of your search service.

1. To obtain a connection string:

   1. From the left pane, select **Security + networking** > **Access keys**.

   1. Copy either connection string, which you specify later in [Set variables](#set-variables).

1. (Optional) Synchronize deletions in your container with deletions in the search index. To configure your indexer for deletion detection:

   1. [Enable soft delete](/azure/storage/blobs/soft-delete-blob-enable?tabs=azure-portal#enable-blob-soft-delete-hierarchical-namespace) on your storage account.

   1. [Add custom metadata](search-howto-index-changed-deleted-blobs.md#soft-delete-strategy-using-custom-metadata) that an indexer can scan to determine which blobs are deleted. Give your custom property a descriptive name. For example, you can name the property "IsDeleted" and set it to false. Repeat this step for every blob in the container. When you want to delete the blob, change the property to true. For more information, see [Change and delete detection when indexing from Azure Storage](search-howto-index-changed-deleted-blobs.md).

### [OneLake](#tab/prepare-data-onelake)

1. Sign in to [Power BI](https://powerbi.com/) and [create a workspace](/fabric/data-engineering/tutorial-lakehouse-get-started).

1. From the left pane, select your new workspace.

1. To assign roles to your workspace:

   1. In the upper-right corner, select **Manage access**.

   1. Select **Add people or groups**.

   1. Enter the name of your search service. For example, if the URL is `https://my-demo-service.search.windows.net`, the service name is `my-demo-service`.

   1. Select a role. The default is **Viewer**, but you need **Contributor** to pull data into a search index.

1. To create a lakehouse and upload your data:

   1. In the upper-left corner, select **New item**.

   1. Select the **Lakehouse** tile.

   1. Enter a name for your lakehouse, and then select **Create**.

   1. On the **Home** tab of your lakehouse, select **Upload files**.

1. To obtain connection IDs:

   1. At the top of your browser, locate the lakehouse URL, which has the following format: `https://msit.powerbi.com/groups/00000000-0000-0000-0000-000000000000/lakehouses/11111111-1111-1111-1111-111111111111?experience=power-bi`.

   1. Copy the workspace ID, which is listed after "groups" in the URL. You specify this ID later in [Set variables](#set-variables). In our example, the workspace ID is `00000000-0000-0000-0000-000000000000`.

   1. Copy the lakehouse ID, which is listed after "lakehouses" in the URL. You specify this ID later in [Set variables](#set-variables). In our example, the lakehouse ID is `11111111-1111-1111-1111-111111111111`.

---

## Prepare your embedding model

In this section, you prepare your Azure AI resource for integrated vectorization by assigning roles, obtaining an endpoint, and deploying a [supported embedding model](#supported-embedding-models).

### [Azure OpenAI](#tab/prepare-model-aoai)

Azure AI Search supports text-embedding-ada-002, text-embedding-3-small, and text-embedding-3-large. Internally, Azure AI Search calls the [Azure OpenAI Embedding skill](cognitive-search-skill-azure-openai-embedding.md) to connect to Azure OpenAI.

1. Sign in to the [Azure portal](https://portal.azure.com/) and select your Azure OpenAI resource.

1. To assign roles:

   1. From the left pane, select **Access control (IAM)**.

   1. Select **Add** > **Add role assignment**.

   1. Under **Job function roles**, select **[Cognitive Services OpenAI User](/azure/ai-services/openai/how-to/role-based-access-control#azure-openai-roles)**, and then select **Next**.

   1. Under **Members**, select **Managed identity**, and then select **Select members**.

   1. Select your subscription and the managed identity of your search service.

1. To obtain an endpoint:

   1. From the left pane, select **Resource Management** > **Keys and Endpoint**.

   1. Copy the endpoint for your Azure OpenAI resource. You specify this URL later in [Set variables](#set-variables).

1. To deploy an embedding model:

   1. Sign in to the [Azure AI Foundry portal](https://ai.azure.com/) and select your Azure OpenAI resource.

   1. From the left pane, select **Model catalog**.

   1. Deploy a [supported embedding model](#supported-embedding-models).

   1. Copy the deployment and model names, which you specify later in [Set variables](#set-variables). The deployment name is the custom name you chose, while the model name is the model you deployed, such as `text-embedding-ada-002`.

### [Azure AI Vision](#tab/prepare-model-ai-vision)

Azure AI Search supports Azure AI Vision image retrieval through multimodal embeddings (version 4.0). Internally, Azure AI Search calls the [multimodal embeddings skill](cognitive-search-skill-vision-vectorize.md) to connect to Azure AI Vision.

1. Sign in to the [Azure portal](https://portal.azure.com/) and select your Azure AI services multi-service resource.

1. To assign roles:

   1. From the left pane, select **Access control (IAM)**.

   1. Select **Add** > **Add role assignment**.

   1. Under **Job function roles**, select **Cognitive Services User**, and then select **Next**.

   1. Under **Members**, select **Managed identity**, and then select **Select members**.

   1. Select your subscription and the managed identity of your search service.

1. To obtain an endpoint:

   1. From the left pane, select **Resource Management** > **Keys and Endpoint**.

   1. Copy the endpoint for your Azure AI services multi-services resource. You specify this URL later in [Set variables](#set-variables).

   > [!NOTE]
   > The multimodal embeddings are built into your Azure AI services multi-services resource, so there's no model deployment step.

### [Azure AI Foundry model catalog](#tab/prepare-model-catalog)

Azure AI Search supports Azure, Cohere, and Facebook embedding models in the [Azure AI Foundry](https://ai.azure.com/) model catalog, but it doesn't currently support the OpenAI CLIP models. Internally, Azure AI Search calls the [Azure Machine Learning (AML) skill](cognitive-search-aml-skill.md) to connect to the catalog.

For the model catalog, you should have an [Azure AI Foundry project](/azure/ai-foundry/how-to/create-projects) with a [hub that's connected to an Azure OpenAI resource and an Azure AI Search service](/azure/ai-foundry/how-to/create-projects#create-a-project).

1. Sign in to the [Azure portal](https://portal.azure.com/) and select your Azure OpenAI resource.

1. To assign roles:

   1. From the left pane, select **Access control (IAM)**.

   1. Select **Add** > **Add role assignment**.

   1. Under **Job function roles**, select **Cognitive Services User**, and then select **Next**.

   1. Under **Members**, select **Managed identity**, and then select **Select members**.

   1. Select your subscription and the managed identity of your search service.

1. To deploy an embedding model:

   1. Sign in to the [Azure AI Foundry portal](https://ai.azure.com/) and select your project.

   1. From the left pane, select **Model catalog**.

   1. Deploy a [supported embedding model](#supported-embedding-models).

   1. Copy the deployment and model names, which you specify later in [Set variables](#set-variables). The deployment name is the custom name you chose, while the model name is the model you deployed, such as `Cohere-embed-v3-english`.

1. To obtain an endpoint:

   1. From the left pane, select **My assets** > **Models + endpoints**.

   1. Select the model you deployed.

   1. On the **Details** tab, copy the endpoint. You specify this URL later in [Set variables](#set-variables).

---

## Set variables

In this section, you specify the connection information for your Azure AI Search service, your [supported data source](#supported-data-sources), and your [supported embedding model](#supported-embedding-models).

### [REST](#tab/set-endpoints-rest)

1. In Visual Studio Code, paste the following placeholders into your `.rest` or `.http` file.

   ```HTTP
   @baseUrl = PUT-YOUR-SEARCH-SERVICE-URL-HERE
   @token = PUT-YOUR-MICROSOFT-ENTRA-TOKEN-HERE
   ```

1. Replace `@baseUrl` with the search endpoint and `@token` with the Microsoft Entra token you obtained in [Get connection information for Azure AI Search](#get-connection-information-for-azure-ai-search).

1. Depending on your data source, add the following variables.

   | Data source | Variables | Enter this information |
   |--|--|--|
   | Azure Blob Storage | `@storageConnectionString` and `@blobContainer` | The connection string and the name of the container you created in [Prepare your data](#prepare-your-data). |
   | ADLS Gen2 | `@storageConnectionString` and `@blobContainer` | The connection string and the name of the container you created in [Prepare your data](#prepare-your-data). |
   | OneLake | `@workspaceId` and `@lakehouseId` | The workspace and lakehouse IDs you obtained in [Prepare your data](#prepare-your-data). |

1. Depending on your embedding model provider, add the following variables.

   | Embedding model provider | Variables | Enter this information |
   |--|--|--|
   | Azure OpenAI | `@aoaiEndpoint`, `@aoaiDeploymentName`, and `@aoaiModelName` | The endpoint, deployment name, and model name you obtained in [Prepare your embedding model](#prepare-your-embedding-model). |
   | Azure AI Vision | `@aiMultiServiceEndpoint` | The endpoint you obtained in [Prepare your embedding model](#prepare-your-embedding-model). |
   | Azure AI Foundry model catalog | `@aoaiEndpoint`, `@aiFoundryDeploymentName`, and `@aiFoundryModelName` | The endpoint, deployment name, and model name you obtained in [Prepare your embedding model](#prepare-your-embedding-model). |

1. To verify the variables, send the following request.

   ```HTTP
   ### List existing indexes by name
   GET  {{baseUrl}}/indexes?api-version=2024-07-01  HTTP/1.1
     Content-Type: application/json
     Authorization: Bearer {{token}}
   ```

   A response should appear in an adjacent pane. If you have existing indexes, they're listed. Otherwise, the list is empty. If the HTTP code is `200 OK`, you're ready to proceed.

### [Python](#tab/set-endpoints-python)

1. In Visual Studio Code, paste the following placeholder into your Jupyter notebook.

   ```Python
   AZURE_SEARCH_SERVICE: str = "PUT YOUR SEARCH SERVICE URL HERE"
   ```

1. Replace `AZURE_SEARCH_SERVICE` with the endpoint you obtained in [Get connection information for Azure AI Search](#get-connection-information-for-azure-ai-search).

1. Depending on your data source, add the following variables.

   | Data source | Variables | Enter this information |
   |--|--|--|
   | Azure Blob Storage | `AZURE_STORAGE_CONNECTION` | The connection string you obtained in [Prepare your data](#prepare-your-data). |
   | ADLS Gen2 | `AZURE_STORAGE_CONNECTION` | The connection string you obtained in [Prepare your data](#prepare-your-data). |
   | OneLake | `XYZ` | ... |

1. Depending on your embedding model provider, add the following variables.

   | Embedding model provider | Variables | Enter this information |
   |--|--|--|
   | Azure OpenAI | `AZURE_OPENAI_ACCOUNT` and `AZURE_DEPLOYMENT_MODEL` | The endpoint, model name, and deployment name you obtained in [Prepare your embedding model](#prepare-your-embedding-model). |
   | Azure AI Vision | `XYZ` | ... |
   | Azure AI Foundry model catalog | `XYZ` | ... |

---

## Connect to your data

In this section, you connect to a [supported data source](#supported-data-sources) for indexer-based indexing. An indexer in Azure AI Search requires a data source that specifies the type, credentials, and container.

### [REST](#tab/connect-data-rest)

1. Use [Create Data Source](/rest/api/searchservice/data-sources/create) to define a data source that provides connection information during indexing.

   ```HTTP
   ### Create a data source
   POST {{baseUrl}}/datasources?api-version=2024-07-01  HTTP/1.1
     Content-Type: application/json
     Authorization: Bearer {{token}}

     {
       "name": "my-data-source",
       "type": "azureblob",
       "subtype": null,
       "credentials": {
           "connectionString": "{{storageConnectionString}}"
       },
       "container": {
           "name": "{{blobContainer}}",
           "query": null
       },
       "dataChangeDetectionPolicy": null,
       "dataDeletionDetectionPolicy": null
     }
   ```

1. Set `type` to your data source: `azureblob`, `azureadlsgen2`, or `onelake`.

1. If you're using OneLake, set `credentials.connectionString` to `ResourceId={{workspaceId}}` and `container.name` to `{{lakehouseId}}`.

### [Python](#tab/connect-data-python)

1. Define a data source that provides connection information during indexing.

   ```Python
   from azure.search.documents.indexes import SearchIndexerClient
   from azure.search.documents.indexes.models import (
       SearchIndexerDataContainer,
       SearchIndexerDataSourceConnection
   )

   # Create a data source 
   indexer_client = SearchIndexerClient(endpoint=AZURE_SEARCH_SERVICE, credential=credential)
   container = SearchIndexerDataContainer(name="PUT YOUR CONTAINER NAME OR LAKEHOUSE ID HERE")
   data_source_connection = SearchIndexerDataSourceConnection(
       name="mydatasource",
       type="azureblob",
       connection_string=AZURE_STORAGE_CONNECTION,
       container=container
   )
   data_source = indexer_client.create_or_update_data_source_connection(data_source_connection)

   print(f"Data source '{data_source.name}' created or updated")
   ```

1. Set `type` to your data source: `azureblob`, `azureadlsgen2`, or `onelake`.

1. If you're using OneLake, set `connection_string` to `XYZ` and...

---

## Create a skillset

In this section, you create a [skillset](cognitive-search-working-with-skillsets.md) that calls a built-in skill to chunk your content and an embedding skill to create vector representations of the chunks. The skillset is executed during indexing in a [later section](#create-an-indexer).

### Call a built-in skill to chunk your content

Partitioning your content into chunks helps you meet the requirements of your embedding model and prevents data loss due to truncation. For more information about chunking, see [Chunk large documents for vector search solutions](vector-search-how-to-chunk-documents.md).

For built-in data chunking, Azure AI Search offers the [Text Split skill](cognitive-search-skill-textsplit.md) and [Document Layout skill](cognitive-search-skill-document-intelligence-layout.md). The Text Split skill breaks text into sentences or pages of a particular length, while the Document Layout skill breaks content based on paragraph boundaries.

### [REST](#tab/built-in-skill-rest)

1. Use [Create Skillset](/rest/api/searchservice/skillsets/create) to define a skillset.

   ```HTTP
   ### Create a skillset
   POST {{baseUrl}}/skillsets?api-version=2024-07-01  HTTP/1.1
     Content-Type: application/json
     Authorization: Bearer {{token}}

     {
       "name": "my-skillset",
       "skills": []
     }
   ```

1. In the `skills` array, call the Text Split skill or Document Layout skill. You can paste one of the following definitions.

   ```HTTP
       "skills": [
        {
          "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
          "name": "my-text-split-skill",
          "textSplitMode": "pages",
          "maximumPageLength": 1000,
          "pageOverlapLength": 100,
          "maximumPagesToTake": 1,
          "defaultLanguageCode": "en",
          "inputs": [
           {
             "name": "text",
             "source": "/document/content",
             "inputs": []
           }
          ],
          "outputs": [
           {
             "name": "text_items",
             "targetName": "my_pages"
           }
          ],
        },
        {
          "@odata.type": "#Microsoft.Skills.Util.DocumentIntelligenceLayoutSkill",
          "name": "my-document-layout-skill",
          "context": "/document",
          "outputMode": "oneToMany",
          "markdownHeaderDepth": "h3"
          "inputs": [
           {
             "name": "file_data",
             "source": "/document/file_data"
           }
          ],
          "outputs": [
           {
             "name": "markdown_document",
             "targetName": "markdown_document"
           }
          ]
        }
       ]
   ```

   > [!NOTE]
   > The Document Layout skill is in public preview. If you want to call this skill, use a preview REST API.

1. If you're using Azure AI Vision, [attach your Azure AI services multi-service resource](cognitive-search-attach-cognitive-services.md) after the `skills` array.

   ```HTTP
       "skills": [],
       "cognitiveServices": {
         "@odata.type": "#Microsoft.Azure.Search.AIServicesByIdentity",
         "subdomainUrl": "{{aiMultiServiceEndpoint}}",
         "identity": null
        }

### [Python](#tab/built-in-skill-python)

---

### Call an embedding skill to vectorize the chunks

To vectorize your chunked content, the skillset needs an embedding skill that points to a [supported embedding model](#supported-embedding-models). This article covers three embedding skills:

+ [Azure OpenAI Embedding skill](cognitive-search-skill-azure-openai-embedding.md)
+ [Azure AI Vision skill](cognitive-search-skill-vision-vectorize.md)
+ [AML skill](cognitive-search-aml-skill.md) for the Azure AI Foundry model catalog

### [REST](#tab/embedding-skill-rest)

1. After the built-in chunking skill in the `skills` array, call the Azure OpenAI Embedding skill, Azure AI Vision skill, or AML skill. You can paste one of the following definitions.

   ```HTTP
        {
          "@odata.type": "#Microsoft.Skills.Text.AzureOpenAIEmbeddingSkill",
          "resourceUri": "{{aoaiEndpoint}}",
          "deploymentId": "{{aoaiDeploymentName}}",
          "modelName": "{{aoaiModelName}}",
          "dimensions": 1536,
          "inputs": [
            {
              "name": "text",
              "source": "/document/pages*"
            }
          ],
          "outputs": [
            {
              "name": "vector"
            }
          ]
        },
        {
          "@odata.type": "#Microsoft.Skills.Vision.VectorizeSkill",
          "context": "/document",
          "inputs": [
            {
              "name": "url",
              "source": "/document/metadata_storage_path"
            },
            {
              "name": "queryString",
              "source": "/document/metadata_storage_sas_token"
            }
          ],
          "outputs": [
            {
              "name": "vector"
            }
          ]
        },
        {
          "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
          "resourceId": "{{modelEndpoint}}",
          "context": "/document",
          "inputs": [
            {
              "name": "text",
              "source": "/document/content"
            }
          ],
          "outputs": [
            {
              "name": "detected_language_code"
            }
          ]
        }
   ```

   > [!NOTE]
   > The Azure AI Vision skill and AML skill (for indexer connections to the Azure AI Foundry model catalog) are in public preview. If you want to call these skills, use a preview REST API.

1. If you're using the Azure OpenAI Embedding skill, set `dimensions` to the [number of embeddings generated by your embedding model](cognitive-search-skill-azure-openai-embedding.md#supported-dimensions-by-modelname).

### [Python](#tab/embedding-skill-python)

---

## Create a vector index

In this section, you set up physical data structures on your Azure AI Search service by creating a [vector index](vector-store.md). The schema for a vector index requires the following:

+ Name
+ Key field (string)
+ One or more vector fields
+ Vector configuration

Vector fields store numerical representations of your chunked data. They must be searchable and retrievable, but they can't be filterable, facetable, or sortable. They also can't have analyzers, normalizers, or synonym map assignments.

In addition to vector fields, the sample index in the following steps contains nonvector fields for human-readable content. It's common to include plain-text equivalents of the content you want to vectorize. For more information, see [Create a vector index](vector-search-how-to-create-index.md).

### [REST](#tab/vector-index-rest)

1. Use [Create Index](/rest/api/searchservice/indexes/create) to define the schema for a vector index.

   ```HTTP
   ### Create a vector index
   POST  {{baseUrl}}/indexes?api-version=2024-07-01  HTTP/1.1
     Content-Type: application/json
     Authorization: Bearer {{token}}

     {
       "name": "my-vector-index",
       "fields": [],
       "vectorSearch": []
     }
   ```

1. Add a [vector search configuration](vector-search-how-to-create-index.md#add-a-vector-search-configuration) to the `vectorSearch` object.

   ```HTTP
       "vectorSearch": {
         "algorithms": [
           {
             "name": "hnsw-algorithm",
             "kind": "hnsw",
             "parameters": {
               "m": 4,
               "efConstruction": 400,
               "efSearch": 100,
               "metric": "cosine"
             }
           }
         ],
         "profiles": [
           {
             "name": "vector-profile-hnsw",
             "algorithm": "hnsw-algorithm"
           }
         ]
       }
   ```

   `vectorSearch.algorithms` specifies the algorithm used for indexing and querying vector fields, while `vectorSearch.profiles` links the algorithm configuration to a profile that you can assign to vector fields.

1. Depending on your embedding model, update `vectorSearch.algorithms.metric`. [Valid values for distance metrics](/rest/api/searchservice/indexes/create-or-update#vectorsearchalgorithmmetric) are `cosine`, `dotproduct`, `euclidean`, and `hamming`.

1. Add fields to the `fields` arrays. Include a key field for document identification, nonvector fields for human-readable content, and vector fields for embeddings.

   ```HTTP
       "fields": [
         {
           "name": "id",
           "type": "Edm.String",
           "key": true,
           "filterable": true
         },
         {
           "name": "title",
           "type": "Edm.String",
            "searchable": true,
            "filterable": true,
            "sortable": true,
            "retrievable": true
         },
         {
           "name": "titleVector",
           "type": "Collection(Edm.Single)",
            "searchable": true,
            "retrievable": true,
            "stored": true,
            "dimensions": 1536,
            "vectorSearchProfile": "hnsw-algorithm"
         },
         {
           "name": "content",
           "type": "Edm.String",
            "searchable": true,
            "retrievable": true
         },
         {
           "name": "contentVector",
           "type": "Collection(Edm.Single)",
            "searchable": true,
            "retrievable": false,
            "stored": false,
            "dimensions": 1536,
            "vectorSearchProfile": "hnsw-algorithm"
         }
       ]
   ```

1. Depending on your embedding skill, update the `dimensions` property for each vector field.

   | Embedding skill | Set `dimensions` to |
   |--|--|
   | Azure OpenAI | The [number of embeddings generated by your embedding model](cognitive-search-skill-azure-openai-embedding.md#supported-dimensions-by-modelname). |
   | Azure AI Vision | `1024` |
   | AML | The [number of embeddings generated by your embedding model](vector-search-vectorizer-azure-machine-learning-ai-studio-catalog.md#expected-field-dimensions). |

### [Python](#tab/vector-index-python)

---

## Add a vectorizer to the index

In addition to indexing time, you can use integrated vectorization at query time, where the embedding model that indexes your data can also decode a search string or image into a vector for vector search.

In this section, you enable vectorization at query time by [adding a vectorizer definition](vector-search-how-to-configure-vectorizer.md) to your index and listing the vectorizer in a vector profile.

### [REST](#tab/vectorizer-rest)

1. Use [Create or Update Index](/rest/api/searchservice/indexes/create-or-update) to add the [Azure OpenAI vectorizer](vector-search-vectorizer-azure-open-ai.md), [Azure AI Vision vectorizer](vector-search-vectorizer-ai-services-vision.md), or [Azure AI Foundry model catalog vectorizer](vector-search-vectorizer-azure-machine-learning-ai-studio-catalog.md) to your index. You can paste one of the following definitions.

   ```HTTP
       "vectorizers": [
        {
          "name": "my-openai-vectorizer",
          "kind": "azureOpenAI",
          "parameters": {
            "resourceUri": "{{aoaiEndpoint}}",
            "deploymentId": "{{aoaiDeploymentName}}",
            "modelName": "{{aoaiModelName}}"
          }
        }
        {
          "name": "my-ai-services-vision-vectorizer",
          "kind": "aiServicesVision",
          "parameters": {
            "resourceUri": "{{aiMultiServiceEndpoint}}",
            "modelVersion": "2023-04-15"
          }
        }
        {
          "name": "my-model-catalog-vectorizer",
          "kind": "aml",
          "parameters": {
              "uri": "{{aoaiEndpoint}}",
              "modelName": "{{aiFoundryModelName}}",
          }
        }
      ]
   ```

1. In `vectorSearch.profiles`, which you created in the previous step, specify your vectorizer.

   ```HTTP
         "profiles": [
           {
             "name": "vector-profile-hnsw",
             "algorithm": "hnsw-algorithm"
             "vectorizer":"my-open-ai-vectorizer"
           }
         ]
   ```

### [Python](#tab/vectorizer-python)

---

## Create an indexer

In this section, you create an [indexer](search-indexer-overview.md) to drive the entire vectorization pipeline, from data retrieval to skillset execution to indexing. We recommend that you [run the indexer on a schedule](search-howto-schedule-indexers.md) to process changes or documents that were missed due to throttling.

### [REST](#tab/indexer-rest)

Use [Create Indexer](/rest/api/searchservice/indexers/create) to define an indexer that executes the vectorization pipeline.

```HTTP
### Create an indexer
POST {{baseUrl}}/indexers?api-version=2024-07-01  HTTP/1.1
  Content-Type: application/json
  Authorization: Bearer {{token}}

  {
    "name": "my-indexer",
    "dataSourceName": "my-data-source",
    "targetIndexName": "my-vector-index",
    "skillsetName": "my-skillset",
    "schedule": {
      "interval": "PT2H"
    },
    "parameters": {
      "batchSize": null,
      "maxFailedItems": null,
      "maxFailedItemsPerBatch": null
    }
  }
```

### [Python](#tab/indexer-python)

---

## Confirm indexing with a vector query

In this section, you verify that your content was successfully indexed by [running vector queries](vector-search-how-to-query.md). If you have a raw vector, you can provide it in the query. If you added a vectorizer, the search engine will decode plain text or an image into a vector for query execution.

### [REST](#tab/vector-queries-rest)

### [Python](#tab/vector-queries-python)

---

## Related content

+ [Integrated vectorization sample](https://github.com/Azure/azure-search-vector-samples/blob/main/demo-python/code/integrated-vectorization/azure-search-integrated-vectorization-sample.ipynb)
