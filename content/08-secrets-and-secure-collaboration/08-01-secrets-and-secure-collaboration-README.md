# Module 8: Secrets and Secure Collaboration

Managing secrets securely is a vital part of working with APIs, especially in teams and across environments. In this module, we'll explore how Insomnia supports secret management and how you can integrate with enterprise-grade external vaults to securely store and share sensitive data.

We will:

- Explore how secret management works in Insomnia
- Compare local secrets with enterprise external vaults
- Focus on using **HashiCorp Vault (on-premises)** with **`AppRole`** authentication
- Walk through a full example using the `transactions-service` and a locally running HCV instance

Official docs:

- [Insomnia Secret Management](https://docs.insomnia.rest/insomnia/environment-variables#secret-environment-variables)
- [Enterprise Vault Integration](https://docs.insomnia.rest/insomnia/external-vault-integration)

## Prerequisites

In order to complete this module you will need to work on a copy of the `transactions-service` template GitHub repository. This was set up in the last module. If you are jumping directly into this module without having completed the previous one please follow these steps to set up your copy of the `transactions-service`.

### Create Your Repository from the Template

Begin by locating the template repository at:

`https://your-git-enterprise-repo/konghq-cx/transactions-service.git`

(or for Kong Internal teams its)
[https://github.com/kongHQ-CX/transactions-service.git]

#### Generate from Template

1. Open the template repository in your browser.
2. Click **"Use this template"** or **"Generate from template"** (the wording may vary depending on your GitHub Enterprise setup).
3. Name your new repository using the convention of appending your initials:

   ```text
   transactions-service-<your-initials>
   ```

   For example: `transactions-service-sb` if your initials are SB.
4. Complete the repository creation process.

#### Clone the Repository Locally

Once your repository is created, clone it to your local machine:

```text
git clone https://<customer-git-enterprise-repo>/konghq-cx/transactions-service-<your-initials>.git
cd transactions-service-<your-initials>
```

If you're running this internally at Kong, use:

```text
git clone https://github.com/konghq-cx/transactions-service-<your-initials>.git
cd transactions-service-<your-initials>
```

### Open Insomnia and add the `Transactions-Service` as a project

1. Open Insomnia v11, and make sure you are signed in with your company email.
2. Click the + (plus) button in the Projects panel.
3. Choose "Clone from Git Repository".
4. Set name for the project e.g. `transactions-service-<your-initials>`
5. Enter the full repository path:

    ```text
    konghq-cx/transactions-service-<your-initials>
    ```

6. Click next

    ![`transactions-service-import`](../07-mocking/images/transactions-service-import-git-sync.png)

7. Select the repository from the dropdown when it appears.
    - If the repository does not appear you may need to authorise Insomnia to you GitHub organisation
    - If this happens you can click on the `Can't find a repository? Configure the App` link, which will take you off to GitHub to authorise Insomnia
8. You will see a notification that Insomnia has found an existing Insomnia specification file and will import it.

![`transactions-service-import-files`](../07-mocking/images/transactions-service-import-files.png)

## Step 1: Understanding Why Secrets Management Matters

Secrets like API keys, tokens, and credentials should **never be stored in plaintext** or committed to source control. Insomnia provides secure and flexible options to keep sensitive data safe while still enabling teams to collaborate effectively.

## Step 2: Creating a Feature Branch

Before we start configuring secrets management, let's create a dedicated feature branch:

1. In Insomnia, select the `transactions-service` project and click on the branch name in the Git panel (lower left)
2. Create a new branch named `feature/vault-integration`
3. Click "Create" to switch to the new branch

## Step 3: Local Secret Environment Variables (Developer-only)

Insomnia supports **local secret environments**, which allow individuals to store secrets securely and privately.

### Features

- Secrets are stored encrypted on disk
- Not synced with Git or shared with others
- Values are masked in the UI
- Accessed using `vault.<variableName>`

### How to Set One Up

Note: We want to focus on enabling secure secret collaboration, which the private vault does not
provide therefore the following steps are just for reference.

To set up a Private Vault:

1. Navigate to `Insomnia` → `Settings` → `Generate Vault Key` and generate your Vault Key.
    ![`transactions-service-settings-gen-key-vault`](./images/transactions-service-setting-vault-gen.png)
2. Copy the generated key somewhere safe
3. In your environment settings, create a **Private Sub-Environment**.
    ![`transactions-service-settings-edit-vault-env`](./images/transactions-service-edit-env-vault.png)
    ![`transactions-service-create-private-env`](./images/transactions-service-private-env-create.png)
4. Add secret variables, setting the type to `Secret`.

**Note:** Local secrets cannot be shared — ideal for individual developer use only.

## Step 4: Understanding Enterprise Vault Integration

Insomnia Enterprise allows integration with **external secret managers**, enabling **secure collaboration** across your organisation.

### Supported Providers

- **HashiCorp Vault** (Cloud or `On-Prem`)
- AWS Secrets Manager
- GCP Secret Manager
- Azure Key Vault

This module focuses on **HashiCorp Vault (`On-Prem`)** with **`AppRole`** authentication. We want to be able to add a secret value to our `transactions-service` environment, so we can use it as the value for an API key in a request.

## Step 5: Setting Up HashiCorp Vault with `AppRole`

We'll now walk through how to configure Insomnia to retrieve secrets from an HCV server and use them in requests from the `transactions-service`.

### Start a Local HCV and `transactions-service` implementation

From your local cloned copy of the `transactions-service` repository, run the following

```bash
docker-compose up -d
```

The docker compose file sets up HCV with an API key secret at the following path:
`secret/transactions-api/api-key`

Then we need to retrieve the role ID and secret ID, so we can use them in Insomnia,
there is a helper script to get these credentials in the `transactions-service` repository:

```bash
./get-vault-creds.sh
role_id: 1e90574e-2d87-0bb2-5efb-46692083d872
secret_id: 3256202a-d9d3-b284-e2c8-37dd91911928
```

Save both values — we'll need them in Insomnia.

## Step 6: Configure Insomnia to Use HashiCorp Vault

1. Navigate to `Insomnia` → `Settings` → `Cloud Credentials`
2. Click Add Credentials
3. Select HashiCorp Vault
   ![`transactions-service-setup-hcv`](./images/transactions-service-setup-hcv.png)
4. Input the following:
   - Credential Name: `local-hcv`
   - Vault Server: [http://localhost:8200] (or your internal Vault URL)
   - Environment: On-Premises
   - Authentication Method: `AppRole`
   - Role ID: (your retrieved role-id)
   - Secret ID: (your retrieved secret-id)
   ![`transactions-service-hcv-details`](./images/transactions-service-setup-hcv-details.png)
5. Save the credential

## Step 7: Reference the Secret in the Environment

In your transactions-service project:

1. Open the Environment Editor
2. Switch to table view
3. Add a new variable called `api_key`
   ![`transactions-service-env-var-select`](./images/transactions-service-vault-env-var-select.png)
4. Right click in the value field and select external vault then HashiCorp Vault
5. Fill in the details as follows:

   |Field Name| Value|
   | -------- | ---- |
   |Credential For Vault Service Provider|`local-hcv`|
   |Secret Name|`transactions-api/api-key`|
   |KV Secret Engine Version|`v2`|
   |Secret Engine Path|`secret`|
   |Secret Key|`value`|

   The live preview should automatically update with the secret value
   ![`transactions-service-create-env-var`](./images/transactions-service-vault-env-var.png)

   ![`transactions-service-env-var-done`](./images/transactions-service-vault-done.png)

### Commit Your Environment Configuration

After adding the vault reference:

1. Open the Git panel in Insomnia
2. Commit with a message:

   ```text
   feat: add HashiCorp Vault integration for API key management
   ```

3. Push your changes

## Step 8: Use the Secret in a Request

Update a request in the transaction's collection to use the secret:

1. Select the GET health request
2. Select the `Auth` tab
3. Choose `API Key` from the drop-down list
4. Set the key name to `X-BanKong-API-Key`
5. Set the key value to `{{ api_key }}`
   ![`transactions-service-vault-api-key`](./images/transactions-service-request-vault-api-key.png)
6. Make a request from the GET health request, and view that the console displays the decoded API key secret
7. You'll see the value remains masked in the editor but will be correctly injected during request execution.
   ![`transactions-service-request-vault-select-api-key`](./images/transactions-service-vault-env-var-select.png)

### Commit Your Request Updates

After updating requests to use vault variables:

1. Open the Git panel in Insomnia
2. Commit with a message:

   ```text
   refactor: use vault-managed API key in request headers
   ```

3. Inspect your commit changes to make sure no sensitive data is shared in source control
   ![`transactions-service-commit-log`](./images/transactions-service-commit-log.png)
4. Push your changes

## Step 11: Final Merge

Once everything is working correctly:

1. Open the Git panel
2. Make sure all changes are committed and pushed
3. Create a pull request in your Git provider's interface
4. After review, merge the `feature/vault-integration` branch into your main branch

## Summary

In this module, you've:

- Explored Insomnia's built-in secret management features
- Understood the difference between local secrets and external vaults
- Configured HashiCorp Vault with `AppRole` authentication
- Retrieved and injected secrets securely into requests
- Used secure secrets in a collaborative environment
- Committed all changes using Git integration in Insomnia, ensuring no sensitive data is stored in version control

This approach ensures sensitive values never appear in plaintext, while still allowing your team to collaborate using shared specs and environments.
