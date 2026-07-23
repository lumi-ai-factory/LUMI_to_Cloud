---
title: "Migrating from LUMI to the Cloud"
nav_order: 1
---

# Migrating your AI workloads from LUMI to the Cloud

> [!WARNING]
> **Work in Progress**
> This guide is currently under active development. Some sections are incomplete.

If you are an industry user looking to move your AI workloads (such as fine-tuned Large Language Models) from LUMI to a production-ready cloud environment, this guide provides the necessary steps.

Once you have trained or fine-tuned a model on LUMI, you will often want to **host** it somewhere it can answer requests, as an API that your application, colleagues, or users can send questions to. LUMI is designed for research and development; for hosting a running service, you move to a cloud provider.

These materials walk you through doing that on several platforms. The provider-specific pages differ in the details, but the overall journey is the same everywhere: get your model weights into the provider's storage, rent a GPU-backed machine or managed service, load the model, and call it as an API. 

This page collects the things that are **true on every platform**, so the provider pages don't have to repeat them. It is written to be followed even if you have never used a cloud provider before. Where a step involves a cloud-specific concept, there is a short plain-language explanation of what it is and why it matters. Technical readers can skim the explanations and go straight to the commands.

## The shape of the task

Regardless of provider, hosting a fine-tuned LLM comes down to four moves:

1. **Move your weights** off LUMI and into the provider's storage.
2. **Get a GPU machine or managed hosting service** big enough to hold the model.
3. **Load the model** into a serving program that turns it into an API.
4. **Send requests** to it, and **switch it off** when you are done.

## Cost: the one thing to internalise first

GPU compute is expensive and is billed **by the hour, for every hour it is switched on**, not by how much you actually use it, just like LUMI. This is true on every cloud. A machine large enough to host a big model can cost several to well over ten euros per hour, so a machine left running over a weekend by accident is the most common way people get an unpleasant bill. Build the habit of **shutting a model down as soon as you finish using it**, and only turning it back on when you need it. Each provider page shows exactly how to switch things off for that platform.

## Preparing your weights (any platform)

Serving programs expect your model in the standard **Hugging Face format**: a single folder containing the configuration files, the weight files (usually several `*.safetensors` files), and the tokenizer files. Two things help on every platform:

- Keep the model in **one clean folder** with nothing else in it. If your training run left behind optimizer states, checkpoint sub-folders, or other files, copy just the model files into a fresh folder. Storage services upload everything you give them, so extra files just cost time and money.
- Leave the files **uncompressed** (don't zip or tar them). Serving stacks load an uncompressed folder directly and faster.

## How much GPU memory does my model need?

The model's weights must fit in the machine's total GPU memory, with headroom left over for the conversation itself (longer prompts and answers need more). A useful rule of thumb for a standard 16-bit (bf16/fp16) model is roughly **2 GB of GPU memory per billion parameters**, plus overhead for KV cache and more.

## Choosing a Cloud Provider

When deciding which cloud provider to use for your AI workloads, a great rule of thumb is to **stick to the cloud provider that your team is already working with or has experience with**. If you are starting fresh, consider the following nuances:

- **Amazon Web Services (AWS)**: 
  - **Pros & Cons**: AWS is the most mature and flexible cloud ecosystem, but it can also be the most complicated to set up. It is great for teams looking for extensive managed services and enterprise support.
  - **Capacity in EU/Finland**: AWS does not have a region inside Finland. The closest region is in Stockholm, Sweden (`eu-north-1`), which provides excellent latency to Finland. [View global infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/).
  - **Pricing**: [AWS Pricing Calculator](https://calculator.aws/)

- **Google Cloud Platform (GCP)**: 
  - **Pros & Cons**: Known to be the most beginner-friendly with excellent out-of-the-box ML tooling. However, you might find it restrictive if you need to do highly advanced, custom networking or specialized infrastructure setups.
  - **Capacity in EU/Finland**: GCP has a dedicated region located in Hamina, Finland (`europe-north1`), powered by sustainable energy. [View global infrastructure](https://cloud.google.com/about/locations).
  - **Pricing**: [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator)

- **Microsoft Azure**: 
  - **Pros & Cons**: Ideal for teams deeply integrated into the Microsoft ecosystem or those requiring exclusive integrations, such as specific OpenAI models hosted directly on Azure infrastructure.
  - **Capacity in EU/Finland**: Azure does not have a region inside Finland yet. The closest region is Sweden Central. [View global infrastructure](https://azure.microsoft.com/en-us/explore/global-infrastructure/geographies/).
  - **Pricing**: [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/)

- **Finnish / European Providers (e.g., UpCloud)**: 
  - **Pros & Cons**: A great option for projects with strict data residency, privacy requirements, or teams that want to support the local Finnish/European economy. They often offer straightforward pricing and excellent local support.
  - **Capacity in EU/Finland**: Dedicated data centers located directly in Helsinki and other European hubs.
  - **Pricing**: [UpCloud Pricing](https://upcloud.com/pricing)
