---
title: "Amazon Web Services (AWS)"
nav_order: 2
---

# Amazon Web Services (AWS)

This section covers hosting your fine-tuned model on AWS. We outline the different hosting services available and how they differ, along with the **core concepts and requirements that are the same regardless of which you choose**: permissions, GPU quotas, choosing a machine size, and cost. Read this page once to understand the landscape, [upload your files to S3](1.1_Move_data.md), and then follow the page for your chosen hosting method.

## Prerequisites

Before starting the migration, ensure you have the following ready:

- **An active AWS account**: If you don't have one, you can create it at the [AWS Signup page](https://signin.aws.amazon.com/signup?request_type=register). You must be on a paid tier since the free tier doesn't allow creating instances with GPUs. 
- **A LUMI account**: You should have your AI models ready on LUMI.

## 1. Ways to host on AWS

- **[EC2](1.2_EC2.md)**: you rent a virtual machine with GPUs and run the model server yourself. Most control, most manual work. This is most similar to doing things on LUMI.
- **[SageMaker AI](1.3_SageMaker-AI.md)**: AWS provisions the machine and runs the server for you; you just select the machine type and container, and it provides an API endpoint. More expensive than EC2 on the same hardware.
- **Bedrock**: a fully managed service where you don't manage any machine at all. It charges per-token and is mostly used for AWS's pre-trained foundation models. You can host your own fine-tuned weights through Custom Model Import, but you are then billed for each active model copy at rates well above renting the equivalent GPU yourself, so it is rarely worth it for a self-hosted model.

### What it takes to turn either one into an API

**EC2: A traditional web API.** When you deploy on EC2 using vLLM, it listens on a port. Your application calls it with an API key you set, and any OpenAI-compatible client library works. However, because you are running a public server, you are entirely responsible for securing the connection with HTTPS certificates, configuring firewall rules, and restarting the server if it crashes.

**SageMaker AI: A private, secure endpoint.** SageMaker does not provide a public IP address or rely on simple API keys. Instead, every request must be cryptographically signed using your AWS credentials via an AWS SDK. AWS handles encryption automatically and restarts the model if it fails. If you want to expose your model to third parties or use standard OpenAI clients, you must build a lightweight proxy server to sit in front of SageMaker. This proxy holds your AWS credentials, validates incoming API keys, and forwards the prompts to SageMaker.

**Which should you choose?**
- Choose **EC2** if you explicitly need a direct, public-facing API, or if minimising hourly infrastructure costs outweighs the convenience of a managed service.
- Choose **SageMaker** if your own application backend is the only consumer of the model and you prefer a fully managed service over handling server maintenance and security.

## 2. Permissions (IAM)

Before an AWS service or virtual machine can access your model weights, it must be explicitly granted permission to do so, and getting this wrong is the most common reason a first deployment fails. You configure this as an administrator in the **AWS Console** (AWS's website where you sign in and manage things).

![AWS Console](./assets/AWS_Console.png)

**IAM** (Identity and Access Management) is the part of the Console that decides who is allowed to do what. Two kinds of identity matter here:

- A **user** is you, a human logging into the Console. Your admin user (or the person who set up the account) can change permissions.
- A **role** is an identity that a *machine* or *service* takes on. Your EC2 instance runs as a role; your SageMaker endpoint runs as a role. The role needs permission to read your weights from S3.

The single most important rule: **a machine's role cannot grant itself permissions.** You always edit permissions as a human administrator in the IAM Console, never from inside the running machine or notebook. If a notebook or instance tries to change its own role, AWS refuses with an "Access Denied / not authorised" error.

### Your role needs read access to your S3 bucket

Amazon S3 (Simple Storage Service) is AWS's object storage service, which acts like an infinitely scalable hard drive in the cloud. An **S3 bucket** is the top-level folder where you store your files, such as your model weights.

Permissions are granted through a **policy**: a document, in AWS's format, listing what an identity may do. AWS provides ready-made ones (called managed policies), and an administrator can also attach a small custom one, called an **inline policy**, for a single specific need. Your role does not automatically get read access to your bucket: on EC2 it starts with no S3 access at all, and on SageMaker the default `AmazonSageMakerFullAccess` policy only covers buckets whose **name contains the word "sagemaker"**. So an administrator usually has to grant the role read access to your specific bucket with a small inline policy.

The exact steps to configure these roles and the required policy documents are provided on the specific hosting method pages (**[EC2](1.2_EC2.md)** and **[SageMaker AI](1.3_SageMaker-AI.md)**) since they vary slightly depending on the service you choose.

## 3. Choosing a machine size

AWS offers dozens of different machine types (instances), but for hosting LLMs, you need **GPU instances**. It is important to know which instance you need *before* requesting a quota increase.

### How to check available hardware

Because SageMaker is built on top of EC2, both services use the exact same underlying hardware. Therefore, the best way to browse available instances and see details on the hardware is through the EC2 console:

1. Open the **Amazon EC2** console and select your desired region in the top-right corner (e.g., Stockholm `eu-north-1`).
2. Click **Instance Types** on the left menu.
3. To filter for GPU machines, click the search bar ("Find instance types by attributes"), select **GPUs**, choose **>=**, and enter **1**.
4. Click on any instance name to see its hardware details.

![EC2 Instance Types](./assets/EC2_Instance_types_Stockholm.png)

> [!NOTE] Note for SageMaker users
> A SageMaker instance is just an EC2 instance with an `ml.` prefix added to its name. For example, if you find that the `g6e.xlarge` EC2 instance has the hardware you want, the SageMaker equivalent is exactly the same hardware under the name `ml.g6e.xlarge`.

### GPU Families in Stockholm (`eu-north-1`)

Here is a summary of the primary GPU instance families available:

- **`p5en`** (H200 GPUs): In Stockholm, these are only available as massive 8-GPU nodes (`p5en.48xlarge`), making them suited only for the largest enterprise models.
- **`p4d`** (A100 GPUs): Like the `p5en`, these are massive 8-GPU nodes that are very expensive and notoriously difficult to get quotas for.
- **`g7e`** (RTX PRO 6000 Blackwell GPUs): The newest generation, offering 96GB VRAM per card, enough to hold a 32B model on a single GPU. AWS announced Stockholm availability in July 2026, but SageMaker refused the `ml.g7e` types there when we tried, so check whether your service actually offers it before planning around it. A family usually reaches EC2 well before SageMaker.
- **`g6e`** (L40S GPUs): Offers 48GB VRAM per card.
- **`g6`** (L4 GPUs): Offers 24GB VRAM per card.
- **`g5`** (A10G GPUs): Offers 24GB VRAM per card. Older generation than `g6` but still viable if `g6` is unavailable.
- **`g4dn`** (T4 GPUs): An older generation GPU; avoid it for modern LLMs.


## 4. GPU quotas

A brand-new AWS account usually has a limit of **zero** for GPU machines, so if you try to deploy immediately, you may encounter a "ResourceLimitExceeded" error. To prevent this, you should raise your quota first:

1. In the Console, open **Service Quotas > AWS services** and search for the service you will use: open **Amazon Elastic Compute Cloud (Amazon EC2)** or **Amazon SageMaker**.
2. Search for the relevant quota:
   - For **EC2**, search for *"Running On-Demand G and VT instances"* (for the G family: `g5`/`g6`/`g6e`/`g7e`) or *"Running On-Demand P instances"* (for the P family: `p4d`/`p5en`).
   - For **SageMaker**, search for the instance type and ensure you select the one for **endpoint usage** (e.g., *"ml.g6e.12xlarge for endpoint usage"*). The `ml.` prefix indicates it is a managed SageMaker instance. The [SageMaker notebook](1.3_SageMaker-AI.md) uses **`ml.g6e.12xlarge`** (or **`ml.g5.12xlarge`** as a backup), so we recommend requesting quota for both.
3. Click **Request increase at account level**. 
   - **Important for EC2:** EC2 quotas are measured in **vCPUs**, not instances. A `12xlarge` instance has 48 vCPUs, so you must request a quota of at least 48. 
   - **Important for SageMaker:** SageMaker quotas are measured in **instances**, so you only need to request a quota of 1.

Small requests (e.g., for a small number of instances) are usually automatically accepted within an hour or two. However, large requests can take longer. EC2 and SageMaker have **separate** quotas even for the same GPU type.

## 5. Cost

GPU compute is expensive and the rates are quoted **by the hour**.
However, AWS actually bills EC2 and SageMaker instances **per second** (with a 60-second minimum). You are only billed for the time the instance is actively running. This means if you test a model and turn it off after exactly 15 minutes, you will only be charged for 15 minutes. 

To estimate the cost of running different instance types, use the [AWS Pricing Calculator](https://calculator.aws/). Remember that you are billed continuously while the instance is running/InService, regardless of whether you are actively sending requests to it or if traffic is zero.

> [!NOTE] SageMaker Markup vs. EC2
> When making your hosting choice, note that SageMaker's managed layer adds a markup. A SageMaker `ml.` instance (e.g., `ml.g6e.12xlarge`) typically costs 20 to 40% more than the exact same raw EC2 instance (`g6e.12xlarge`). You pay this premium because SageMaker handles the heavy lifting: it automatically provisions the machine, installs the necessary GPU drivers and serving software, downloads your weights from S3, and wraps it all into a ready-to-use API. With EC2, you rent a blank machine and have to do all of that setup manually.

## 6. Logs

AWS writes logs to a service called **CloudWatch**. When something fails to start or behaves oddly, CloudWatch is where the detailed error messages are. The specific location of the logs depends on the hosting method and is described on each method's page.
