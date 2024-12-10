---
title: Generating embeddings for Semantic Kernel Vector Store connectors
description: Describes how you can generate embeddings to use with Semantic Kernel vector store connectors.
zone_pivot_groups: programming-languages
author: westey-m
ms.topic: conceptual
ms.author: westey
ms.date: 09/23/2024
ms.service: semantic-kernel
---

# Generating embeddings for Semantic Kernel Vector Store connectors

> [!WARNING]
> The Semantic Kernel Vector Store functionality is in preview, and improvements that require breaking changes may still occur in limited circumstances before release.

Semantic Kernel supports generating embeddings using many popular AI services out of the box.

::: zone pivot="programming-language-csharp"

These services can be constructed directly or added to a dependency injection container and resolved from there.

## Constructing an embedding generator

You can construct instances of the text embedding services provided by Semantic Kernel directly.
They all implement the `ITextEmbeddingGenerationService` interface.

```csharp
// Constructing an Azure Open AI embedding generation service directly.
ITextEmbeddingGenerationService azureOpenAITES = new AzureOpenAITextEmbeddingGenerationService(
    "text-embedding-ada-002",
    "https://{myservice}.openai.azure.com/",
    "apikey");

// Constructing an Olama embedding generation service directly.
ITextEmbeddingGenerationService olamaTES = new OllamaTextEmbeddingGenerationService(
    "mxbai-embed-large",
    new Uri("http://localhost:11434"));
```

You can also use helpers to register them with a dependency injection container.

```csharp
// Registering Google AI embedding generation service with a service collection.
var services = new ServiceCollection();
services.AddGoogleAIEmbeddingGeneration("text-embedding-004", "apiKey");

// Registering Mistral AI embedding generation service with the dependency injection container on
// the kernel builder.
var kernelBuilder = Kernel
    .CreateBuilder()
    .AddMistralTextEmbeddingGeneration("mistral-embed", "apiKey");
```

## Generating embeddings

To use the `ITextEmbeddingGenerationService` you created, just call the `GenerateEmbeddingAsync` method
on it.

Here is an example of generating embeddings when uploading records.

```csharp
public async Task GenerateEmbeddingsAndUpsertAsync(
    ITextEmbeddingGenerationService textEmbeddingGenerationService,
    IVectorStoreRecordCollection<ulong, Hotel> collection)
{
    // Upsert a record.
    string descriptionText = "A place where everyone can be happy.";
    ulong hotelId = 1;

    // Generate the embedding.
    ReadOnlyMemory<float> embedding =
        await textEmbeddingGenerationService.GenerateEmbeddingAsync(descriptionText);

    // Create a record and upsert with the already generated embedding.
    await collection.UpsertAsync(new Hotel
    {
        HotelId = hotelId,
        HotelName = "Hotel Happy",
        Description = descriptionText,
        DescriptionEmbedding = embedding,
        Tags = new[] { "luxury", "pool" }
    });
}
```

Here is an example of generating embeddings when searching.

```csharp
public async Task GenerateEmbeddingsAndSearchAsync(
    ITextEmbeddingGenerationService textEmbeddingGenerationService,
    IVectorStoreRecordCollection<ulong, Hotel> collection)
{
    // Upsert a record.
    string descriptionText = "Find me a hotel with happiness in mind.";

    // Generate the embedding.
    ReadOnlyMemory<float> searchEmbedding =
        await textEmbeddingGenerationService.GenerateEmbeddingAsync(descriptionText);

    // Search using the already generated embedding.
    List<VectorSearchResult<Hotel>> searchResult = await collection.VectorizedSearchAsync(searchEmbedding).ToListAsync();

    // Print the first search result.
    Console.WriteLine("Score for first result: " + searchResult.FirstOrDefault()?.Score);
    Console.WriteLine("Hotel description for first result: " + searchResult.FirstOrDefault()?.Record.Description);
}
```

> [!TIP]
> For more information on generating embeddings, refer to [Embedding generation in Semantic Kernel](./../ai-services/embedding-generation/index.md).

## Embedding dimensions

Vector databases typically require you to specify the number of dimensions that each vector has when creating the collection.
Different embedding models typically support generating vectors with different dimension sizes. E.g. Open AI `text-embedding-ada-002`
generates vectors with 1536 dimensions. Some models also allow a developer to choose the number of dimensions they want in the
output vector, e.g. Google `text-embedding-004` produces vectors with 768 dimension by default, but allows a developer to
choose any number of dimensions between 1 and 768.

It is important to ensure that the vectors generated by the embedding model have the same number of dimensions as the
matching vector in the database.

If creating a collection using the Semantic Kernel Vector Store abstractions, you need to specify the number of dimensions
required for each vector property either via annotations or via the record definition. Here are examples of both setting
the number of dimensions to 1536.

```csharp
[VectorStoreRecordVector(Dimensions: 1536)]
public ReadOnlyMemory<float>? DescriptionEmbedding { get; set; }
```

```csharp
new VectorStoreRecordVectorProperty("DescriptionEmbedding", typeof(float)) { Dimensions = 1536 }
```

> [!TIP]
> For more information on how to annotate your data model, refer to [defining your data model](./defining-your-data-model.md).
> [!TIP]
> For more information on creating a record definition, refer to [defining your schema with a record definition](./schema-with-record-definition.md).

::: zone-end
::: zone pivot="programming-language-python"

## Coming soon

More info coming soon.

::: zone-end
::: zone pivot="programming-language-java"

## Coming soon

More info coming soon.

::: zone-end