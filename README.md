# CST8915 – Lab 8 Submission (Full-stack Cloud-native Development)

## Student

- **Name:** Ilyas Zazai
- **Course:** CST8915 – Full-stack Cloud-native Development
- **Lab:** Lab 8

---

## Demo Video

- `<ADD_YOUR_YOUTUBE_LINK_HERE>`

---

## Repository

- `<ADD_YOUR_GITHUB_REPOSITORY_LINK_HERE>`

---

## Overview

This lab focused on deploying the Algonquin Pet Store application to Azure Kubernetes Service (AKS) using Kubernetes manifests. The environment included multiple microservices such as `store-front`, `store-admin`, `order-service`, `product-service`, `makeline-service`, `ai-service`, `mongodb`, and `rabbitmq`.

The objective of the lab was to deploy the application using AKS, verify service creation, and prepare the required deployment files for Task 1 and Task 2. During the setup process, Azure subscription restrictions affected the available VM sizes and quota in some regions, so alternative AKS configurations had to be tested.

---

## Work Completed

### 1. Repository Preparation
- Cloned the Lab 8 repository locally.
- Replaced the `secrets.yaml` file with my own local copy.
- Protected the secret file from being pushed publicly by using `.gitignore`.
- Created the following required files:
  - `aps-all-in-one-Task1.yaml`
  - `aps-all-in-one-Task2.yaml`

### 2. Azure and AKS Setup
- Signed in to Azure using Azure CLI.
- Created a resource group:
  - `AlgonquinPetStoreRG`
- Attempted AKS deployment in different configurations because of subscription VM-size and quota limitations.
- Successfully created an AKS cluster in `centralus` with:
  - Cluster name: `AlgonquinPetStoreCluster`
  - Node count: `1`
  - VM size: `Standard_B2ps_v2`
- Connected `kubectl` to the AKS cluster and verified node readiness.

### 3. Kubernetes Deployment
- Applied:
  - `config-maps.yaml`
  - `secrets.yaml`
  - `aps-all-in-one.yaml`
- Verified that Kubernetes resources were created.
- Confirmed that some infrastructure services started successfully:
  - `mongodb`
  - `rabbitmq`

### 4. Deployment Issues Observed
During deployment, application pods did not fully run successfully. Some containers showed:
- `ErrImagePull`
- `CrashLoopBackOff`
- `Error`
- `Init:0/1`

The main application services that had issues included:
- `ai-service`
- `makeline-service`
- `order-service`
- `product-service`
- `store-admin`
- `store-front`

A likely reason was platform/image compatibility and deployment-image issues, while the official MongoDB and RabbitMQ containers were able to start successfully.

---

## Current Status

At the time of this draft:
- The AKS cluster was successfully created and connected.
- Kubernetes manifests were applied.
- Core stateful services (`mongodb` and `rabbitmq`) were running.
- Some custom application containers failed and still require further troubleshooting.
- Cloud resources were prepared for cleanup to avoid unnecessary charges.

---

## Task 1 File

`aps-all-in-one-Task1.yaml` was created as the Task 1 submission file. It will be updated with the final working image references and deployment changes after troubleshooting is completed.

---

## Task 2 File

`aps-all-in-one-Task2.yaml` was created as the Task 2 submission file. It will be updated later with:
- MongoDB replica configuration
- Headless service configuration
- Persistent volume settings
- RabbitMQ persistent storage configuration

---

## Design / Troubleshooting Notes

- Azure subscription restrictions affected which VM sizes could be used in each region.
- The default AKS VM size was not allowed in `eastus`.
- A second AKS deployment in `centralus` succeeded with an allowed VM size.
- `kubectl` initially pointed to the Docker Desktop Kubernetes context, so the kubeconfig had to be corrected to use the AKS context.
- After deployment, the running status of MongoDB and RabbitMQ showed that the Kubernetes environment itself was working, but several application containers still needed image or compatibility fixes.

---

## Cleanup

To avoid charges, Azure resources can be deleted with:

```bash
az group delete --name AlgonquinPetStoreRG --yes --no-wait
```

---

## Files for Final Submission

Later, the final submission repository should include:
- `README.md`
- `Deployment Files/aps-all-in-one-Task1.yaml`
- `Deployment Files/aps-all-in-one-Task2.yaml`
- `Deployment Files/admin-tasks.yaml`
- `Deployment Files/config-maps.yaml`

Do **not** upload:
- `Deployment Files/secrets.yaml`

---

## Conclusion

This lab established the Azure Kubernetes Service environment and deployed the Lab 8 application manifests. The AKS cluster was created successfully, Kubernetes resources were applied, and the supporting services were partially operational. Additional troubleshooting is still needed to make all custom application services run successfully and to complete the final Task 1 and Task 2 deployment files.


*AI is used for documentation*