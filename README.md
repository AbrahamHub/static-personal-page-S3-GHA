# Personal Website - Abraham.com.mx

This repository contains the source code for my static personal website and instructions on how to automate its deployment to an Amazon S3 bucket using GitHub Actions.

## Steps

### 1. Set up Amazon S3 to host the static website

1. Log in to the AWS console.
2. Navigate to **Amazon S3** and click **Create bucket**.
   - **Bucket name**: Choose a unique name that resembles a website (e.g., `abraham.com.mx`).
   - **Region**: Select a region near you (e.g., Europe - eu-west-2).
   - **Public access**: Uncheck **Block all public access** and confirm.
3. Once created, click on your bucket and select the **Properties** tab.
   - Scroll down to **Static website hosting**, click **Edit**.
   - **Enable website hosting**.
   - In the **Index document**, enter `index.html`.
   - Save changes.
4. Configure a bucket policy:
   - Go to the **Permissions** tab of your bucket.
   - Under **Bucket Policy**, click **Edit** and enter the following policy:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [
         {
           "Effect": "Allow",
           "Principal": "*",
           "Action": "s3:GetObject",
           "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
         }
       ]
     }
     ```
   - Replace `YOUR_BUCKET_NAME` with the name of your bucket and save.

### 2. How to Get Secrets for GitHub Actions

To allow GitHub Actions to interact with AWS S3, you need to provide three secrets: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_S3_BUCKET`. Follow these steps to get them:

#### 2.1 Get `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

1. Log in to the [AWS Console](https://aws.amazon.com/).
2. Navigate to **IAM** (Identity and Access Management).
3. Click on **Users** and then **Add User**.
4. Choose a name (e.g., `github-actions-deploy`), and select **Programmatic access**.
5. Assign the **AmazonS3FullAccess** policy. Alternatively, create a custom policy to restrict access to only your bucket.
6. Save the keys:
   - **Access Key ID** will be used as `AWS_ACCESS_KEY_ID`.
   - **Secret Access Key** will be used as `AWS_SECRET_ACCESS_KEY`.

#### 2.2 Get `AWS_S3_BUCKET`

The value for `AWS_S3_BUCKET` is simply the name of the S3 bucket you created. For example, if your bucket is named `abraham.com.mx`, then the value for `AWS_S3_BUCKET` is `abraham.com.mx`.

#### 2.3 Add Secrets to GitHub

1. Go to your GitHub repository.
2. Navigate to **Settings** > **Secrets and variables** > **Actions**.
3. Click **New repository secret** and add the following:
   - **Name**: `AWS_ACCESS_KEY_ID`, **Value**: The access key from AWS.
   - **Name**: `AWS_SECRET_ACCESS_KEY`, **Value**: The secret access key from AWS.
   - **Name**: `AWS_S3_BUCKET`, **Value**: Your bucket name (e.g., `abraham.com.mx`).

## 3. Automated Deployment with GitHub Actions

1. Create a file in your repository: `.github/workflows/deploy.yml`.
2. Add the following content to automate the deployment to S3:

```yaml
name: Deploy to S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Sync to S3
        uses: jakejarvis/s3-sync-action@v0.5.1
        with:
          args: --delete
        env:
          AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SOURCE_DIR: './'
