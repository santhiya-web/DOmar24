# GCP CI/CD Setup Guide

This guide provides the exact steps to generate the three secrets required for the GitHub Actions deployment pipeline (`GCP_PROJECT_ID`, `GCP_SERVICE_ACCOUNT`, and `GCP_WORKLOAD_IDENTITY_PROVIDER`).

---

## 💡 Terminology & Significance

Understanding these three variables is key to mastering secure cloud deployments:

*   **`GCP_PROJECT_ID` (The Destination)**: This is the unique identifier for your Google Cloud Project. It acts as a logical container for all your resources (Cloud Run, Artifact Registry, etc.). Setting this ensures your CI/CD pipeline knows exactly which "environment" it is deploying to.
*   **`GCP_SERVICE_ACCOUNT` (The Worker)**: Think of this as a "Robot Identity." It is a non-human account that is granted specific permissions (Roles) to perform tasks like building images and updating services. Instead of using your personal credentials, the pipeline "acts as" this service account to maintain security.
*   - **`GCP_WORKLOAD_IDENTITY_PROVIDER` (The Secure Bridge)**: This is the most critical piece for modern security. It establishes a trust relationship between GitHub and Google Cloud. Instead of storing a raw "master key" (JSON key) in GitHub, this provider allows GitHub to prove its identity using a temporary token. It ensures that only your specific repository can "borrow" the permissions of the Service Account to run deployments.

---

## Step 1: Initial Variables
Set these variables in your terminal to make the following commands copy-pasteable:
```bash
export GCP_PROJECT_ID="YOUR_PROJECT_ID"
export GITHUB_REPO="YOUR_USERNAME/YOUR_REPO" # e.g., firdousali86/DOmar24
```

---

## Step 2: Create a Service Account
This identity will perform the deployments.
```bash
# Create the service account
gcloud iam service-accounts create github-deployer \
    --project="${GCP_PROJECT_ID}" \
    --display-name="GitHub Actions Deployer"

# Assign necessary roles
for ROLE in "roles/run.admin" "roles/artifactregistry.admin" "roles/iam.serviceAccountUser" "roles/secretmanager.secretAccessor"; do
  gcloud projects add-iam-policy-binding "${GCP_PROJECT_ID}" \
      --member="serviceAccount:github-deployer@${GCP_PROJECT_ID}.iam.gserviceaccount.com" \
      --role="${ROLE}"
done
```
**Value for GitHub Secret (`GCP_SERVICE_ACCOUNT`):**
`github-deployer@${GCP_PROJECT_ID}.iam.gserviceaccount.com`

---

## Step 3: Create Artifact Registry Repository
Docker images must be pushed to a repository in Artifact Registry. You must create this once manually.
```bash
gcloud artifacts repositories create "containers" \
    --project="${GCP_PROJECT_ID}" \
    --repository-format="docker" \
    --location="us-central1" \
    --description="Docker repository for frontend and backend images"
```

---

## Step 4: Setup Workload Identity Federation
This allows GitHub to securely connect to GCP without using sensitive JSON keys.

### 1. Create the Pool
```bash
gcloud iam workload-identity-pools create "github-pool" \
    --project="${GCP_PROJECT_ID}" \
    --location="global" \
    --display-name="GitHub Actions Pool"
```

### 2. Create the Provider (with security condition)
```bash
gcloud iam workload-identity-pools providers create-oidc "github-provider" \
    --project="${GCP_PROJECT_ID}" \
    --location="global" \
    --workload-identity-pool="github-pool" \
    --display-name="GitHub Actions Provider" \
    --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
    --attribute-condition="attribute.repository == '${GITHUB_REPO}'" \
    --issuer-uri="https://token.actions.githubusercontent.com"
```

### 3. Grant the Pool access to the Service Account
You need your **Project Number** for this step.
```bash
# Get your project number
export PROJECT_NUMBER=$(gcloud projects describe "${GCP_PROJECT_ID}" --format="value(projectNumber)")

# Link the provider to the service account
gcloud iam service-accounts add-iam-policy-binding "github-deployer@${GCP_PROJECT_ID}.iam.gserviceaccount.com" \
    --project="${GCP_PROJECT_ID}" \
    --role="roles/iam.workloadIdentityUser" \
    --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/github-pool/attribute.repository/${GITHUB_REPO}"
```

---

## Step 4: Collect your GitHub Secrets

Run these commands to get the final values to paste into **GitHub > Settings > Secrets and variables > Actions**:

1.  **`GCP_PROJECT_ID`**:
    ```bash
    echo ${GCP_PROJECT_ID}
    ```
2.  **`GCP_SERVICE_ACCOUNT`**:
    ```bash
    echo "github-deployer@${GCP_PROJECT_ID}.iam.gserviceaccount.com"
    ```
3.  **`GCP_WORKLOAD_IDENTITY_PROVIDER`**:
    ```bash
    gcloud iam workload-identity-pools providers describe "github-provider" \
        --project="${GCP_PROJECT_ID}" \
        --location="global" \
        --workload-identity-pool="github-pool" \
        --format="value(name)"
    ```
