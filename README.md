# Lab 8 - Algonquin Pet Store (On Steroids)
**CST8915 Full-stack Cloud-native Development**
**Student:** Ilyas Zazai
**GitHub:** [Ilyzazai/26W_CST8915_Lab8](https://github.com/Ilyzazai/26W_CST8915_Lab8)

---

## Demo Video

[Click here to watch the demo video](YOUR_YOUTUBE_LINK_HERE)

---

## Task 1: Build, Push, and Deploy Your Own Docker Images

For Task 1, I forked the required service repositories, built Docker images for each service, pushed them to my Docker Hub account (`ilyaszazai`), and updated the Kubernetes configuration to use my own images.

### My Docker Hub Images

| Service | Docker Hub Image |
|---|---|
| store-front | ilyaszazai/store-front:latest |
| store-admin | ilyaszazai/store-admin-l8:latest |
| order-service | ilyaszazai/order-service:latest |
| product-service | ilyaszazai/product-service:latest |
| makeline-service | ilyaszazai/makeline-service-l8:latest |
| ai-service | ilyaszazai/ai-service-l8:latest |
| virtual-customer | ilyaszazai/virtual-customer-l8:latest |
| virtual-worker | ilyaszazai/virtual-worker-l8:latest |

The updated deployment file is: `aps-all-in-one-Task1.yaml`

---

## Task 2: Improving and Extending the Deployment

The updated deployment file for Task 2 is: `aps-all-in-one-Task2.yaml`

### Problem: Ephemeral Storage

In the original deployment, both MongoDB and RabbitMQ used ephemeral (temporary) storage. This means every time a pod was deleted or the cluster restarted, all data — including orders in the database and messages in the queue — was permanently lost. This is not acceptable for a production system.

---

### Solution 1: MongoDB High Availability and Persistent Storage

#### What I changed
- Increased MongoDB replicas from **1 to 3**
- Added a **`volumeClaimTemplates`** section so each pod automatically gets its own **PersistentVolumeClaim (PVC)** with 1Gi of storage
- Added a **`volumeMount`** pointing to `/data/db` (MongoDB's default data directory)
- Changed the MongoDB Service to **headless** (`clusterIP: None`)

#### What is a PersistentVolumeClaim (PVC)?
A PVC is a request for storage in Kubernetes. Instead of data living inside the container (which disappears when the pod restarts), the data is stored on a separate persistent disk managed by Azure. Even if the pod crashes or gets deleted, the disk remains and the new pod reattaches to it automatically.

#### What is a MongoDB Replica Set?
A MongoDB Replica Set is a group of MongoDB instances that all hold the same data. One instance is the **Primary** (handles all writes), and the others are **Secondaries** (keep copies of the data). If the Primary goes down, one of the Secondaries is automatically promoted to Primary. This means:
- **No data loss** — data is copied across all 3 replicas
- **High availability** — the system keeps running even if one pod fails
- **Automatic failover** — no manual intervention needed

#### What is a Headless Service (`clusterIP: None`)?
A headless service does not have a virtual IP. Instead, DNS queries return the individual pod IPs directly. This gives each MongoDB pod a stable, predictable DNS name like `mongodb-0.mongodb`, `mongodb-1.mongodb`, and `mongodb-2.mongodb`. These stable names are required for replica set members to find and communicate with each other consistently.

---

### Solution 2: RabbitMQ Persistent Storage

#### What I changed
- Added a **`volumeClaimTemplates`** section to the RabbitMQ StatefulSet with 1Gi of storage per pod
- Added a **`volumeMount`** pointing to `/var/lib/rabbitmq` (where RabbitMQ stores all its data including queues and messages)
- Kept the existing ConfigMap volume mount for plugins (both volumes now coexist)

#### Why this matters
Without persistent storage, if the RabbitMQ pod restarts, all queued orders that have not yet been processed by the makeline-service are permanently lost. With the PVC mounted at `/var/lib/rabbitmq`, RabbitMQ writes its queue data to the persistent disk. When the pod restarts, it reattaches to the same disk and resumes exactly where it left off — no messages are lost.

---

### Azure Managed Service Alternatives

Instead of running MongoDB and RabbitMQ ourselves inside the cluster, Azure offers managed services that handle scaling, backups, patching, and high availability automatically.

#### MongoDB → Azure Cosmos DB for MongoDB

**Azure Cosmos DB** is a fully managed, globally distributed NoSQL database that is API-compatible with MongoDB. You can point your existing MongoDB connection strings at Cosmos DB with little or no code changes.

**Why it is a good fit:**
- Automatic replication across multiple Azure regions for high availability
- Built-in backups and point-in-time restore
- Scales storage and throughput automatically based on demand
- No need to manage StatefulSets, PVCs, or replica sets manually
- Pay only for what you use

#### RabbitMQ → Azure Service Bus

**Azure Service Bus** is a fully managed enterprise message broker that supports queues and publish/subscribe topics, similar to what RabbitMQ provides.

**Why it is a good fit:**
- Messages are automatically persisted and replicated by Azure — no PVC configuration needed
- Supports the AMQP 1.0 protocol, which is compatible with the order-service and makeline-service
- Scales automatically to handle high message volumes
- Built-in dead-letter queues for failed messages
- No need to manage RabbitMQ pods, plugins, or storage volumes manually
- 99.9% availability SLA guaranteed by Microsoft

---

## Deployment Files

| File | Purpose |
|---|---|
| `Deployment Files/aps-all-in-one-Task1.yaml` | Task 1 - deployment with student's own Docker images |
| `Deployment Files/aps-all-in-one-Task2.yaml` | Task 2 - MongoDB HA + RabbitMQ persistence |
| `Deployment Files/config-maps.yaml` | RabbitMQ plugin configuration |
| `Deployment Files/secrets.yaml` | OpenAI API keys (base64 encoded) |
| `Deployment Files/admin-tasks.yaml` | Virtual customer and worker deployments |

---

*AI is used for documentation*
