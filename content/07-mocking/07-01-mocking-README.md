# Module 7: Mocking

In this module, we explore the value of API mocking and demonstrate how to use Insomnia's **self-hosted mocking** to support dependent service development — in our case, the `transactions-service` which relies on the `accounts-service`.

Mocking is an important capability in modern API design and development. It allows teams to:

- Work in parallel — consumers can develop against a stable interface before the backend is live.
- Prototype integrations without depending on backend uptime.
- Validate request/response formats early and often.

Insomnia supports two modes of mocking:

1. **Cloud-based mocking** — available to any logged-in user. These are hosted by Kong's infrastructure.
2. **Self-hosted mocking** — available to enterprise users. These mocks are stored in version control and can be deployed via Docker using [`Mockbin`](https://github.com/Kong/insomnia-mockbin/tree/master).

For organisations with compliance, security, or offline development needs, **self-hosted mocking is recommended**.

> [Documentation: Insomnia Mocking](https://docs.insomnia.rest/insomnia/api-mocking)

## Prerequisites

In this module, you'll need both the accounts-service and the new transactions-service projects loaded into Insomnia. You will also need to have both codebases cloned locally.

The accounts-service is required, so we can generate the mock server using live requests in Insomnia. This means the accounts-service must be checked out locally, and its implementation must be running, so it can respond to requests from Insomnia. These live responses will be captured and used to populate the mock.

Once the mock is fully configured and committed, we will shut down the running accounts-service and begin development of the transactions-service against the mock instead.

The steps below will guide you through setting everything up correctly:

### Prerequisite Step 1: Create Your Repository from the Template

Begin by locating the template repository at:

[https://github.com/kongHQ-CX/transactions-service.git]

   > Note: `https://github.com/konghq-cx` is the address for the public version of this repository, if your company uses an enterprise git offering then this address will be updated in the course notes.

#### Create Your Own Repository

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

```bash
git clone https://github.com/konghq-cx/transactions-service-<your-initials>.git
cd transactions-service-<your-initials>
```

   > Note: `https://github.com/konghq-cx` is the address for the public version of this repository, if your company uses an enterprise git offering then this address will be updated in the course notes.

#### Docker Compose Configuration

Inside the repository, you'll find:

- A working `OpenAPI` specification file
- Predefined Insomnia request collections
- A docker-compose.yaml file to launch supporting services

This `docker-compose` file includes not just the transactions-service itself, but also several supporting components used in the final modules of the workshop:

```yaml
services:
  transactions-api:
    image: kongcx/transactions:1.0.0
    container_name: transactions
    ports:
      - "8082:8082"
    environment:
      - LOG_LEVEL=info
      - ACCOUNTS_SERVICE_URL=http://accounts-service-mock:8080/bin/${MOCK_PATH_ID}
    restart: unless-stopped
  accounts-service-mock:
    image: ghcr.io/kong/insomnia-mockbin:v2.0.2
    container_name: account-service-mock
    environment:
      MOCKBIN_REDIS: "redis://redis:6379"
      MOCKBIN_QUIET: "false"
      MOCKBIN_PORT: "8080"
      MOCKBIN_REDIS_EXPIRE_SECONDS: 1000000000
    links:
      - redis
    ports:
      - "8080:8080"
  redis:
    image: redis
  vault:
    image: hashicorp/vault:1.19.2
    container_name: vault
    ports:
      - "8200:8200"
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: root
      VAULT_ADDR: http://0.0.0.0:8200
    volumes:
      - ./vault-init.sh:/vault-init.sh
    command: >
      sh -c "vault server -dev -dev-root-token-id=root -dev-listen-address=0.0.0.0:8200 &
             sleep 5 &&
             sh /vault-init.sh &&
             tail -f /dev/null"
```

#### Understanding the Compose Configuration

- The `transactions-api` container depends on the accounts service to perform credit and debit operations.
- Rather than connecting to a live version of accounts-service, we point it to a mock instance running in the container `account-service-mock`.
- This mock container uses Kong's Insomnia `Mockbin` image, which allows us to simulate API responses during testing.
- The `Mockbin` container requires Redis, which is included to store route and response definitions.
- Lastly, for the Secrets Management module, we include a local instance of HashiCorp Vault to demonstrate external secret integration with Insomnia.

**Warning:** Don't Start the Containers Yet

We'll hold off on starting the Compose stack for now. The `transactions-api` service requires a valid value for the `MOCK_PATH_ID` environment variable, which we haven't created yet.

To generate this ID, we must first create a mock entity in Insomnia. Once that's done, we'll return to this Compose file and launch the services with the correct configuration.

### Prerequisite Step 2: Open Insomnia and add the `Transactions-Service` as a project

1. Open Insomnia v11, and make sure you are signed in with your company email.
2. Click the + (plus) button in the Projects panel.
3. Choose "Git Sync".
4. Set name for the project e.g. `transactions-service-<your-initials>`
5. Enter the full repository path:

    ```text
    konghq-cx/transactions-service-<your-initials>
    ```

6. Click next

    ![`transactions-service-import`](./images/transactions-service-import-git-sync.png)

7. Select the repository from the dropdown when it appears.
    - If the repository does not appear you may need to authorise Insomnia to you GitHub organisation
    - If this happens you can click on the `Can't find a repository? Configure the App` link, which will take you off to GitHub to authorise Insomnia
8. You will see a notification that Insomnia has found an existing Insomnia file and will import it.

![`transactions-service-import-files`](./images/transactions-service-import-files.png)

### Prerequisite Step 3: Make Sure the `accounts-service` is Running

To generate a working mock of the `accounts-service`, we need to send real requests to its implementation. Insomnia will capture the live responses and use them to populate the mock routes.

If you've followed along with the previous modules, your `accounts-service` project should already contain all the required requests. In this case, you simply need to ensure the implementation is running locally.

> **Note:** If the service is not running, start it with:

```bash
docker compose up -d
```

#### Starting from This Module?

If you are jumping in at this point of the workshop (e.g., for a focused demo or standalone practice), follow these steps to get the correct project state:

1. Clone the `insomnia-workshop-exports` repository if you haven't already:

   ```bash
   git clone https://github.com/konghq-cx/insomnia-workshop-exports.git
   ```

2. Open Insomnia and create a new Git Sync project, or open an existing one.

3. From the Insomnia Project Overview, click Import From File, and navigate to:

   ```text
   <path-to-your-clone>/module-exports/module-06/api-spec/module-06-end-state.yaml
   ```

   > Note: `https://github.com/konghq-cx` is the address for the public version of this repository, if your company uses an enterprise git offering then this address will be updated in the course notes.

This file represents the state of the accounts-service project at the end of Module 6, including:

- A complete API spec
- Request collections
- Test suites

This will ensure you have everything needed to proceed with the mocking workflow in this module.

Once imported, make sure to start the real accounts-service implementation locally so that Insomnia can interact with the live API and capture accurate responses.

## Step 1: Creating a Mock Server for the Accounts API

We'll now create a mock version of the `accounts-service`, which will be used by the `transactions-service` for local development and integration.

### Creating a Feature Branch

Before we start working with mock servers, let's create a dedicated feature branch:
Make sure you have the `accounts-service` project selected in Insomnia, then

1. Click on the branch name in the Git panel (lower left)
2. Create a new branch named `accounts-service-add-mock-server`
3. Click "Create" to switch to the new branch

### Create the Mock Server

In Insomnia in the `accounts-service` project:

1. Click + Create → Mock Server.

![`transactions-service-create-mock`](./images/transactions-service-create-mock.png)

Set the following details:

- Name: `accounts-service-mock`
- Filename: `accounts-service-mock`
- Folder where the file will be saved in the repository: `mocks/` folder
- Self-hosted mock server URL: `http://localhost:8080`

![`transactions-service-mock-details`](./images/transactions-service-mock-details.png)

### Commit Your Mock Server Configuration

After creating the mock server:

1. Open the Git panel in Insomnia
2. Commit with a message:

    ```text
    feat: add mock server configuration
    ```

3. Push your changes

## Step 4: Populate the Mock Routes

We'll now use the request collection in the `accounts-service` to seed the mock routes.
For each request:

1. Run the request in the Collection tab.
2. Select the Mock tab in the right panel.
3. Choose `accounts-service-mock`.
4. Click Create New Mock Route.
5. Accept the default route name

   ![`transactions-service-generate-mock`](./images/transactions-service-generate-mock.png)

6. For methods like HEAD, ensure you manually update the method from GET to HEAD after creating the route.
7. For endpoints with multiple methods sharing the same path (such as /health or /accounts), create the non-GET methods (e.g. HEAD, POST) first.
Insomnia defaults new mock routes to the GET method. This default behavior can cause duplicate path conflicts, preventing other methods from being added later if the GET route is created first. To avoid this, always add the GET route last.
Always add the GET route last to avoid this conflict.

![`transactions-service-update-mock-request`](./images/transactions-service-update-mock-request.png)

Repeat this for:

- HEAD `/health`
- GET `/health`
- POST `/accounts` (select POST at the mock request screen)
- GET `/accounts`
- POST `/accounts/{accountId}/credit` (select POST at the mock request screen)
- POST `/accounts/{accountId}/debit` (select POST at the mock request screen)
- GET `/accounts/{accountId}`

At this point, your mock entity is complete and can be versioned along with the repository.

### Commit Your Mock Routes

After adding all routes:

1. Open the Git panel in Insomnia
2. Commit with a message:

    ```text
    feat: add complete set of mock routes for accounts service
    ```

3. Push your changes

Stop the live `accounts-service`, in the `accounts-service` repo run:

```bash
docker compose down
```

## Step 5: Creating a Feature Branch for the `Transactions-Service`

Before we copy the mock server into the `transactions-service`, let's create a dedicated feature branch:
Make sure you have the `transactions-service` project selected in Insomnia, then

1. Click on the branch name in the Git panel (lower left)
2. Create a new branch named `transactions-service-add-mock-server`
3. Click "Create" to switch to the new branch

## Step 6: Copy the Mock Server

From the `accounts-service-<your-initials>` project:

1. Open `accounts-service-mock`
2. Click the Duplicate button
3. Set the destination to the `transactions-service-<your-initials>` project

![`transactions-service-move-mock`](./images/transactions-service-move-mock.png)

## Step 7: Using the Mock in the Transactions API

Now let's consume the mock from another service.

### Extract Required Details

Select the `accounts-service-mock` in the `transactions-service` project

1. Open the mock route for POST /accounts/{`accountId`}/credit
2. Click Show URL
3. Copy:
   - The mock hash (e.g. mock_b9b5b297315c47c1964d94b2f79fa8f8)
   - The `accountId` from the path (696d0848-b30a-4c9e-b2c7-108aeacc8432 in our example)

![`transactions-service-copy-mock-details`](./images/transactions-service-copy-mock-details.png)

Now use the mock hash value as the value for the `MOCK_PATH_ID` environment variable when starting docker compose.
Run the following now to start the transactions-service and its supporting services:

```bash
MOCK_PATH_ID=<value-of-mock-hash> docker compose up -d
```

This starts:

- The self-hosted mock (`Mockbin` + Redis)
- The `transactions-api` container

Note: HashiCorp Vault is also started but not used in this module.

## Step 8: Activate the Mock in Insomnia

Back in Insomnia:

1. Go to the `accounts-service-mock`
2. Click the Test button on each route to send data to `Mockbin`

This activates and stores the mock responses inside `Mockbin`.

## Step 9: Update the Transaction Service Environment

In the `transactions-service` project:

1. Select the `Collection` tab
2. Edit the `OpenAPI env localhost:8081` environment
3. Add a `accountId` to match the one from the mock URL
   (e.g., 696d0848-b30a-4c9e-b2c7-108aeacc8432)

![`transactions-service-edit-environment`](./images/transactions-service-edit-environment.png)
![`transactions-service-update-environment-account-id`](./images/transactions-service-update-environment-account-id.png)

Now, you can run each request in the collection — all of which will interact with the mock `accounts-service`!

![`transactions-service-health-check-mock`](./images/transactions-service-health-check-mock.png)
![`transactions-service-credit-mock`](./images/transactions-service-credit-mock.png)

### Commit Environment Updates

After updating the environment:

1. Open the Git panel in Insomnia
2. Commit with a message:

    ```text
    chore: update environment to work with mocked accounts service
    ```

3. Push your changes

## Step 7: Merge Your Mock Server Branch

Once everything is working correctly:

1. Open the Git panel
2. Make sure all changes are committed and pushed
3. Create a pull request in your Git provider's interface
4. After review, merge the `transactions-service-add-mock-server` branch into your main branch
5. Do the same for the accounts-service feature branch `accounts-service-add-mock-server`

## Summary

In this module, you:

- Created a self-hosted mock using Insomnia and `Mockbin`
- Populated the mock routes from live requests
- Used Git integration to version control your mock configuration
- Configured Docker Compose to wire up the mock service
- Enabled local development of a dependent API without requiring the upstream service to be running

This allows teams to work asynchronously, reduce dependencies, and prototype faster — all while staying compliant with your version-controlled design contracts.

## What's Next?

In the next module, we'll cover how Insomnia helps you securely manage sensitive data such as tokens, credentials, and environment variables across teams.

Continue to: `Module 8: Secrets and Secure Collaboration`
