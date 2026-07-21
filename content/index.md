---
title: "Moving from LUMI to AWS"
nav_order: 1
---

# Migrating your AI workloads from LUMI to AWS

If you are an industry user looking to move your AI workloads (such as fine-tuned Large Language Models) from LUMI to a production-ready environment, this guide provides the necessary steps.

LUMI is designed for research and development, but it is not intended for hosting production services. Amazon Web Services (AWS) is a scalable cloud infrastructure tailored for deploying models in production. 

This guide details how to migrate your data, models, and workloads to AWS.

## Prerequisites

Before starting the migration, ensure you have the following ready:

- **An active AWS account**: If you don't have one, you can create it at the [AWS Signup page](https://signin.aws.amazon.com/signup?request_type=register). You must be on a paid tier since free tier doesn't allow creating instances with GPUs. 
- **A LUMI account**: You should have your AI models ready on LUMI. 

