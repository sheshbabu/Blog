---
title: Why use Azure OpenAI when you have OpenAI?
date: 2023-06-22 01:03:26
keywords: LLM, Azure OpenAI
image: /images/2023-why-use-azure-openai-when-you-have-openai/2023-why-use-azure-openai-when-you-have-openai.png
tags:
  - LLM
---

In addition to the [ChatGPT](https://chat.openai.com/) application, OpenAI has also released APIs that allow you to integrate it in your products. Confusingly, Microsoft has also released something called [Azure OpenAI Service](https://azure.microsoft.com/en-ca/products/cognitive-services/openai-service), which appears to be doing the same.

So, what’s the difference?

![](/images/2023-why-use-azure-openai-when-you-have-openai/2023-why-use-azure-openai-when-you-have-openai.png)

## Data Privacy

One of the biggest concerns people have with these LLM services is that they might use the company data provided by you to further train the model. This is problematic when you’re working with sensitive data from industries like healthcare, finance, etc. Both [Azure OpenAI Service](https://learn.microsoft.com/en-ca/azure/cognitive-services/openai/faq) and [OpenAI](https://platform.openai.com/docs/models/how-we-use-your-data) don’t use your data for training.

Another concern people have is whether their prompts or conversation history is being stored by these services. Both [Azure](https://learn.microsoft.com/en-ca/legal/cognitive-services/openai/data-privacy#how-does-the-azure-openai-service-process-data) and [OpenAI](https://platform.openai.com/docs/models/how-we-use-your-data) log your API requests and retain them for 30 days to identify abuse. However, you can opt-out of this from both services.

So, when it comes to data privacy, both Azure OpenAI Service and OpenAI are identical.

## Data Residency

When you’re working with cloud services, choosing the right region is very important. By selecting a region that’s closest to your customers, you can reduce the network latency and thereby provide better performance to your customers.

Being able to choose a region is also important for compliance reasons. If you work in regulated industries like healthcare, finance, etc, you might be required by local regulations to store your data in only a few geographical locations like Europe, US, etc.

Azure OpenAI Service is available in [multiple regions](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/concepts/models#model-summary-table-and-region-availability) like East US, South Central US, UK South, West Europe, France Central. OpenAI currently doesn’t support multiple regions.

## Infrastructure Security

Defense in depth is an important concept in security. Azure provides network security tools such as [VNets](https://learn.microsoft.com/en-us/azure/cognitive-services/cognitive-services-virtual-networks) and Private Link which can be used to add more security layers like network isolation, routing traffic through private backbone instead of public internet, using firewalls to limit inbound traffic only to certain IP addresses etc which can be used with Azure OpenAI Service.

In addition to the network security, access control is also an important part of infrastructure security - making sure that only the correct people can access the cloud resources. Azure provides tools such as Azure AD and Azure RBAC for handling this.

In comparison, OpenAI doesn’t have such sophisticated security tools.

## Integration with Microsoft ecosystem

This is one of the obvious benefits of using Azure OpenAI - close integration with other Microsoft/Azure products such as [SharePoint](https://learn.microsoft.com/en-gb/azure/search/search-howto-index-sharepoint-online), [Power Automate](https://learn.microsoft.com/en-us/ai-builder/azure-openai-model-pauto), [Azure SQL](https://learn.microsoft.com/en-gb/azure/search/search-howto-connecting-azure-sql-database-to-azure-search-using-indexers), [Azure Blob Storage](https://learn.microsoft.com/en-gb/azure/search/search-howto-indexing-azure-blob-storage) etc.

As most enterprises use Microsoft products such as SharePoint, O365 etc, building internal applications that leverage LLMs would be easier to accomplish using Azure OpenAI Service. OpenAI doesn’t have this level of integration.

## Model parity

OpenAI [recently released](https://openai.com/blog/function-calling-and-other-api-updates) new versions of GPT-4 and GPT-3.5 Turbo. The GPT-4 model (0613) introduced the function calling feature. GPT-3.5 Turbo also received an update (0613) which included function calling, a new 16k context variant, and cost reduction.

These changes have not propagated to [Azure OpenAI](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/concepts/models#model-summary-table-and-region-availability) yet.

## Pricing

We pay based on the type of the model, its max token limit and how many tokens we consume. Surprisingly, both [OpenAI](https://openai.com/pricing) and [Azure](https://azure.microsoft.com/en-gb/pricing/details/cognitive-services/openai-service/) have the same pricing model!

| Service      | Model         | Context | Input   | Output |
| ------------ | ------------- | ------- | ------- | ------ |
| OpenAI       | GPT-4         | 8K      | $0.03   | $0.06  |
| Azure OpenAI | GPT-4         | 8K      | $0.03   | $0.06  |
| OpenAI       | GPT-4         | 32K     | $0.06   | $0.12  |
| Azure OpenAI | GPT-4         | 32K     | $0.06   | $0.12  |
| OpenAI       | GPT-3.5 Turbo | 4K      | $0.0015 | $0.002 |
| Azure OpenAI | GPT-3.5 Turbo | 4K      | $0.002  | $0.002 |
| OpenAI       | GPT-3.5 Turbo | 16K     | $0.003  | $0.004 |
| Azure OpenAI | GPT-3.5 Turbo | 16K     | N/A     | N/A    |

The price is per 1000 tokens.

## Rate Limits

The API requests are rate limited in two ways - either by requests per minute (RPM) which might be familiar to you working with other APIs, and also tokens per minute (TPM). You hit these limits either by sending too many requests (RPM limit) or by sending bigger requests (TPM limit).

Both offer different rate limits but of the same order of magnitude. In [Azure](https://learn.microsoft.com/en-ca/azure/cognitive-services/openai/quotas-limits), the limits are [scoped to a region](https://learn.microsoft.com/en-us/azure/cognitive-services/openai/how-to/quota), so you can spin up new regions to get around this issue. [OpenAI](https://platform.openai.com/docs/guides/rate-limits) doesn't have multiple region support yet.

| Service      | Model         | Context | TPM (In thousands) | RPM  |
| ------------ | ------------- | ------- | ------------------ | ---- |
| OpenAI       | GPT-4         | 8K      | 40                 | 200  |
| Azure OpenAI | GPT-4         | 8K      | 20                 | 120  |
| OpenAI       | GPT-4         | 32K     | 150                | 20   |
| Azure OpenAI | GPT-4         | 32K     | 60                 | 360  |
| OpenAI       | GPT-3.5 Turbo | 4K      | 90                 | 3500 |
| Azure OpenAI | GPT-3.5 Turbo | 4K      | 240                | 1440 |
| OpenAI       | GPT-3.5 Turbo | 16K     | 180                | 3500 |
| Azure OpenAI | GPT-3.5 Turbo | 16K     | N/A                | N/A  |

## Conclusion

In summary, choose Azure OpenAI if you have compliance and security requirements, and if your organization is already using Microsoft's products or Azure infrastructure.
