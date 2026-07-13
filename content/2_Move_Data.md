---
title: "Moving data to AWS"
nav_order: 2
---

# Moving data to AWS

Once your AWS IAM credentials are set up, you are ready to move your datasets and model weights from LUMI to an AWS S3 bucket. S3 (Simple Storage Service) is the standard object storage solution on AWS.

We recommend using `s3cmd`, a command-line tool available directly on LUMI, to perform this transfer.

## Using `s3cmd`

`s3cmd` is a reliable tool for interacting with S3-compatible storage. It is pre-installed on LUMI as part of the `lumio` module.

### 1. Load the required module

First, load the module that contains `s3cmd` on LUMI:

```sh
module load lumio
```

### 2. Configure `s3cmd`

Run the configuration wizard to link the tool with your AWS account:

```sh
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

```sh
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