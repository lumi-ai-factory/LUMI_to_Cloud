---
title: "Moving data to AWS"
nav_order: 2
---

# Moving data to AWS

> [!note]
> **No LUMI-O transfer required**
> You can upload data directly from the fast Lustre filesystem on LUMI to AWS. There is no need to move your data to the LUMI-O object storage first.

To move your datasets and model weights from LUMI to an AWS S3 bucket, you first need to configure IAM credentials and then use a data transfer tool like `s3cmd`.

## 1. AWS IAM Setup

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

## 2. Transferring data with `s3cmd`

S3 (Simple Storage Service) is the standard object storage solution on AWS. We recommend using `s3cmd`, a reliable command-line tool available directly on LUMI, to perform this transfer. It is pre-installed on LUMI as part of the `lumio` module.

### 1. Load the required module

First, load the module that contains `s3cmd` on LUMI:

```bash
module load lumio
```

### 2. Configure `s3cmd`

Run the configuration wizard to link the tool with your AWS account:

```bash
s3cmd --configure
```

You will be presented with a series of prompts. Answer them as follows:

- **Access Key**: Paste the Access Key you generated during the IAM setup.
- **Secret Key**: Paste the Secret Key.
- **Default Region**: Enter the region code where your bucket is located (e.g., `eu-north-1` for EU North).
- **S3 Endpoint**: Press **Enter** to accept the default `[s3.amazonaws.com]`.
- **DNS-style bucket+hostname:port template**: Paste the name of your target bucket followed immediately by `.s3.amazonaws.com`. For example: `my-lumi-models.s3.amazonaws.com`.
- **Encryption password**: Press **Enter** to skip (no password needed).
- **Path to GPG program**: Press **Enter** to skip.
- **Use HTTPS protocol [Yes]**: Type `Yes` and press **Enter**.
- **HTTP Proxy server name**: Press **Enter** to skip.
- **Test access with supplied credentials?**: Type `Y`. 
  *(You should see a success message indicating the connection worked fine.)*
- **Save settings?**: Type `y` to save the configuration. *(Note: This might take a minute or two.)*

### 3. Upload your data

You can now synchronise a directory from LUMI's Lustre filesystem to your newly created S3 bucket. 

```bash
s3cmd sync LOCAL_DIR s3://BUCKET_NAME[/PREFIX] 
```

- `LOCAL_DIR`: The path to the folder on LUMI containing your models or data.
- `BUCKET_NAME`: The exact name of your AWS S3 bucket.
- `PREFIX` A specific folder within the bucket where you want the files to land. It is optional, but your AI models need to be in a directory as if it's in the root, it will give a "wrong URL" error. 

> [!note]
> The command will appear to hang for a few minutes while it indexes the files, after which the upload progress will begin.

> [!tip]
> To view all available `s3cmd` commands, simply run `s3cmd` without any arguments. Note that not all commands may work on the LUMI installation.

---

## Using `rclone` (Alternative)

*To be done.*