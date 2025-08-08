---
title: "Collections and Scripting"
weight: 4
---



Now that we have built up our API specification and committed our work, it's time to move into the **Collections** phase of the Insomnia workflow.

In this module, you will:

- Generate a full request collection
- Understand how a complete specification improves collection generation
- Add basic scripting to streamline account management
- Commit your work and merge it back to `main`

## Prerequisites

If you are starting the course from this module and have not completed the previous steps, you can import the Insomnia metadata to bring you up to date.

<details>
<summary><strong>Expand to see how to import the required metadata</strong></summary>

1. Clone the `insomnia-workshop-exports` repository:

   ```bash
   git clone https://github.com/KongHQ-CX/insomnia-workshop-exports.git
   ```

   > Note: `https://github.com/konghq-cx` is the address for the public version of this repository, if your company uses an enterprise git offering then this address will be updated in the course notes.

2. The structure of the repository is:

   ```text
   insomnia-workshop-exports
   |── accounts-service
   │   ├── module-02
   │   │   └── api-spec
   │   │       └── module-2-end-state.yaml
   │   ├── module-03
   │   ├── module-04
   │   │   └── api-spec
   │   │       └── module-04-end-state.yaml
   │   ├── module-05
   │   │   └── api-spec
   │   │       └── module-05-end-state.yaml
   │   ├── module-06
   │   │   └── api-spec
   │   │       └── module-06-end-state.yaml
   │   ├── module-07
   │   │   ├── api-spec
   │   │   │   └── module-07-end-state.yaml
   │   │   └── mock-spec
   │   │       └── module-07-mock-end-state.yaml
   ├── README.md
   └── transactions-service
       ├── module-07
       │   ├── api-spec
       │   │   └── module-07-end-state.yaml
       │   └── mock-spec
       │       └── module-07-mock-end-state.yaml
       └── module-08
           └── api-spec
               └── module-08-end-state.yaml
   ```

3. Import the correct "end state" YAML file for the previous module.
   - Example: If starting Module 3, import `accounts-service/module-02/api-spec/module-2-end-state.yaml` into Insomnia.

> **Note:** Some modules also include mock metadata. In those cases, import both the `api-spec` and `mock-spec` files.

</details>

This ensures you are starting the module with the correct project state.

## Introduction to Scripting in Insomnia

In this module, we will explore generating request collections and custom scripting in Insomnia. By now, your `OpenAPI` specification file fully formed and we are ready to generate our requests and utilise Insomnia scripting if required.

Insomnia supports pre-request and after-response scripting — lightweight JavaScript snippets that run either *before* a request is sent or *after* the response is received. These scripts are incredibly useful for preparing data, chaining requests, validating responses, or working with environment variables.

## Step 1: Create a Feature Branch

Before we begin working with scripts, let's create a feature branch to manage these changes:

1. In Insomnia, click on the branch name in the Git panel (lower left corner)
2. Create a new branch named: `feature/api-collections-and-scripting`
3. Click "Create" to switch to the new branch.

![`module-03-feature-branch`](./images/module-03-feature-branch.png)

> This branch will contain all of our collection requests & scripting additions, keeping them isolated until we're ready to share with the team.

## Step 2: Generate a Request Collection

Now generate the request collection based on your final spec.

1. In the sidebar under **Spec**, click the gear icon next to the words `Spec` and `Preview`.
2. Select **Generate Collection**.
![`module-03-generate-collection`](./images/module-03-generate-collection.png)
This will:

- Create structured requests for each endpoint
- Apply example request bodies and parameters
- Group requests by tag (e.g., `health`, `accounts`)
- Attach appropriate validation schemas

![`module-03-collection-full-generated`](./images/module-03-collection-full-generated.png)

> **Note:** This demonstrates the value of a complete specification — better examples, validations, and structure are automatically included.

From the collections screen we should also update our environment. To do this:

- Click on the `Base Environment` on the top right-hand side of the `Collections` screen
- Then select the edit symbol
![`module-03-select-env`](./images/module-03-select-env.png)
- Once in the environment editing window, make sure your `Base Environment`: `base_url` variable looks like this: `"{{ _.scheme }}://{{ _.host }}"`
![`module-03-base-env`](./images/module-03-base-env.png)
- And your auto generated `OpenAPI env localhost:8081` environment variables look like this:

```json
{
  "scheme": "http",
  "host": "localhost:8081"
}
```

![`module-03-local-env`](./images/module-03-local-env.png)

When leaving the environment editor window make sure the `OpenAPI env localhost:8081` environment is selected

![`module-03-local-env-selected`](./images/module-03-local-env-selected.png)

## Step 3: Understanding Pre- and After-Response Scripts

**Pre-request scripts** run *before* your request is sent. Common use cases include:

- Setting environment variables
- Authenticating with tokens
- Creating prerequisite data (e.g., a bank account before making a deposit)

> [Insomnia Docs: Pre-request scripting](https://docs.insomnia.rest/insomnia/pre-request-script)

**After-response scripts** run *after* the response is received. They're ideal for:

- Extracting data from the response and storing it for later use
- Viewing or logging response data
- Simple validation and verification tasks

> [Insomnia Docs: After-response scripting](https://docs.insomnia.rest/insomnia/after-response-script)

## Step 4: Add After-Response Script to Store `accountId`

In the "Create a new account" request:

1. Select the request.
2. Go to the **Scripts** tab.
3. Add an **After-Response Script**:

```javascript
// Extract account_id from the response and store it in the environment
let data = insomnia.response.json();
insomnia.environment.set("accountId", data.account_id);
```

This script will automatically save the `accountId` of a newly created account, making follow-up actions much easier.

### Example Response

```json
{
  "account_id": "88c09a4f-bcab-450f-a86f-d4c074f2fab9",
  "type": "savings",
  "balance": 5000
}
```

### Optional Quick Validation Example

```javascript
console.log(`Response status: ${insomnia.response.code}`);
if (insomnia.response.code !== 201) {
  console.log("Warning: Unexpected status code");
}
```

While useful during development, for production-grade testing we recommend using proper Test Suites (covered in Module 4).

Please run the request now by clicking the send button to validate it is working.

![`module-03-create-account-request`](./images/module-03-create-account-response.png)

You can also validate the `accountId` environment variable has been set by clicking on the `OpenAPI env localhost:8081` environment and selecting the edit icon.

![`module-03-local-env-account-id`](./images/module-03-local-env-account-id.png)

## Step 5: Add Pre-Request Script to Auto-Create an Account if Needed

Several requests assume the presence of a valid `accountId`:

- **Credit an account**
- **Debit an account**
- **Retrieve a single account**

To automate this:

Add a **Pre-Request Script** for each of these requests:

```javascript
if (!insomnia.environment.get("accountId")) {
  const host = insomnia.environment.get("host");
  const scheme = insomnia.environment.get("scheme");
  const baseUrl = `${scheme}://${host}`;

  const createAccountRequest = {
    url: `${baseUrl}/accounts`,
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: {
      mode: 'raw',
      raw: JSON.stringify({
        type: 'savings',
        initial_balance: 5000.00,
      }),
    },
  };

  const response = await new Promise((resolve, reject) => {
    insomnia.sendRequest(createAccountRequest, (err, resp) => {
      if (err != null) reject(err);
      else resolve(resp);
    });
  });

  if (response.code !== 201) {
    throw new Error(`Account creation failed. Status code: ${response.code}`);
  }

  const responseBody = JSON.parse(response.body.toString());
  insomnia.environment.set('accountId', responseBody.account_id);
}
```

This ensures your tests are reliable and do not depend on manual setup.

### Description of the Insomnia Pre-request Script

This pre-request script dynamically manages the creation of an `accountId` if it does not already exist in the Insomnia environment. Here is a detailed breakdown of its actions:

1. **Check for Existing `accountId`**:
   - The script first checks whether the `accountId` environment variable is already set.

2. **Retrieve Environment Variables**:
   - If `accountId` is not set, it retrieves the `host` and `scheme` environment variables.
   - It constructs the `baseUrl` by combining `scheme` and `host`.

3. **Prepare the Account Creation Request**:
   - It defines a `createAccountRequest` object configured to:
     - Use the `POST` method.
     - Target the `/accounts` endpoint at the constructed `baseUrl`.
     - Set the `Content-Type` header to `application/json`.
     - Send a JSON payload specifying the creation of a `savings` account with an initial balance of `5000.00`.

4. **Send the Account Creation Request**:
   - It uses `insomnia.sendRequest` within a `Promise` to handle asynchronous execution.
   - If the request encounters an error, it is rejected. Otherwise, the response is resolved.

5. **Handle the Response**:
   - The script checks if the HTTP status code of the response is `201` (Created).
   - If the status code is not `201`, it throws an error indicating account creation failure.

6. **Extract and Store the `accountId`**:
   - It parses the response body to extract the `account_id`.
   - It then sets the `accountId` in the environment variables for subsequent use.

### Purpose

This script ensures that each request requiring an `accountId` has one readily available, avoiding manual setup or prior dependency on external processes. It dynamically manages prerequisites, improving automation and reducing potential human error during testing or development workflows.

### Run the Requests

Once you have added the pre-request scripts to the `Credit an account`, `Debit an account` and the `Retrieve a single account` requests, execute each request by hitting the `Send` button for each one.

- `Credit an account` Response Example:

   ```json
   {
     "account_id": "88c09a4f-bcab-450f-a86f-d4c074f2fab9",
     "type": "savings",
     "balance": 5100
   }
   ```

   Note the increased balance to 5100

- `Debit an account` Response Example

   ```json
   {
     "account_id": "88c09a4f-bcab-450f-a86f-d4c074f2fab9",
     "type": "savings",
     "balance": 5000
   }
   ```

   Note the decreased balance back to 5000

- `Retrieve a single account` Response Example

   ```json
   {
     "account_id": "88c09a4f-bcab-450f-a86f-d4c074f2fab9",
     "type": "savings",
     "balance": 5000
   }
   ```

   Note the expected balance is 5000

## Step 6: Commit and Push Your Changes

Use the Insomnia Git commit menu:

1. Click on the Git dropdown in the left sidebar.
2. Select "Commit".
3. Inspect the changes.
4. If happy, stage the changes.
5. Enter a meaningful commit message: `feat: generate collection and add scripting for automatic account management`
6. Click **Commit**.
7. Click **Push** to update your remote repository.

![`module-03-final-commit`](./images/module-03-final-commit.png)

## Step 7: Raise a Pull Request

After committing and pushing:

- Create a pull request from your `feature/api-collections-and-scripting` branch into `main`.
- Once reviewed, merge your branch to keep the `main` branch up-to-date with your completed work.

## Summary

You have now:

- Generated a complete and structured collection
- Learned how and when to use pre-request and after-response scripting
- Added scripts to automate account creation and ID management
- Committed and merged your changes back into `main`

**Next:** `In Module 4 - Testing`, we'll focus on writing structured, reusable test suites for our API!
