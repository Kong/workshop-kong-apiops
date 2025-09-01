---
title: "APIOps with Kong Konnect and Insomnia"
weight: 0
---

# Introduction

Insomnia is an open source desktop application that simplifies designing, debugging, and testing APIs. Insomnia combines an easy-to-use interface with advanced functionality, like authentication helpers, code generation, and environment variables.


# Learning Objectives

In this workshop, you will:

* Setup a project and integrate it with GitHub
* Build an API Specification
* Understand Insomnia Collections and Scripting
* Create Test Suites
* Create custom Linting process
* Build a CI/CD testing with Inso CLI
* Create Mocks
* Understand Secrets and Secure Collaboration


---



### Workshop Overview

Since that content was first developed, Insomnia has evolved significantly, especially with the release of Insomnia v11, which introduced:

- A new Git-based storage model (no longer relying on a `.insomnia` directory)
- Enhanced scripting support (pre-request and post-response)
- Secure secret handling via local and external enterprise vaults
- Support for mocking APIs during design and development

This workshop focuses heavily on helping enterprise teams adopt modern workflows using Git-backed projects, external vault integrations, custom-linting and secrets from external vaults.

The repository currently contains a set of modules, and inside each module, documentation on how it should be run. It is intended to support the development of several onboarding artefacts:

- Collateral for in-person instructor-led workshops
- Storyboards and exercises for video-based tutorials
- Self-led labs that can be run independently by teams or individuals

### Supporting Artefacts

This workshop also makes use of two additional repositories that provide working implementations for the API specifications built throughout the modules. These implementations allow us to demonstrate a broader range of Insomnia's advanced features, including mocking and secret management.

The original `BanKonG` application combined both accounts and transactions into a single, GUI-based app. However, for the purposes of this workshop, we have split the functionality into two separate services: one for accounts and one for transactions, with no GUI.

This separation makes it easier to demonstrate developing a dependent API (`transactions-service`) using mocking when the upstream API (`accounts-service`) is not available.



## Workshop Structure and Learning Path

Participants will work through a real-world use case to design, debug, and test an API using Insomnia's full capabilities — both the desktop IDE and `Inso CLI`.

### Participants Will Learn How To

- Set up a Git-backed Insomnia project
- Design and version a full `OpenAPI` specification
- Build and organise request collections
- Write reusable test suites
- Implement pre-request and post-request scripting
- Create and consume mock APIs
- Configure custom Spectral linting rules
- Integrate with external vaults like HashiCorp Vault for secrets management
- Automate testing and linting via `CI/CD` with `Inso CLI`
- Utilise Insomnia mocking to aid API design

### The Core Use Case

Participants will build out a complete API for the `accounts-service`, including all its key operations (create, list, credit, debit, etc.). As the workshop progresses, they will also explore a second dependent service — the `transactions-service` — which integrates with the `accounts-service`.

This structure allows us to demonstrate mocking and inter-service collaboration in realistic development workflows.

## Services Used in This Workshop

To participate fully in the workshop, participants need access to the following repositories:

- [`accounts-service`](https://github.com/KongHQ-CX/accounts-service)  
  Contains an implementation for the banking Accounts API. This is the primary focus of the workshop. And we will use it to design the specification and demonstrate Insomnias features.

  Container image: `kongcx/accounts-service:1.0.0`

- [`transactions-service`](https://github.com/KongHQ-CX/transactions-service)  
  A secondary service used to demonstrate mocking and secret-sharing workflows. This service depends on the `accounts-service` to function.

  Container image: `kongcx/transactions-service:1.0.0`

These repositories contain ready-to-use Docker setups, Insomnia specs, and example configurations tailored for this workshop.

These supporting artefacts must be made available in the customer's environment. Coordinate with the customer's account team to ensure that the artefacts are uploaded to the correct locations.

### Enterprise Considerations

Many large organisations will not be able to use public GitHub or docker hub, for these situations the repositories can be imported into a customer enterprise GitHub environment and the container images into an approved artefact store.

## Workshop Structure

Each numbered directory corresponds to a key module in the learning journey:

```text
├── 00-insomnia-overview
├── 01-project-setup-and-git-integration
├── 02-designing-an-api-specification
├── 03-collections 
├── 04-scripting-and-testing 
├── 05-custom-linting 
├── 06-CICD-testing 
├── 07-mocking 
└── 08-secrets-and-secure-collaboration
```

Each subdirectory includes one or more markdown files detailing the steps, objectives, and exercises for that section. Some modules also contain screenshots, YAML configs, or example response data to help guide the learning process.

## Other Supporting Repositories

The course is designed to be completed sequentially, from **Module 01** through to **Module 08**, with each module building on the work from the previous one. **Module 00** serves as an initial overview of the Insomnia tool itself.

However, we recognise that some users may prefer to focus on specific topics rather than working through the entire sequence. For example, a participant who already feels confident with Insomnia basics, Git integration, API specification design, collections, and testing may wish to dive directly into **Module 07: Custom Linting**. To support this flexibility, users should be able to start any module by importing the correct project state, so that their Insomnia environment matches the expected starting point for that module.

To enable this, we provide a **supporting repository** containing Insomnia metadata exports for each module.

Most modules include only the Insomnia v5 **`OpenAPI`** specification file, which captures the design document, collections, environments, and tests.  

For **Modules 07** and **08**, additional **mock metadata files** are included to support the mocking workflows covered in those sections.

### Structure of the Insomnia Exports Repository

```text
insomnia-workshop-exports
├── accounts-service
│   ├── module-02
│   │   └── api-spec
│   │       └── module-02-end-state.yaml
│   ├── module-03
│   ├── module-04
│   │   └── api-spec
│   │       └── module-04-end-state.yaml
│   ├── module-05
│   │   └── api-spec
│   │       ├── module-05-01-end-state.yaml
│   │       └── module-05-02-end-state.yaml
│   ├── module-06
│   │   └── api-spec
│   │       └── module-06-end-state.yaml
│   ├── module-07
│   │   ├── api-spec
│   │   │   └── module-07-end-state.yaml
│   │   └── mock-spec
│   │       └── module-07-mock-end-state.yaml
│   └── module-08
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

Where there are two end-state files for a single module, it indicates that the module contains multiple parts, and each part has its own corresponding final state export.

This export structure ensures that users can pick up the workshop at any point while maintaining a consistent and correct project setup in Insomnia.

Like with the other supporting artefact this repository will also need to be made available in the customer's environment, the appropriate steps should be taken to make these supporting artefacts are uploaded to the correct places.

## List of Supporting Artefacts and locations

In order for a customer's employees to undertake the course, the following artefacts should be uploaded to the customer's environment (e.g., GitHub Enterprise, Nexus).

- [This repo](https://github.com/KongHQ-CX/insomnia-workshop-templates) or access Kong Academy to obtain the course materials
- [`accounts-service`](https://github.com/KongHQ-CX/accounts-service). Set this up as a `GitHub` or `GitLab` repository template. (If `BitBucket` clone and create new repository from clone)
- [`transactions-service`](https://github.com/KongHQ-CX/transactions-service). Set this up as a `GitHub` or `GitLab` repository template. (If `BitBucket` clone and create new repository from clone)
- `accounts-service` docker image `docker.io/kongcx/accounts-service:1.0.0`
- `transactions-service` docker image `docker.io/kongcx/transactions-service:1.0.0`
- `redis` docker image `docker.io/library/redis:7.4.3`
- `insomnia-mockbin` docker image `ghcr.io/kong/insomnia-mockbin:v2.0.2`
- `HashiCorp-Vault` docker image `docker.io/hashicorp/vault:1.19.3`
- [Export repository](https://github.com/KongHQ-CX/insomnia-workshop-exports) - Used to allow participants to jump into any module and load the end state from the last module

## Where to Start

The process should begin with [00-insomnia-overview](./00-insomnia-overview/00-01-insomnia-overview-README.md), and teams should progress through each module in order. By the end, they will have developed a fully defined and testable API workflow — ready to integrate into their organisation’s `APIOps` pipeline.
The content has also been structured to support flexibility. Teams can choose to start with a specific module and import all previous improvements up to that point. This allows them to dive directly into a topic of interest without having to complete every preceding module first.



# Expected Duration

* Prerequisite Environment Setup (20 minutes)
* Architectural Walkthrough (10 minutes)
* Sample Application and addressing the use cases (120 minutes)
<!-- * Observability (20 minutes) -->
* Next Steps and Cleanup (5 min)

