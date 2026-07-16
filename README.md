# springboot-github-actions
springboot-github-actions

# Spring Boot GitHub Actions CI/CD Pipeline

## Overview

This project demonstrates an enterprise-style CI/CD pipeline for a Spring Boot application using:

- Spring Boot 3.x
- Java 21
- Maven
- Docker
- GitHub Actions
- Docker Hub
- GitHub Environments
- Manual Approval Gates

The pipeline follows the **Build Once, Promote Many Times** principle.

The same Docker image built during Development is promoted through:
DEV ---> UAT ---> PRODUCTION

without rebuilding the application.

---

# CI/CD Architecture
Developer
|
|
v
Git Push to Main Branch
|
|
v
+----------------------+
| Dev CI Pipeline |
| dev-build.yml |
+----------------------+
|
|
| Maven Build
| Unit Test
| Docker Build
|
v
Docker Image Created
springboot-poc:dev-<SHA>
|
|
v
+----------------------+
| UAT Deployment |
| uat-deploy.yml |
+----------------------+
|
|
v
UAT Approval Gate
|
|
v
+----------------------+
| Manual Release |
| release-manual.yml |
+----------------------+
|
|
v
Production Approval
|
|
v
Docker Release Image
springboot-poc:v1.x.x

---

# Repository Structure
.
├── src
│ └── main
│ └── java
│
├── src
│ └── test
│ └── java
│
├── Dockerfile
│
├── pom.xml
│
└── .github
└── workflows
|
├── dev-build.yml
|
├── uat-deploy.yml
|
├── release-manual.yml
|
└── release-after-uat-disabled.yml

---

# Pipeline Components

## 1. Development CI Pipeline

Workflow:
.github/workflows/dev-build.yml

Purpose:

- Checkout source code
- Setup Java 21
- Build Spring Boot application
- Execute Maven tests
- Build Docker image
- Push Docker image to registry

Flow:
Code Commit
|
v
Checkout Code
|
v
Setup Java 21
|
v
Maven Build
|
v
Docker Build
|
v
Docker Push

---

# Docker Image Strategy

The pipeline creates immutable development artifacts.

Example:
springboot-poc:dev-3940ad0

Where:
3940ad0

is the Git commit SHA.

Example:
Git Commit
3940ad0
 |
 v
Docker Image
springboot-poc:dev-3940ad0

---

# Why SHA-Based Docker Tags?

SHA tags provide:

## Traceability

You can identify exactly which source code created the image.

Example:
Production Version:
v1.8.0
Created From:
dev-3940ad0

---

## Immutability

The image:
dev-3940ad0

always represents the same build artifact.

---

## Rollback Capability

Previous releases can always be restored.

Example:
v1.7.0
v1.8.0

---

# Development Docker Tags

Example Docker Registry:
springboot-poc
dev-latest
dev-3940ad0
dev-1ea5df3
dev-12

## dev-latest

Purpose:

Points to the latest successful development build.

---

## dev-SHA

Purpose:

Immutable artifact used for promotion.

Example:
dev-3940ad0

This is the image that moves through environments.

---

# UAT Deployment Pipeline

Workflow:
.github/workflows/uat-deploy.yml

Purpose:

Deploy the validated development image into UAT.

Flow:
Dev CI Success
    |
    v
Deploy to UAT
    |
    v
Approval Required
    |
    v
UAT Deployment Complete

---

# UAT Environment Approval

GitHub Environment:
uat

Configured with:
Required Reviewers

When UAT deployment starts:
Waiting for review

appears.

The workflow pauses until approval is provided.

---

# Why UAT Approval?

UAT approval validates:

- Application functionality
- Integration testing
- Business acceptance
- Deployment readiness

After approval:
UAT PASSED

The artifact becomes eligible for release.

---

# Production Release Strategy

## Manual Release Promotion

Workflow:
.github/workflows/release-manual.yml

Production releases are intentionally manual.

Reasons:

- Prevent accidental production deployment
- Allow release scheduling
- Allow version control
- Provide audit trail

---

# Release Promotion Model

The release workflow promotes an existing Dev artifact.

Example:

Input:
Image:
dev-3940ad0
Version:
v1.8.0

Promotion:
Pull
springboot-poc:dev-3940ad0
Tag
springboot-poc:v1.8.0
Push
springboot-poc:v1.8.0

No rebuild occurs.

---

# Production Approval Gate

GitHub Environment:
production

Configured with:
Required Reviewers

Release workflow pauses:
Waiting for review
Environment:
production

Reviewer approves:
Approve and deploy

Only then the Docker image is pushed.

---

# Complete End-to-End Flow
             GitHub
Developer
|
|
v
Commit Code
|
v
Dev CI
|
|
v
springboot-poc:dev-SHA
|
v
UAT Deployment
|
v
UAT Approval
|
v
Manual Release Workflow
|
v
Production Approval
|
v
springboot-poc:v1.8.0

---

# Release Version Management

## Previous Approach

Hardcoded version:

```yaml
VERSION=v1.7.0
Problem:
Every release required modifying YAML.
Current Approach
Manual workflow input:
Release Version:

v1.8.0
Benefits:
No YAML changes
Supports multiple releases
Better governance
Release manager control
Examples:
v1.8.0
v1.9.0
v2.0.0
Disabled Automatic Release Workflow
Previous workflow:
release-after-uat.yml
Behavior:
UAT Passed
     |
     v
Automatically create release
This was disabled.
Reason:
Production releases should require explicit human approval.
Current approach:
UAT Passed

      |

Release Decision

      |

Production Release
GitHub Secrets
Stored securely in:
Repository Settings
        |
        v
Secrets and Variables
        |
        v
Actions Secrets
Configured secrets:
DOCKER_USERNAME

DOCKER_PASSWORD
Secrets are never stored inside workflow files.
Testing Scenarios Completed
Scenario 1 - Developer Commit
Action:
git push origin main
Expected:
Spring Boot Dev CI triggered
Result:
✅ Successful
Scenario 2 - Docker Build
Expected:
springboot-poc:dev-SHA

springboot-poc:dev-latest
Result:
✅ Successful
Scenario 3 - UAT Approval
Expected:
Deploy to UAT

Waiting for Review
Approval:
Approve and deploy
Result:
✅ Successful
Scenario 4 - Release Promotion
Expected:
dev-SHA

        |

v1.x.x
Result:
✅ Successful
Security Improvements
Implemented:
Repository Secrets
Environment Protection Rules
Manual Approval Gates
Immutable Docker Tags
Future Enhancements
1. Container Security Scanning
Add:
Trivy
Before release:
Docker Build

      |

Trivy Scan

      |

Release
2. Kubernetes Deployment
Add:
Kubernetes
Helm Charts
ArgoCD
GitOps
3. Deployment Strategies
Implement:
Blue/Green Deployment
Canary Deployment
Automated Rollback
4. Artifact Repository
Move from Docker Hub to:
AWS ECR
Azure Container Registry
Google Artifact Registry
5. Automated Release Notes
Generate:
v1.8.0 Release Notes
from Git commits.
Final Architecture Summary
This project implements a production-style CI/CD pipeline with:
✅ Automated Development CI
✅ Maven Build and Test
✅ Docker Image Build
✅ SHA-Based Immutable Artifacts
✅ UAT Deployment
✅ UAT Approval Gate
✅ Manual Production Release
✅ Production Approval Gate
✅ Version Controlled Releases
Core principle:
Build once, validate once, promote the same artifact across environments.
DEV IMAGE

dev-<SHA>

     |

     v

UAT VALIDATION

     |

     v

PRODUCTION RELEASE

v1.x.x
Author
Mahesh Dolas
Project:
Spring Boot GitHub Actions CI/CD POC
