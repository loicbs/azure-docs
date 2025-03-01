---
title: Azure OpenAI Service quotas and limits
titleSuffix: Azure Cognitive Services
description: Quick reference, detailed description, and best practices on the quotas and limits for the OpenAI service in Azure Cognitive Services.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: openai
ms.topic: conceptual
ms.date: 06/08/2023
ms.author: mbullwin
---

# Azure OpenAI Service quotas and limits

This article contains a quick reference and a detailed description of the quotas and limits for Azure OpenAI in Azure Cognitive Services.

## Quotas and limits reference

The following sections provide you with a quick guide to the default quotas and limits that apply to Azure OpenAI:

| Limit Name | Limit Value |
|--|--|
| OpenAI resources per region per Azure subscription | 30 |
| Default quota per model and region (in tokens-per-minute)<sup>1</sup> |Text-Davinci-003: 120 K <br> GPT-4: 20 K <br> GPT-4-32K: 60 K  <br> All others: 240 K |
| Maximum prompt tokens per request | Varies per model. For more information, see [Azure OpenAI Service models](./concepts/models.md)|
| Max fine-tuned model deployments | 2 |
| Total number of training jobs per resource | 100 |
| Max simultaneous running training jobs per resource | 1 |
| Max training jobs queued | 20 | 
| Max Files per resource | 30 |
| Total size of all files per resource | 1 GB | 
| Max training job time (job will fail if exceeded) | 720 hours |
| Max training job size (tokens in training file) x (# of epochs) | 2 Billion |

<sup>1</sup> Default quota limits are subject to change.

### General best practices to remain within rate limits

To minimize issues related to rate limits, it's a good idea to use the following techniques:

- Implement retry logic in your application.
- Avoid sharp changes in the workload. Increase the workload gradually.
- Test different load increase patterns.
- Increase the quota assigned to your deployment. Move quota from another deployment, if necessary.

### How to request increases to the default quotas and limits

Quota increase requests can be submitted from the [Quotas](./how-to/quota.md) page of Azure AI Studio. Please note that due to overwhelming demand, we are not currently approving new quota increase requests. Your request will be queued until it can be filled at a later time.

For other rate limits, please [submit a service request](/azure/cognitive-services/cognitive-services-support-options?context=%2Fazure%2Fcognitive-services%2Fopenai%2Fcontext%2Fcontext).

## Next steps

Explore how to [manage quota](./how-to/quota.md) for your Azure OpenAI deployments.
Learn more about the [underlying models that power Azure OpenAI](./concepts/models.md).
