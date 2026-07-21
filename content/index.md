---
title: "Migrating from LUMI to the Cloud"
nav_order: 1
---

# Migrating your AI workloads from LUMI to the Cloud

If you are an industry user looking to move your AI workloads (such as fine-tuned Large Language Models) from LUMI to a production-ready cloud environment, this guide provides the necessary steps.

LUMI is designed for research and development, but it is not intended for hosting production services. Cloud providers offer scalable infrastructure tailored for deploying models in production.

This guide details how to migrate your data, models, and workloads to various cloud providers.

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
