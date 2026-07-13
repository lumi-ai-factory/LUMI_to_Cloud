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

- **An active AWS account**: If you don't have one, you can create it at the [AWS Signup page](https://signin.aws.amazon.com/signup?request_type=register).
- **A LUMI account**: You should have your AI models ready on LUMI. 

> [!note]
> **No LUMI-O transfer required**
> You can upload data directly from the fast Lustre filesystem on LUMI to AWS. There is no need to move your data to the LUMI-O object storage first.

## Step 1: AWS IAM Setup

To securely transfer your data from LUMI to AWS, you will need to create an Identity and Access Management (IAM) user with the appropriate permissions and generate access keys.

1. **Log in to the AWS Console**
   Navigate to [aws.amazon.com](https://aws.amazon.com/) and sign in. Use the search bar at the top to search for and open **IAM**.
   
2. **Create a new IAM User**
   In the left-hand menu, click on **IAM users**, and then click the **Create user** button. Enter a descriptive name for the user (e.g., `lumi-migration-user`) and click **Next**.
   
3. **Assign Permissions**
   - Choose **Add user to group**.
   - If you don't have a group yet, click **Create group**. 
   - Filter the policies by type: select **AWS managed - job function** and choose the appropriate policy. *(Note: `AdministratorAccess` gives full control, which is the easiest but least restrictive option. For stricter security, use `AmazonS3FullAccess` if you only need to transfer data.)*
   - Click **Create user group**, tick the newly created group, and click **Next**.
   - Finally, click **Create user**.

4. **Generate Access Keys**
   - On the success screen (green pop-up), click **View user**.
   - Navigate to the **Security credentials** tab.
   - Scroll down to the **Access keys** section and click **Create access key**.
   - Select **Third-party service**, tick the confirmation box, and click **Next**.
   - Give a name to the key and click **Create access key**.

> [!warning]
> **Safely store your credentials!**
> Be sure to save the **Access key** and **Secret access key** in a secure place. You will be prompted to enter these credentials when configuring `s3cmd` on LUMI in the next step.
