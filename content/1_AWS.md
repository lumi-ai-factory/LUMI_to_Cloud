---
title: "Amazon Web Services (AWS)"
nav_order: 2
---

# Amazon Web Services (AWS)

This section covers hosting your fine-tuned model on AWS. There are a few different ways to do it, and this page covers the setup that is the **same regardless of which you choose**: permissions, moving your weights, GPU quotas, choosing a machine size, and cost. Read this page once, then follow the page for your chosen hosting method.

## Prerequisites

Before starting the migration, ensure you have the following ready:

- **An active AWS account**: If you don't have one, you can create it at the [AWS Signup page](https://signin.aws.amazon.com/signup?request_type=register). You must be on a paid tier since the free tier doesn't allow creating instances with GPUs. 
- **A LUMI account**: You should have your AI models ready on LUMI.

## 1. Ways to host on AWS

- **[EC2](1.2_EC2.md)**: you rent a virtual machine with GPUs and run the model server yourself. Most control, most manual work. This is most similar to doing things on LUMI.
- **[SageMaker AI](1.3_SageMaker-AI.md)**: AWS runs the machine and the server for you; you just describe what you want. A good middle ground, and the recommended starting point.
- **Bedrock**: a fully managed service where you don't manage any machine at all.

## 2. Permissions (IAM)

This is the part that most often trips people up, so it is worth understanding clearly. **IAM** (Identity and Access Management) is how AWS decides who is allowed to do what. Two ideas matter here:

- A **user** is you, a human logging into the Console. Your admin user (or the person who set up the account) can change permissions.
- A **role** is an identity that a *machine* or *service* takes on. Your EC2 instance runs as a role; your SageMaker endpoint runs as a role. The role needs permission to read your weights from S3.

The single most important rule: **a machine's role cannot grant itself permissions.** You always edit permissions as a human administrator in the IAM Console, never from inside the running machine or notebook. If a notebook or instance tries to change its own role, AWS refuses with an "Access Denied / not authorized" error. That is a safety feature, not a mistake. Trying to work around it from the machine is the wrong path.

### Giving your role access to your S3 bucket

An **S3 bucket** is simply a top-level folder in AWS's cloud storage service (Simple Storage Service) where you can store files, such as your model weights.

AWS's built-in policies (like `AmazonSageMakerFullAccess`) grant a lot, but they only allow reading S3 buckets whose **name contains the word "sagemaker"**. If your bucket is named anything else, you must explicitly grant access. Here is how, as an administrator:

1. In the Console, open **IAM → Roles** and click the role your compute uses. 
   *(Note: You chose or created this role when you launched the EC2 instance or set up your SageMaker environment. For SageMaker, it is called the execution role; for EC2, it is the instance profile role).*
2. Click **Add permissions → Create inline policy**.
3. Switch to the **JSON** tab and paste the policy below, replacing `YOUR-BUCKET` with your bucket name (in both places):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadModelWeights",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::YOUR-BUCKET",
        "arn:aws:s3:::YOUR-BUCKET/*"
      ]
    }
  ]
}
```

4. Click **Next**, give the policy a name (for example `read-model-weights`), and click **Create policy**.

The two lines matter for different reasons: `s3:ListBucket` on `arn:aws:s3:::YOUR-BUCKET` lets the role *see* what is in the bucket, and `s3:GetObject` on `arn:aws:s3:::YOUR-BUCKET/*` lets it *download* the files.

> [!TIP] Checking a role's permissions
> Open **IAM → Roles → (your role) → Permissions** tab to see everything attached. If a deployment fails with a 403 on S3, this is the first place to look.

## 3. Choosing a machine size

AWS offers dozens of different machine types (instances), but for hosting LLMs, you need **GPU instances**. It is important to know which instance you need *before* requesting a quota increase.

To see what instances are available and their specifications:
- Go to the **Amazon EC2** console, select your desired region in the top-right corner (e.g., Stockholm `eu-north-1` or another European region), and click **Instance Types** on the left menu.
- Filter the "Instance category" by "Accelerated Computing" to see GPU-backed instances. This view will show you exactly which instances are physically available in your selected region.
- **`g5` (NVIDIA A10G)** or **`g6` (NVIDIA L4)** families: These are the most cost-effective choices for standard inference. They have 24GB of VRAM per GPU. A single `g6.xlarge` (1 GPU) is enough to comfortably host an 8-billion parameter model. Multi-GPU sizes (like `g6.12xlarge` with 4 GPUs) are available for medium-sized models.
- **`g6e` (NVIDIA L40S)** family: This is the mid-range option with 48GB of VRAM per GPU. It's excellent when a model is too large for `g6` but doesn't require massive A100s. For example, a 32-billion parameter model fits comfortably on a `g6e.12xlarge` (4x48GB = 192GB VRAM), whereas it would be a tight squeeze on a `g6.12xlarge` (4x24GB = 96GB VRAM).
- **`p4` (A100)** and **`p5` (H100)** families: These are required for larger models (e.g., 70B+ parameters) that need high VRAM and memory bandwidth. However, they are incredibly expensive (a `p4d.24xlarge` with 8x A100s costs over $32 per hour) and it is notoriously difficult to get AWS to grant quotas for them without a dedicated AWS account manager.

## 4. GPU quotas

A brand-new AWS account usually has a limit of **zero** for GPU machines, so your very first deployment can fail with a "ResourceLimitExceeded" error before anything even starts. To raise it:

1. In the Console, open **Service Quotas → AWS services → Amazon SageMaker** (or **Amazon EC2** if you are using EC2).
2. Search for the exact instance type you plan to use: for SageMaker the quota is named like *"ml.g6e.12xlarge for endpoint usage"*.
3. Click **Request increase at account level** and ask for at least 1.

Small requests (e.g., for 1 instance with a small number of GPUs) are usually automatically accepted within an hour or two. However, large requests can take days and may require opening a support ticket. EC2 and SageMaker have **separate** quotas even for the same GPU type, and each instance size is its own quota.

## 5. Cost

GPU compute is expensive and the rates are quoted **by the hour**.
However, AWS actually bills EC2 and SageMaker instances **per second**. You are only billed for the time the instance is actively running (for SageMaker, this means while the endpoint is in the `InService` status, not while it is creating or deleting). This means if you test a model and turn it off after exactly 15 minutes, you will only be charged for 15 minutes. 

To estimate the cost of running different instance types, use the [AWS Pricing Calculator](https://calculator.aws/). Remember that you are billed continuously while the instance is running/InService, regardless of whether you are actively sending requests to it or if traffic is zero.

> [!NOTE] SageMaker Markup vs. EC2
> When making your hosting choice, note that SageMaker's managed layer adds a markup. A SageMaker `ml.` instance typically costs 20 to 40% more than the exact same raw EC2 instance. You pay this premium because SageMaker handles the heavy lifting: it automatically provisions the machine, installs the necessary GPU drivers and serving software, downloads your weights from S3, and wraps it all into a ready-to-use API. With EC2, you rent a blank machine and have to do all of that setup manually.

## 6. Logs

AWS writes logs to a service called **CloudWatch**. When something fails to start or behaves oddly, CloudWatch is where the detailed error messages are. The specific location of the logs depends on the hosting method and is described on each method's page.
