---
title: "06 - CI/CD Testing with Inso CLI"
weight: 7
---



In this module, we introduce the **`Inso CLI`** — the command-line companion to the Insomnia desktop application. `Inso CLI` is designed for use in terminal workflows and automation pipelines, bringing the power of Insomnia into your CI/CD systems.

> Official docs: [`Inso CLI` Introduction](https://docs.insomnia.rest/inso-cli/introduction)

## Step 1: Understanding `Inso CLI`

`Inso CLI` is built on Node.js and uses the same core libraries as the Insomnia app. It allows you to:

- Run **linting** against your `OpenAPI` spec
- Execute **unit tests** defined in Insomnia
- Export and execute **collections**
- Integrate seamlessly into CI/CD pipelines

Its primary use case is to **enforce API quality and consistency** at key stages in the software delivery lifecycle — not just on a developer's machine, but in the merge and release process itself.

## Step 2: Understanding the CI/CD Benefits

While Insomnia's desktop app ensures quality during local development, organisations often need to guarantee that **central standards are applied across all environments**. This includes:

- Enforcing consistent API structure and design
- Preventing non-compliant specs from being merged
- Ensuring functionality through test coverage

`Inso CLI` supports these goals by making automated validation a natural part of your pipeline.

A simple CI workflow might look like this:

```plaintext
1. Checkout branch
2. Setup Node.js and install Inso
3. Run linting with `inso lint spec`
4. Run unit tests with `inso run test`
```

**Note:** If your organisation supports GitHub Actions, you can use the Setup `Inso` GitHub Action.
[`Inso-GitHub-Action-Docs`](https://docs.insomnia.rest/inso-cli/continuous-integration#setup-inso-workflow)
[`Inso-GitHub-Action`](https://github.com/marketplace/actions/setup-inso)

## Step 3: Core Commands for This Module

We'll focus on two key `Inso CLI` commands:

### `inso lint spec`

Lints your `OpenAPI` specification using Spectral.

### `inso run test`

Executes unit tests created in Insomnia.

Full command reference available here: [`Inso CLI` Command Reference](https://docs.insomnia.rest/inso-cli/cli-command-reference)

## Step 4: Using `inso lint spec`

This command checks your `OpenAPI` document against Spectral rules.

### Syntax

```bash
inso lint spec [identifier]
```

You'll be prompted to select the specification if no identifier is provided.

### Options

| Option | Description |
|--------|-------------|
| `--workingDir` / -w | Specify a working directory to locate your Insomnia spec file or export |

### Example for our Accounts API

```bash
# Get the ID of the specification and store in a variable
INSO_SPEC_ID=$(awk '/spc_/{print $2}'  api/accounts-service.yaml)
# Lint the specification
inso lint spec -w api/accounts-service.yaml $INSO_SPEC_ID
```

### Expected Output

```text
No linting errors or warnings.
```

This ensures your spec meets both default and custom Spectral standards (see Module 7).

## Step 5: Using `inso run test`

This command executes unit tests written inside Insomnia.

### Command Syntax

```bash
inso run test [identifier]
```

You can target either a test suite or document, and optionally filter by test name.

### Common Options

| Option | Alias | Description |
|--------|-------|-------------|
| `--env` | -e | Environment name or ID |
| `--testNamePattern` | -t | Regex to run specific tests |
| `--reporter` | -r | Output format: spec, list, dot, min, progress |
| `-bail` | -b | Exit after the first failure |
| `--keepFile`|| Keep generated test file (debugging) |

### Run a Single Test

Firstly just run one test by specifying it name:

```bash
INSO_SPEC_ID=$(awk '/spc_/{print $2}'  api/accounts-service.yaml)
inso run test -w api/accounts-service.yaml $INSO_SPEC_ID --env "OpenAPI env localhost:8081" --testNamePattern "Create New Account" "Accounts Test S
uite"
```

### Expected Output for `inso run test`

```bash
  Accounts Test Suite
    ✓ Create New Account (249ms)

  1 passing (252ms)
```

### Run All Tests

Now run all the tests

```bash
INSO_SPEC_ID=$(awk '/spc_/{print $2}'  api/accounts-service.yaml)
inso run test -w api/accounts-service.yaml $INSO_SPEC_ID --env "OpenAPI env localhost:8081" "Accounts Test Suite"
```

### Expected Output for targeted `inso run test`

```bash
  Accounts Test Suite
    ✓ Create New Account (269ms)
    ✓ List All Accounts
    ✓ Health Check
    ✓ Credit Account
    ✓ Debit Account
    ✓ Get Account By ID

  6 passing (355ms)
```

## Step 6: Creating a CI Configuration File

Your organisation may use a different CI/CD system than GitHub Actions — for example, GitLab CI, Jenkins, or Azure Pipelines. The example below provides a **GitHub Actions workflow** that can easily be adapted to fit other pipeline technologies or formats.

This configuration demonstrates how to automate validation of your `OpenAPI` specification and run your Insomnia test suite during pull requests.

### Example: GitHub Actions – `.github/workflows/api-validation.yml`

```yaml
name: API Validation
on:
  pull_request:
    paths:
      - 'api/**'
      - '.spectral.yaml'
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Inso CLI
        uses: kong/setup-inso@v1
        with:
          inso-version: 7.x
      
      - name: Lint API Specification
        run: inso lint spec
      
      - name: Run API Tests
        run: inso run test "Accounts Test Suite" --env "OpenAPI env localhost:8081"
```

This workflow:

- Runs automatically on pull requests that modify the API spec or `.spectral.yaml` file
- Installs and configures the `Inso CLI`
- Lints the `OpenAPI` spec using Spectral
- Executes your Insomnia test suite (Accounts Test Suite) against the specified environment

Feel free to replace or extend these steps depending on how your team manages environments, branches, or naming conventions in your pipelines.

## Note on Exports

If needed, the raw `OpenAPI` specification can be extracted from the design document using the `Inso CLI` export command.

To output the specification to `stdout` — without any Insomnia-specific metadata — run:

```bash
INSO_SPEC_ID=$(awk '/spc_/{print $2}' api/accounts-service.yaml)
inso export spec -w ./api/accounts-service.yaml $INSO_SPEC_ID
```

Alternatively, you can export directly to a file:

```bash
INSO_SPEC_ID=$(awk '/spc_/{print $2}' api/accounts-service.yaml)
inso export spec -w ./api/accounts-service.yaml $INSO_SPEC_ID -o /tmp/accounts-service.yaml
```

This is especially useful when working in CI/CD pipelines or when passing the clean `OpenAPI` document to other tools.

## Summary

In this module, we:

- Introduced `Inso CLI` as the automation tool for Insomnia
- Learned how to lint `OpenAPI` specs in `CI/CD` pipelines
- Learned how to run unit tests for the Accounts API from the terminal
- Saw an example `CI` configuration file to automate API validation

By using `Inso CLI` in your pipelines, you ensure your APIs remain compliant and functional throughout the development lifecycle — not just at the developer's desk.

## What's Next?

In the next module, we'll look at how to deploy and use mock servers, allowing teams to prototype and test against your APIs even before the backend is implemented.

Continue to: `Module 7: Mocking`
