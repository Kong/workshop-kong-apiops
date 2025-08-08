---
title: "API Management with Kong Konnect"
weight: 0
---

# Introduction

Kong Konnect is an API lifecycle management platform delivered as a service. The management plane is hosted in the cloud by Kong, while the runtime environments are deployed in your AWS accounts. Management plane enables customers to securely execute API management activities such as create API routes, define services etc. Runtime environments connect with the management plane using mutual transport layer authentication (mTLS), receive the updates and take customer facing API traffic.

# Learning Objectives

In this workshop, you will:

* Get an architectural overview of Kong Konnect platform.
* Set up Konnect runtime on Amazon Elastic Kubernetes Service (EKS).
* Learn what are services, routes and plugin.
* Deploy a sample microservice and access the application using the defined route.
* Use the platform to address the following API Gateway use cases
    * Proxy caching
    * Authentication and Authorization
    * Response Transformer
    * Request Callout
    * Rate limiting
    <!-- * Invoke AWS Lambda -->
    <!-- * Learn how to do observability -->

* And the following AI Gateway use cases
    * Prompt Engineering
    * LLM-based Request and Reponse transformation
    * Semantic Caching
    * Token-based Rate Limiting
    * Semantic Routing
    * RAG - Retrieval-Augmented Generation

# Expected Duration

* Pre-Requisite Environment Setup (20 minutes)
* Architectural Walkthrough (10 minutes)
* Sample Application and addressing the use cases (60 minutes)
<!-- * Observability (20 minutes) -->
* Next Steps and Cleanup (5 min)
