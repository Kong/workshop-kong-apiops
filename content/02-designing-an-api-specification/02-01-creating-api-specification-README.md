# Module 2: Building the API Specification

In this module, we will design and build a structured `OpenAPI` specification for the Accounts API, using Insomnia's built-in Git integration and linting features.

By the end of this module, you'll have:

- Created a Git-tracked design document
- Defined a valid and useful `OpenAPI` specification
- Built endpoints with proper schemas, examples, and metadata
- Practiced clean Git commit habits
- Prepared the foundation for generating collections and creating tests

> **Important:** We will commit and push changes at key points, following best practice.

## Step 1: Create a Feature Branch in Insomnia

In the previous chapter you should have created the `feature/api-spec-design` branch, if you did not or are coming to this chapter directly then please create the feature branch now and switch to using it.

1. Open your `accounts-service` project in Insomnia.
2. In the lower left Git panel, click on the branch name.
3. Create a new branch named: `feature/api-spec-design`
4. Click **Create** to switch to it.

> Working on feature branches makes collaboration easier and prevents conflicts.

## Step 2: Create a New API Specification

Let's create a new `OpenAPI` design document inside the Git project.

1. Click the `+ Create` button and choose **Design Document**.
   ![`create-document`](./images/accounts-service-create-design-document.png)
2. Set:
   - Name: `accounts-service`
   - Location: `api/accounts-service.yaml`
3. Click **Create Document**.
![`create-document-details`](./images/accounts-service-design-document-details.png)
You'll now have an empty spec file, fully tracked in Git.

### Commit and Push Your Changes

Use the Insomnia Git commit menu:

1. Click on the Git dropdown in the left sidebar.
2. Select "Commit".
3. Inspect the changes.
4. If happy, stage the changes.
5. Enter a meaningful commit message: `chore: create initial OpenAPI specification document`
6. Click **Commit**.
7. Click **Push** to update your remote repository.

![`commit-1`](./images/accounts-service-commit-screen.png)

## Step 3: Add a Minimal Valid `OpenAPI` Structure

Let's start with the simplest valid `OpenAPI` structure.

Paste this into your design document:

```yaml
openapi: 3.0.3
info:
  title: Accounts API
  version: 1.0.0
paths: {}
```

You will immediately see linting feedback at the bottom from Spectral.

- Missing description
- Missing contact
- Missing servers

In order to fix these here's what to add and why:

| Issue | Fix | Why It Matters |
|-------|-----|----------------|
| `info.description` | Add a description to the `info` block | Gives consumers an overview of what the API does and its intended use case |
| `info.contact.name` | Add the maintainer or team name under `info.contact.name`| Makes it clear who to reach out to for support, clarification, or ownership |
| `info.contact.email`  | Add an email address under `info.contact.email` | Provides a direct contact point for API consumers or stakeholders |
| `servers` | Define a `servers` block at the root level | Provides users and tools with the base URLs for different environments (e.g., local, test, production) |

## Step 4: Update the Info Section

Update the info section and add supporting metadata to fix the linting errors:

```yaml
openapi: 3.0.3
info:
  title: Accounts API
  version: 1.0.0
  description: This API manages account information and balances.
  contact:
    email: stephen.brown@konghq.com

servers:
  - url: http://localhost:8081
    description: Local development server

paths: {}
```

## Step 5: Add the `/accounts` Endpoint

Now we define the `POST /accounts` operation to create new accounts.

Replace the `paths` section with this:

```yaml
paths:
  /accounts:
    post:
      summary: Create account
      responses:
        '201':
          description: Account created
```

You will notice that this new endpoint will trigger linter warnings

## What's Missing?

While this minimal spec is valid YAML, it will:

- Trigger Spectral linter warnings
- Fail to generate a usable request body in Insomnia
- Be incomplete for testing and mocking

| Missing Element | Why It Matters |
|-----------------|----------------|
| `description` | Without it, documentation lacks detail |
| `operationId` | Required by tools to uniquely identify the operation |
| `tag` | Used by Insomnia to group requests |
| `global tag` | Define the tag at the top level, Ensures documentation tools can display it properly |
| `requestBody` | Needed to define the payload of the request |
| `components/schemas` | Centralises definitions for validation and reuse |

To correct these issues lets replace the `paths` section with a more complete example

```yaml
paths:
  /accounts:
    post:
      tags:
        - accounts
      summary: Create account
      description: Creates a new account with an initial balance. The account ID is automatically generated.
      operationId: createAccount
      requestBody:
        description: Details of the new account
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateAccountRequest'
            examples:
              savings_account:
                summary: Savings account example
                value:
                  type: savings
                  initial_balance: 5000
      responses:
        '201':
          description: Account created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Account'
              examples:
                created_account:
                  summary: Created account
                  value:
                    account_id: "123e4567-e89b-12d3-a456-426614174000"
                    type: savings
                    balance: 5000
```

Also create the `components` section:

```yaml
components:
  schemas:
    CreateAccountRequest:
      type: object
      required:
        - type
        - initial_balance
      properties:
        type:
          type: string
          description: Type of account (e.g. checking, savings)
        initial_balance:
          type: number
          format: double
          description: Initial deposit amount
    Account:
      type: object
      properties:
        account_id:
          type: string
          format: uuid
        type:
          type: string
        balance:
          type: number
```

Create a global tag section and add the accounts tag:

```yaml
tags:
  - name: accounts
    description: Accounts endpoints
```

### Reflection

Why does the completeness of these sections matter?

Incomplete or missing `responses` would lead to:

- No machine-readable response format
- Can't validate tests
- Can't generate mocks

`Schema` definitions enable:

- Request and mock generation in Insomnia
- Linting and validation against expected types
- SDK and client generation using tools like `OpenAPI` Generator
- Better documentation previews

Without them, we would have:

| Problem | Improvement |
|---------|-------------|
| `Unclear structure` | Enables mocks and validation |
| `No reuse` | Centralised schema management |
| `No examples` | Improves docs and auto-generated requests which aids better test coverage |

### Commit and Push Your Changes for the New Endpoints

This is where `OpenAPI` becomes a contract between teams and systems, so this seems like a good point to commit our changes.
Use the Insomnia Git commit menu:

1. Click on the Git dropdown in the left sidebar.
2. Select "Commit".
3. Inspect the changes.
4. If happy, stage the changes.
5. Enter a meaningful commit message: `feat: add /accounts endpoint and associated schemas`
6. Click **Commit**.
7. Click **Push** to update your remote repository.

![`commit-screen-2`](./images/accounts-service-commit-screen-2.png)

Now that we've constructed basic endpoint in our specification and explored the importance of a complete, linted API design, we are ready to import a fully completed specification for the `accounts-service`.

## Step 6: Paste the Completed Specification

Now, paste the full final spec to finish Module 2 cleanly.

<details>
<summary><strong>Click to expand and copy the full OpenAPI specification</strong></summary>

```yaml
openapi: 3.0.3
info:
  title: Accounts API
  version: 1.0.0
  description: This API manages account information and balances.
  contact:
    email: stephen.brown@konghq.com
servers:
  - url: http://localhost:8081
    description: Local development server
tags:
  - name: accounts
    description: Operations related to account management (retrieval and balance updates)
  - name: health
    description: Health check endpoints for monitoring service status

paths:
  /health:
    get:
      tags:
        - health
      operationId: healthCheck
      summary: Health check endpoint
      description: Returns the API's health status
      responses:
        '200':
          description: API is healthy
          content:
            application/json:
              schema:
                type: string
              examples:
                success:
                  summary: Healthy response
                  value: "Accounts API is running"
    head:
      tags:
        - health
      operationId: healthCheckHead
      summary: Health check endpoint (HEAD)
      description: Returns the API's health status without body
      responses:
        '200':
          description: API is healthy

  /accounts:
    get:
      tags:
        - accounts
      operationId: listAccounts
      summary: List all accounts
      description: Returns a list of all accounts with basic details.
      responses:
        '200':
          description: A list of accounts was successfully retrieved
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Account'
              examples:
                multiple_accounts:
                  summary: Multiple accounts example
                  value:
                    - account_id: "123e4567-e89b-12d3-a456-426614174000"
                      type: "checking"
                      balance: 1500.00
                    - account_id: "987fcdeb-51a2-43e7-89bc-765432198765"
                      type: "savings"
                      balance: 5000.00
        '500':
          description: Failed to retrieve accounts due to internal server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                lock_error:
                  summary: Database lock error
                  value:
                    error_code: "INTERNAL_ERROR"
                    message: "Lock error: failed to acquire lock while listing accounts"
    post:
      tags:
        - accounts
      operationId: createAccount
      summary: Create a new account
      description: Creates a new account with an initial balance. The account ID is automatically generated.
      requestBody:
        description: Details for the new account including type and initial balance. The account ID will be automatically generated.
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateAccountRequest'
            examples:
              checking_account:
                summary: Create checking account
                value:
                  type: "checking"
                  initial_balance: 1000.00
              savings_account:
                summary: Create savings account
                value:
                  type: "savings"
                  initial_balance: 5000.00
      responses:
        '201':
          description: Account created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Account'
              examples:
                new_account:
                  summary: Newly created account
                  value:
                    account_id: "123e4567-e89b-12d3-a456-426614174000"
                    type: "checking"
                    balance: 1000.00
        '400':
          description: Failed to create account due to invalid input
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                invalid_balance:
                  summary: Invalid initial balance
                  value:
                    error_code: "INVALID_INPUT"
                    message: "Failed to create account: Initial balance must be non-negative"
        '500':
          description: Failed to create account due to internal server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                creation_error:
                  summary: Account creation failed
                  value:
                    error_code: "INTERNAL_ERROR"
                    message: "Failed to create account: Internal server error occurred"

  /accounts/{accountId}:
    get:
      tags:
        - accounts
      operationId: getAccountById
      summary: Retrieve a single account
      description: Returns details for the specified account including current balance.
      parameters:
        - name: accountId
          in: path
          required: true
          description: The UUID of the account to retrieve
          schema:
            type: string
            format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
      responses:
        '200':
          description: Account details retrieved successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Account'
              examples:
                checking_account:
                  summary: Example checking account
                  value:
                    account_id: "123e4567-e89b-12d3-a456-426614174000"
                    type: "checking"
                    balance: 1500.00
        '404':
          description: Account lookup failed - specified account ID does not exist
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                not_found:
                  summary: Account not found error
                  value:
                    error_code: "NOT_FOUND"
                    message: "Failed to retrieve account: ID 123e4567-e89b-12d3-a456-426614174000 not found"
        '500':
          description: Failed to retrieve account details due to internal server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                server_error:
                  summary: Internal server error
                  value:
                    error_code: "INTERNAL_ERROR"
                    message: "Failed to retrieve account: Internal server error occurred"

  /accounts/{accountId}/debit:
    post:
      tags:
        - accounts
      operationId: debitAccount
      summary: Debit an account
      description: Decreases the account's balance by the specified amount.
      parameters:
        - name: accountId
          in: path
          required: true
          description: The UUID of the account to debit
          schema:
            type: string
            format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
      requestBody:
        description: Specifies the amount to debit from the account balance.
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateBalanceRequest'
            examples:
              small_debit:
                summary: Small debit amount
                value:
                  amount: 100.00
              large_debit:
                summary: Large debit amount
                value:
                  amount: 1000.00
      responses:
        '200':
          description: Account debited successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Account'
              examples:
                successful_debit:
                  summary: Account after successful debit
                  value:
                    account_id: "123e4567-e89b-12d3-a456-426614174000"
                    type: "checking"
                    balance: 900.00
        '400':
          description: Debit operation failed due to insufficient funds or invalid amount
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                insufficient_funds:
                  summary: Insufficient funds error
                  value:
                    error_code: "INSUFFICIENT_FUNDS"
                    message: "Failed to debit account: Insufficient funds - balance is 900.00, attempted to debit 1000.00"
        '404':
          description: Debit operation failed - account not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                not_found:
                  summary: Account not found
                  value:
                    error_code: "NOT_FOUND"
                    message: "Failed to debit account: Account does not exist"
        '500':
          description: Failed to process debit operation due to internal server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                server_error:
                  summary: Internal error
                  value:
                    error_code: "INTERNAL_ERROR"
                    message: "Failed to debit account: Internal server error occurred"

  /accounts/{accountId}/credit:
    post:
      tags:
        - accounts
      operationId: creditAccount
      summary: Credit an account
      description: Increases the account's balance by the specified amount.
      parameters:
        - name: accountId
          in: path
          required: true
          description: The UUID of the account to credit
          schema:
            type: string
            format: uuid
          example: "123e4567-e89b-12d3-a456-426614174000"
      requestBody:
        description: Specifies the amount to credit to the account balance.
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateBalanceRequest'
            examples:
              small_credit:
                summary: Small credit amount
                value:
                  amount: 500.00
              large_credit:
                summary: Large credit amount
                value:
                  amount: 5000.00
      responses:
        '200':
          description: Account credited successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Account'
              examples:
                successful_credit:
                  summary: Account after successful credit
                  value:
                    account_id: "123e4567-e89b-12d3-a456-426614174000"
                    type: "checking"
                    balance: 2000.00
        '400':
          description: Credit operation failed due to invalid amount
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                invalid_amount:
                  summary: Invalid amount error
                  value:
                    error_code: "INVALID_INPUT"
                    message: "Failed to credit account: Amount must be positive"
        '404':
          description: Credit operation failed - account not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                not_found:
                  summary: Account not found
                  value:
                    error_code: "NOT_FOUND"
                    message: "Failed to credit account: Account does not exist"
        '500':
          description: Failed to process credit operation due to internal server error
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                server_error:
                  summary: Internal error
                  value:
                    error_code: "INTERNAL_ERROR"
                    message: "Failed to credit account: Internal server error occurred"

components:
  schemas:
    Account:
      type: object
      description: Represents a bank account with its core details including ID, type, and current balance.
      example:
        account_id: "123e4567-e89b-12d3-a456-426614174000"
        type: "checking"
        balance: 1500.00
      properties:
        account_id:
          type: string
          format: uuid
          description: Unique identifier for the account (UUID v4)
          example: "123e4567-e89b-12d3-a456-426614174000"
        type:
          type: string
          description: The type of the account (e.g., checking, savings)
          example: "checking"
        balance:
          type: number
          format: double
          description: The current balance of the account in the account's currency
          example: 1500.00

    CreateAccountRequest:
      type: object
      description: Request payload for creating a new account.
      example:
        type: "savings"
        initial_balance: 5000.00
      required:
        - type
        - initial_balance
      properties:
        type:
          type: string
          description: The type of account to create (e.g., checking, savings)
          example: "savings"
        initial_balance:
          type: number
          format: double
          description: The initial balance to fund the account with
          example: 5000.00

    UpdateBalanceRequest:
      type: object
      description: Request payload for updating an account's balance via credit or debit operations.
      example:
        amount: 100.00
      required:
        - amount
      properties:
        amount:
          type: number
          format: double
          description: The amount to credit or debit from the account
          example: 100.00

    ErrorResponse:
      type: object
      description: Standard error response structure for all error cases.
      example:
        error_code: "NOT_FOUND"
        message: "The requested account does not exist."
      required:
        - error_code
        - message
      properties:
        error_code:
          type: string
          description: A standardized error code identifying the type of error
          example: "NOT_FOUND"
          enum:
            - INTERNAL_ERROR
            - NOT_FOUND
            - INSUFFICIENT_FUNDS
            - INVALID_INPUT
        message:
          type: string
          description: A human-readable description of the error
          example: "The requested account does not exist."
```

</details>

This ensures your design document is complete, Spectral-compliant, and production-ready.

### Endpoint Overview

Here is what each endpoint in the spec is responsible for:

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/health` | GET | Returns basic availability and health of the service |
| `/health` | HEAD | Same as GET but with no response body |
| `/accounts` | GET | Lists all registered accounts |
| `/accounts` | POST | Creates a new account with an initial balance |
| `/accounts/{accountId}` | GET | Fetches full account details by ID |
| `/accounts/{accountId}/debit` | POST | Deducts funds from the specified account |
| `/accounts/{accountId}/credit` | POST | Adds funds to the specified account |

All endpoints are fully documented with:

- summary, description, `operationId`, and tags
- Schemas referenced using `$ref` to reduce duplication
- Structured error handling for 4xx and 5xx cases
- Inline and reusable examples for request and response content

### Final Commit and Push

Use the Insomnia Git commit menu:

1. Click on the Git dropdown in the left sidebar.
2. Select "Commit".
3. Inspect the changes.
4. If happy, stage the changes.
5. Enter a meaningful commit message: `chore: paste completed Accounts API specification`
6. Click **Commit**.
7. Click **Push** to update your remote repository.

After reviewing your changes, create a pull request from your feature branch into main. Once approved, merge your branch to keep the main branch up-to-date with your completed work.

## Summary

You have now:

- Created a Git-backed `OpenAPI` spec in Insomnia
- Built the `/accounts` endpoint with proper structure
- Resolved all linter warnings
- Committed your changes at logical milestones

You are ready to move on!

**Next:** In `Module 3: Collections`, we'll use this complete specification to generate a request collection.
