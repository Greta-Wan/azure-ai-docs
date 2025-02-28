---
title: Document Intelligence Studio
titleSuffix: Azure AI services
description: Learn how to set up Document Intelligence Studio to test Azure AI Document Intelligence features.
author: laujan
manager: nitinme
ms.service: azure-ai-document-intelligence
ms.topic: overview
ms.date: 02/27/2025
ms.author: lajanuar
monikerRange: '>=doc-intel-3.0.0'
---

<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD051 -->

# Studio experience for Document Intelligence

[!INCLUDE [applies to v4.0 v3.1 v3.0](includes/applies-to-v40-v31-v30.md)]

The studio is an online tool to visually explore, understand, train, and integrate features from the Document Intelligence service into your applications. The studio provides a platform for you to experiment with the different Document Intelligence models and sample returned data in an interactive manner without the need to write code. You can use the studio experience to:

* Learn more about the different capabilities in Document Intelligence.
* Use your Document Intelligence resource to test models on sample documents or upload your own documents.
* Experiment with different add-on and preview features to adapt the output to your needs.
* Train custom classification models to classify documents.
* Train custom extraction models to extract fields from documents.
* Get sample code for the language specific `SDKs` to integrate into your applications.

Currently, we're undergoing the migration of features from the [Document Intelligence Studio](https://documentintelligence.ai.azure.com/studio) to the new [Azure AI Foundry portal](https://ai.azure.com/explore/aiservices/vision). There are some differences in the offerings for the two studios, which determine the correct studio for your use case.

## Choosing the correct studio experience

There are currently two studios, the [Azure AI Foundry portal](https://ai.azure.com/explore/aiservices/vision) and the [Document Intelligence Studio](https://documentintelligence.ai.azure.com/studio) for building and validating  Document Intelligence models. As the experiences migrate to the new Azure AI Foundry portal, some experiences are available in both studios, while other experiences/models are only available in only one of the studios. To follow are a few guidelines for choosing the Studio experience for your needs. All of our [prebuilt models](overview.md#prebuilt-models) and [general extraction models](overview.md#document-analysis-models) are available on both studios.

### When to use [Document Intelligence Studio](https://documentintelligence.ai.azure.com/studio)

Document Intelligence Studio contains all features released on or before November 2024. For any of the v2.1, v3.0, v3.1 features, continue to use the Document Intelligence Studio. Studios provide a visual experience for labeling, training, and validating custom models. For custom document field extraction models, use the Document Intelligence Studio for template and neural models. Custom classification models can only be trained and used on Document Intelligence Studio. Use Document Intelligence Studio if you want to try out GA versions of the models from version v3.0, v3.1, and v4.0.

### When to use [Azure AI Foundry portal](https://ai.azure.com/explore/aiservices/vision)

Start with the new Azure AI Foundry and try any of the prebuilt document models from `2024-11-30` version including general extraction models like Read or Layout.

## Learn more about Document Intelligence Studio

Select the studio experience from the following tabs to learn more about each studio and how you can get started.

### [**Document Intelligence Studio**](#tab/di-studio)

> [!IMPORTANT]
>
> * Document Intelligence Studio has distinct URLs for sovereign cloud regions.
> * Azure for US Government: [Document Intelligence Studio (`Azure Fairfax`)](https://formrecognizer.appliedai.azure.us/studio)
> * Microsoft Azure operated by 21Vianet: [Document Intelligence Studio (`Azure China`)](https://formrecognizer.appliedai.azure.cn/studio)

The studio supports Document Intelligence v3.0 and later API versions for model analysis and custom model training. Previously trained v2.1 models with labeled data are supported, but not v2.1 model training. Refer to the [REST API migration guide](v3-1-migration-guide.md) for detailed information about migrating from v2.1 to v3.0.

Use the [Document Intelligence Studio quickstart](quickstarts/try-document-intelligence-studio.md) to get started analyzing documents with document analysis or prebuilt models. Build custom models and reference the models in your applications using one of the [language specific `SDKs`](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-3.0.0&preserve-view=true). 

## Prerequisites

To use Document Intelligence Studio, you need to acquire the following assets from the Azure portal:

* **An Azure subscription** - [Create one for free](https://azure.microsoft.com/free/cognitive-services/).

* **An Azure AI services or Document Intelligence resource**. Once you have your Azure subscription, create a [single-service](https://portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer) or [multi-service](https://portal.azure.com/#create/Microsoft.CognitiveServicesAIServices) resource, in the Azure portal to get your key and endpoint. Use the free pricing tier (`F0`) to try the service, and upgrade later to a paid tier for production.

> [!TIP]
> Create an Azure AI services resource if you plan to access multiple Azure AI services under a single endpoint/key. For Document Intelligence access only, create a Document Intelligence resource. You need a single-service resource if you intend to use [Microsoft Entra authentication](/azure/active-directory/authentication/overview-authentication).
>
> Document Intelligence now supports Azure Active Directory (Azure AD) token authentication in addition to local (key-based) authentication when accessing the Document Intelligence resources and storage accounts. Be sure to follow below instructions to set up correct access roles, especially if your resources are applied with `DisableLocalAuth` policy.

There are added prerequisites for using custom models in Document Intelligence Studio. Refer to the [documentation](quickstarts/studio-custom-project.md) for step by step guidance.

### Authorization policies

Your organization can opt to disable local authentication and enforce Microsoft Entra (formerly Azure Active Directory) authentication for Azure AI Document Intelligence resources and Azure blob storage.

* Microsoft Entra authentication requires that key based authorization is disabled. After key access is disabled, Microsoft Entra ID is the only available authorization method.

* Microsoft Entra allows granting minimum privileges and granular control for Azure resources.

For more information, *see* the following guidance:

  * [Disable local authentication for Azure AI Services](../disable-local-auth.md).
  * [Prevent Shared Key authorization for an Azure Storage account](/azure/storage/common/shared-key-authorization-prevent)
 
 > [!NOTE]
 > If local (key-based) authentication is disabled for your Document Intelligence service resource, be sure to obtain **Cognitive Services User** role and your Azure AD token to authenticate requests on Document Intelligence Studio. The **Contributor** role only allows you to list keys but doesn't give you permission to use the resource when key-access is disabled.

* **Designating role assignments**. Document Intelligence Studio basic access requires the [`Cognitive Services User`](/azure/role-based-access-control/built-in-roles/ai-machine-learning#cognitive-services-user) role. For more information, *see* [Document Intelligence role assignments](quickstarts/try-document-intelligence-studio.md#azure-role-assignments).

> [!IMPORTANT]
>
> * Make sure you have the **Cognitive Services User role**, and not the Cognitive Services Contributor role when setting up Entra authentication.
> * ✔️ **Cognitive Services User**: you need this role to Document Intelligence or Azure AI services resource to enter the analyze page.
> * ✔️ **Contributor**: you need this role to create resource group, Document Intelligence service, or Azure AI services resource.
> * In Azure context, Contributor role can only perform actions to control and manage the resource itself, including listing the access keys.
> * User accounts with a Contributor are only able to access the Document Intelligence service by calling with access keys. However, when setting up access with Entra ID, key-access is disabled and **Cognitive Service User** role is required for an account to use the resources.

### Authentication in Studio

Navigate to the [Document Intelligence Studio](https://formrecognizer.appliedai.azure.com/). If it's your first time logging in, a popup window appears prompting you to configure your service resource. In accordance with your organization's policy, you have one or two options:

* **Microsoft Entra authentication: access by Resource (recommended)**.

  * Choose your existing subscription.
  * Select an existing resource group within your subscription or create a new one.
  * Select your existing Document Intelligence or Azure AI services resource.

    :::image type="content" source="media/studio/configure-service-resource.png" alt-text="Screenshot of configure service resource form from the Document Intelligence Studio.":::

* **Local authentication: access by API endpoint and key**.

  * Retrieve your endpoint and key from the Azure portal.
  * Go to the overview page for your resource and select **Keys and Endpoint** from the left navigation bar.
  * Enter the values in the appropriate fields.

      :::image type="content" source="media/studio/keys-and-endpoint.png" alt-text="Screenshot of the keys and endpoint page in the Azure portal.":::

* After validating the scenario in the Document Intelligence Studio, use the [**C#**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-4.0.0&preserve-view=true), [**Java**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-4.0.0&preserve-view=true), [**JavaScript**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-4.0.0&preserve-view=true), or [**Python**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-4.0.0&preserve-view=true) client libraries or the [**REST API**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-4.0.0&preserve-view=true) to get started incorporating Document Intelligence models into your own applications.

### Document Intelligence model support

Use the help wizard, labeling interface, training step, and interactive visualizations to understand how each feature works.

* **Read**: Try out Document Intelligence's [Studio Read feature](https://documentintelligence.ai.azure.com/studio/read) with sample documents or your own documents and extract text lines, words, detected languages, and handwritten style if detected. To learn more, *see* [Read overview](prebuilt/read.md).

* **Layout**: Try out Document Intelligence's [Studio Layout feature](https://documentintelligence.ai.azure.com/studio/layout) with sample documents or your own documents and extract text, tables, selection marks, and structure information. To learn more, *see* [Layout overview](prebuilt/layout.md).

* **Prebuilt models**: Document Intelligence's prebuilt models enable you to add intelligent document processing to your apps and flows without having to train and build your own models. As an example, start with the [Studio Invoice feature](https://documentintelligence.ai.azure.com/studio/prebuilt?formType=invoice). To learn more, *see* [Models overview](model-overview.md).

* **Custom extraction models**: Document Intelligence's [Studio Custom models feature](https://documentintelligence.ai.azure.com/studio/custommodel/projects) enables you to extract fields and values from models trained with your data, tailored to your forms and documents. To extract data from multiple form types, create standalone custom models or combine two, or more, custom models and create a composed model. Test the custom model with your sample documents and iterate to improve the model. To learn more, *see* the [Custom models overview](train/custom-model.md).

* **Custom classification models**: Document classification is a new scenario supported by Document Intelligence. The document classifier API supports classification and splitting scenarios. Train a classification model to identify the different types of documents your application supports. The input file for the classification model can contain multiple documents and classifies each document within an associated page range. To learn more, *see* [custom classification models](train/custom-classifier.md).

* **Add-on Capabilities**: Document Intelligence supports more sophisticated analysis capabilities. These optional capabilities can be enabled and disabled in the studio using the `Analyze Options` button in each model page. There are four add-on capabilities available: `highResolution`, `formula`, `font`, and `barcode extraction` capabilities. To learn more, *see* [Add-on capabilities](concept-add-on-capabilities.md).

#### Try a Document Intelligence model

* Once your resource is configured, you can try the different models offered by Document Intelligence Studio. From the front page, select any Document Intelligence model to try using with a no-code approach.

* To test any of the document analysis or prebuilt models, select the model and use one of the sample documents or upload your own document to analyze. The analysis result is displayed at the right in the content-result-code window.

* Custom models need to be trained on your documents. See [custom models overview](train/custom-model.md) for an overview of custom models.

* After validating the scenario in the Document Intelligence Studio, use the [**C#**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-3.0.0&preserve-view=true), [**Java**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-3.0.0&preserve-view=true), [**JavaScript**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-3.0.0&preserve-view=true), or [**Python**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-3.0.0&preserve-view=true) client libraries or the [**REST API**](quickstarts/get-started-sdks-rest-api.md?view=doc-intel-3.0.0&preserve-view=true) to get started incorporating Document Intelligence models into your own applications.

#### View resource details

 To view resource details such as name and pricing tier, select the **Settings** icon in the top-right corner of the Document Intelligence Studio home page and select the **Resource** tab. If you have access to other resources, you can switch resources as well.

:::image type="content" source="media/studio/form-recognizer-studio-resource-page.png" alt-text="Screenshot of the studio settings page resource tab.":::

With Document Intelligence, you can quickly automate your data processing in applications and workflows, easily enhance data-driven strategies, and skillfully enrich document search capabilities.


### Troubleshooting

|Scenario     |Cause| Resolution|
|-------------|------|----------|
|You receive the error message</br> `Form Recognizer Not Found` when opening a custom project.|Your Document Intelligence resource, bound to the custom project was deleted or moved to another resource group.| There are two ways to resolve this problem: </br>&bullet; Re-create the Document Intelligence resource under the same subscription and resource group with the same name.</br>&bullet; Re-create a custom project with the migrated Document Intelligence resource and specify the same storage account.|
|You receive the error message</br> `PermissionDenied` when using prebuilt apps or opening a custom project.|The principal doesn't have access to API/Operation" when analyzing against prebuilt models or opening a custom project. It's likely the local (key-based) authentication is disabled for your Document Intelligence resource don't have enough permission to access the resource.|Reference [Azure role assignments](quickstarts/try-document-intelligence-studio.md#azure-role-assignments) to configure your access roles.|
|You receive the error message</br> `AuthorizationPermissionMismatch` when opening a custom project.|The request isn't authorized to perform the operation using the designated permission. It's likely the local (key-based) authentication is disabled for your storage account and you don't have the granted permission to access the blob data.|Reference [Azure role assignments](quickstarts/try-document-intelligence-studio.md#azure-role-assignments) to configure your access roles.|
|You can't sign in to Document Intelligence Studio and receive the error message</br> `InteractionRequiredAuthError:login_required:AADSTS50058:A silent sign-request was sent but no user is signed in`|It's likely that your browser is blocking third-party cookies so you can't successfully sign in.|To resolve, see [Manage third-party settings](#manage-third-party-settings-for-studio-access) for your browser.|

#### Manage third-party settings for Studio access

**Edge**:

* Go to **Settings** for Microsoft Edge
* Search for "**third*party**"
* Go to **Manage and delete cookies and site data**
* Turn off the setting of **Block third*party cookies**

**Chrome**:

* Go to **Settings** for Chrome
* Search for "**Third*party**"
* Under **Default behavior**, select **Allow third*party cookies**

**Firefox**:

* Go to **Settings** for Firefox
* Search for "**cookies**"
* Under **Enhanced Tracking Protection**, select **Manage Exceptions**
* Add exception for **`https://documentintelligence.ai.azure.com`** or the Document Intelligence Studio URL of your environment

**Safari**:

* Choose **Safari** > **Preferences**
* Select **Privacy**
* Deselect **Block all cookies**

### [**Azure AI Foundry portal**](#tab/ai-studio)

Document Intelligence is part of the Azure AI services offerings in the Azure AI Foundry portal. Each of the Azure AI services helps developers and organizations rapidly create intelligent, cutting-edge, market-ready, and responsible applications with out-of-the-box and prebuilt and customizable APIs and models.

Learn how to [connect your AI services hub](../../ai-studio/ai-services/how-to/connect-ai-services.md) with AI services and get started using Document Intelligence.

## Next steps

* Visit [Document Intelligence Studio](https://formrecognizer.appliedai.azure.com/studio).
* Visit [Azure AI Foundry portal](https://ai.azure.com/explore/aiservices/vision).
* Get started with [Document Intelligence Studio quickstart](quickstarts/try-document-intelligence-studio.md).
