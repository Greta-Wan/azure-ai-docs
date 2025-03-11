---
title: 'Quickstart: Use the OpenAI Service image generation Go SDK'
titleSuffix: Azure OpenAI Service
description: Walkthrough on how to get started with Azure OpenAI image generation using the Go SDK. 
#services: cognitive-services
manager: nitinme
ms.service: azure-ai-openai
ms.topic: include
ms.date: 08/28/2023
---

Use this guide to get started generating images with the Azure OpenAI SDK for Go.

[Library source code](https://github.com/Azure/azure-sdk-for-go/tree/main/sdk/ai/azopenai) | [Package](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/ai/azopenai) | [Samples](https://github.com/Azure/azure-sdk-for-go/tree/main/sdk/ai/azopenai)

## Prerequisites

- An Azure subscription - <a href="https://azure.microsoft.com/free/cognitive-services" target="_blank">Create one for free</a>
- [Go 1.8+](https://go.dev/doc/install)
- An Azure OpenAI resource created in a supported region (see [Region availability](/azure/ai-services/openai/concepts/models#model-summary-table-and-region-availability)). For more information, see [Create a resource and deploy a model with Azure OpenAI](../how-to/create-resource.md).


## Setup

[!INCLUDE [get-key-endpoint](get-key-endpoint.md)]

[!INCLUDE [environment-variables](environment-variables.md)]


## Create a new Go application

Open the command prompt and navigate to your project folder. Create a new file *sample.go*.

## Install the Go SDK

Install the OpenAI Go SDK using the following command: 

```console
go get github.com/Azure/azure-sdk-for-go/sdk/ai/azopenai@latest
```

Or, if you use `dep`, within your repo run:

```console
dep ensure -add github.com/Azure/azure-sdk-for-go/sdk/ai/azopenai
```

## Generate images with DALL-E

Open *sample.go* in your preferred code editor.

Add the following code to your script:

```go
package main

import (
	"context"
	"fmt"
	"net/http"
	"os"

	"github.com/Azure/azure-sdk-for-go/sdk/ai/azopenai"
	"github.com/Azure/azure-sdk-for-go/sdk/azcore"
	"github.com/Azure/azure-sdk-for-go/sdk/azcore/to"
)

func main() {
	azureOpenAIKey := os.Getenv("AZURE_OPENAI_API_KEY")

	// Ex: "https://<your-azure-openai-host>.openai.azure.com"
	azureOpenAIEndpoint := os.Getenv("AZURE_OPENAI_ENDPOINT")

	if azureOpenAIKey == "" || azureOpenAIEndpoint == "" {
		fmt.Fprintf(os.Stderr, "Skipping example, environment variables missing\n")
		return
	}

	keyCredential := azcore.NewKeyCredential(azureOpenAIKey)

	client, err := azopenai.NewClientWithKeyCredential(azureOpenAIEndpoint, keyCredential, nil)

	if err != nil {
		// handle error
	}

	resp, err := client.GetImageGenerations(context.TODO(), azopenai.ImageGenerationOptions{
		Prompt:         to.Ptr("a painting of a cat in the style of Dali"),
		ResponseFormat: to.Ptr(azopenai.ImageGenerationResponseFormatURL),
	}, nil)

	if err != nil {
		// handle error
	}

	for _, generatedImage := range resp.Data {
		// the underlying type for the generatedImage is dictated by the value of
		// ImageGenerationOptions.ResponseFormat. In this example we used `azopenai.ImageGenerationResponseFormatURL`,
		// so the underlying type will be ImageLocation.

		resp, err := http.Head(*generatedImage.URL)

		if err != nil {
			// handle error
		}

		fmt.Fprintf(os.Stderr, "Image generated, HEAD request on URL returned %d\nImage URL: %s\n", resp.StatusCode, *generatedImage.URL)
	}
}
```

Run the script using the `go run` command:

```console
go run sample.go
```

## Output

The URL of the generated image is printed to the console.

```console
Image generated, HEAD request on URL returned 200
Image URL: <SAS URL>
```
> [!NOTE]
> The image generation APIs come with a content moderation filter. If the service recognizes your prompt as harmful content, it won't return a generated image. For more information, see the [content filter](../concepts/content-filter.md) article.

## Clean up resources

If you want to clean up and remove an Azure OpenAI resource, you can delete the resource or resource group. Deleting the resource group also deletes any other resources associated with it.

- [Azure portal](../../multi-service-resource.md?pivots=azportal#clean-up-resources)
- [Azure CLI](../../multi-service-resource.md?pivots=azcli#clean-up-resources)

## Next steps

* Explore the image generation APIs in more depth with the [DALL-E how-to guide](../how-to/dall-e.md).
* For more examples check out the [Azure OpenAI Samples GitHub repository](https://github.com/Azure/openai-samples).
