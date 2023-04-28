---
title: "How to Rotate GCP Service Account Keys with GitHub Actions"
description: "How to Rotate GCP Service Account Keys with GitHub Actions"
featuredpath: "date"
slug: "How to Rotate GCP Service Account Keys with GitHub Actions"
date: 2023-04-28
draft: false
comments: true
categories: ["github"]
tags:
- github
---

In this blog post, we'll cover how to automate the rotation of Google Cloud Platform (GCP) service account keys using 
GitHub Actions. Regularly rotating service account keys is a security best practice to help mitigate the risk of 
unauthorized access.

## Prerequisites

Before we start, ensure that you have the following set up:

1. A GCP project with a service account and key.
2. A GitHub account and a GitHub repository.
3. The [Google Cloud SDK (gcloud)](https://cloud.google.com/sdk) installed and configured.
4. The [GitHub CLI (gh)](https://cli.github.com/) installed and authenticated.

## Rotating GCP Service Account Keys using GitHub Actions

Here are the steps to set up a GitHub Action for rotating GCP service account keys:

### 1. Create GitHub Secrets

Create the following secrets in your GitHub repository (Settings > Secrets > New repository secret):

- `GCP_PROJECT_ID`: Your GCP project ID
- `GCP_SA_KEY`: Your GCP service account JSON key file, encoded in base64 (use `base64 <key-file>.json` to get the encoded content)

### 2. Create the GitHub Workflow

Create a new file in your repository: `.github/workflows/rotate-gcp-service-account-key.yml` and add the following content:

```yaml
name: Rotate GCP Service Account Key

on:
  workflow_dispatch:

jobs:
  rotate-key:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Google Cloud SDK
      uses: google-github-actions/setup-gcloud@v0.2.1
      with:
        project_id: ${{ secrets.GCP_PROJECT_ID }}
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        export_default_credentials: true

    - name: Rotate Service Account Key
      run: |
        SERVICE_ACCOUNT_NAME="your-service-account-name"
        KEY_FILE="new-service-account-key.json"
        PROJECT_ID="${{ secrets.GCP_PROJECT_ID }}"
        SERVICE_ACCOUNT_EMAIL="$SERVICE_ACCOUNT_NAME@$PROJECT_ID.iam.gserviceaccount.com"

        # Create a new service account key
        gcloud iam service-accounts keys create $KEY_FILE --iam-account $SERVICE_ACCOUNT_EMAIL

        # Encode the new key file in base64
        ENCODED_KEY=$(base64 $KEY_FILE)

        # Update the GitHub secret with the new key
        echo $ENCODED_KEY | gh secret set GCP_SA_KEY --repo $GITHUB_REPOSITORY --body -

        # Get the existing keys and remove the oldest key
        OLD_KEY_ID=$(gcloud iam service-accounts keys list --iam-account $SERVICE_ACCOUNT_EMAIL --format="get(name.basename())" --sort-by="~validAfterTime" --limit=1)
        if [ ! -z "$OLD_KEY_ID" ]; then
          gcloud iam service-accounts keys delete $OLD_KEY_ID --iam-account $SERVICE_ACCOUNT_EMAIL --quiet
        fi

        # Clean up the generated key file
        rm $KEY_FILE
```

Replace `your-service-account-name` with the name of the service account you want to rotate the key for.

### 3. Run the Workflow
You can now manually trigger the rotation of the GCP service account key by navigating to the "Actions" tab in your 
repository, selecting the "Rotate GCP Service Account Key" workflow, and clicking on "Run workflow."

### Conclusion
In this blog post, we've shown you how to set up a GitHub Actions workflow to rotate GCP service account keys
automatically. By following these steps, you can improve the security of your GCP projects and ensure that your 
service account keys are regularly rotated according to best practices.

GitHub Actions provides a powerful and flexible platform for automating various tasks, including managing cloud
infrastructure, building and deploying applications, and automating security practices. With the integration of GCP
and GitHub Actions, you can create seamless workflows to manage and maintain your cloud resources effectively.

Happy automating!
