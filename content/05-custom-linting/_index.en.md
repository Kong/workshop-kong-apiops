---
title: "05 - Custom Linting with Spectral in Insomnia"
weight: 6
---



In this module, we introduce the concept of extending and customising Spectral linting in Insomnia. Many organisations define internal API design standards that build on top of `OpenAPI`. Insomnia allows you to enforce these standards through custom rules using Spectral — a powerful and flexible linter for structured JSON/YAML documents.

We'll walk through the following:

- What Spectral is and how it works in Insomnia
- How to override, extend, or remove linting rules
- Creating a `.spectral.yaml` configuration
- Enforcing internal standards (such as disallowing `DELETE` endpoints)
- Using Git and semantic versioning best practices to manage these changes

## Step 1: Understanding Spectral Linting

Spectral is a rules engine built specifically for `OpenAPI`, `JSON` Schema, and `AsyncAPI` specifications. It scans your spec and flags issues related to style, structure, consistency, or breaking changes.

By default, Insomnia includes a base Spectral ruleset for `OpenAPI`. But this is just a starting point — you can add, override, or remove rules to enforce your organisation's standards.

**Useful link**  
[Insomnia: Custom Linting Guide](https://docs.insomnia.rest/inso-cli/cli-command-reference/inso-custom-linting)

## Step 2: Creating a Feature Branch

Before making any changes to our linting rules, let's create a dedicated feature branch:

1. In Insomnia, click on the branch name in the Git panel (lower left)
2. Create a new branch named `feature/custom-spectral-rules`
3. Click "Create" to switch to the new branch

![`module-05-feature-branch`](./images/module-05-feature-branch.png)

## Step 3: Prepare the Specification File

As we are removing functionality, this may be considered a breaking change. Follow semantic versioning principles.

1. Increment the version from `1.1.0` to `2.0.0` in your spec's `info.version` field
2. Stage, commit and push the changes

## Examples of Custom Spectral Rules

Let's look at a few scenarios:
(Do not add these rules to you project, they are just generic examples)

### Adding a Rule

Enforce descriptions on all `parameters`:

```yaml
rules:
  parameter-description:
    description: "All parameters must have a description"
    severity: error
    given: "$.paths[*][*].parameters[*]"
    then:
      field: description
      function: truthy
```

### Replacing a Rule

Override the default rule for operation IDs to require a specific naming pattern:

```yaml
rules:
  operation-operationId:
    description: "Use kebab-case for operation IDs"
    severity: error
    given: "$.paths[*][*]"
    then:
      field: operationId
      function: pattern
      functionOptions:
        match: "^[a-z0-9]+(-[a-z0-9]+)*$"
```

### Removing a Rule

To disable a built-in rule entirely:

```yaml
extends: spectral:oas
rules:
  info-contact: off
```

## Step 4: Enabling Custom Rules in Insomnia

To apply your own ruleset, you need to create a `.spectral.yaml` file in the root of your Git repository. This file will contain your custom rules, and Insomnia will automatically detect it and use it to lint your specification. In the following sections, we'll walk through exactly how to do this.

### Disallowing DELETE Endpoints

Let's say your internal API Standards Committee has introduced a policy against destructive operations on core resources — including `DELETE /accounts/{accountId}`.

We'll now walk through enforcing this rule using Spectral.

### Create a Custom Rule

Make sure you have your `accounts-service` code cloned out locally, and ensure you are in the correct branch.

1. Clone the repo locally if you have not done so already
2. Switch to you feature branch `feature/custom-spectral-rules` for custom linting
3. Create `.spectral.yaml` with the following contents in the root of the repository:

   > Note: you can't add the `.spectral.yaml` file from inside Insomnia, you will have to add it via another editor

    ```yaml
    extends: spectral:oas

    rules:
      no-delete-accounts:
        description: DELETE operations on accounts are not permitted
        severity: error
        given: "$.paths[*].delete"
        then:
          function: falsy
    ```

    This rule flags any path that defines a delete method.

    ```bash
    [accounts-service-sb]$ cat .spectral.yaml
    extends: spectral:oas

    rules:
      no-delete-accounts:
        description: DELETE operations on accounts are not permitted
        severity: error
        given: "$.paths[*].delete"
        then:
          function: falsy

    ```

4. Stage, commit and push your changes

### Commit Your Linting Rules

After creating the `.spectral.yaml` file:

1. Open the Git panel
2. Make sure you are in the `feature/custom-spectral-rules` branch
3. Pull the new changes into Insomnia's copy of the repository

## Step 5: Validate in Insomnia

Open your design document in Insomnia. You should now see a new linting error indicating that the `DELETE /accounts/{accountId}` operation violates your organisation's standards.

Take a moment to observe how Insomnia highlights the rule failure directly in the UI, with the custom error message.

![`module-05-spectral-fail`](./images/module-05-spectral-fail.png)

## Step 6: Fix the Linting Issue

To become compliant again:

1. Remove the `Delete Account` from the `Test` tab
2. Remove the `Delete an account` request from `Collection` tab
3. Remove the delete operation from the `/accounts/{accountId}` path in the `OpenAPI` specification file
4. Confirm the linting error disappears in Insomnia

## Step 7: Commit the Change with Context

As we are removing functionality, this may be considered a breaking change. Follow semantic versioning principles.

1. Commit using Insomnia Git: `feat!: remove DELETE /accounts/{accountId} to meet API compliance`

   The `!` in the commit message indicates a breaking change
2. Push to your remote branch

![`module-05-commit-3`](./images/module-05-commit-3.png)

## Step 8: Create a Pull Request

Once you've fixed all linting issues and properly versioned your changes:

1. Go to your Git provider's interface
2. Create a pull request from your `feature/custom-spectral-rules` branch to the main branch
3. Include context about the API standards being enforced and why the version number was incremented
4. And then merge the PR into the main branch

## Summary

In this module, you learned how to:

- Understand and modify the Spectral ruleset used by Insomnia
- Enforce organisation specific standards through a `.spectral.yaml` file
- Identify and resolve rule violations inside the Insomnia UI
- Manage changes using Git and semantic versioning best practices
- Share these API standards with your team through version control

## What's Next?

In the next module, we'll expand beyond manual tooling and show how to integrate API spec validation into a CI/CD pipeline using the `Inso CLI`.

You'll learn to:

- Run linting and tests from the command line
- Validate specs as part of GitHub Actions or Jenkins
- Prevent invalid changes from being merged

Continue to: `Module 6: CI/CD Testing`
