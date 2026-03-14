# AWS FSx – Complete Guide

## Table of Contents

1. [What is AWS FSx?](#what-is-aws-fsx)
2. [Why Use AWS FSx?](#why-use-aws-fsx)
3. [AWS FSx File System Types](#aws-fsx-file-system-types)
   - [FSx for Windows File Server](#1-amazon-fsx-for-windows-file-server)
   - [FSx for Lustre](#2-amazon-fsx-for-lustre)
   - [FSx for NetApp ONTAP](#3-amazon-fsx-for-netapp-ontap)
   - [FSx for OpenZFS](#4-amazon-fsx-for-openzfs)
4. [Key Concepts](#key-concepts)
5. [Architecture Overview](#architecture-overview)
6. [Storage Types](#storage-types)
7. [Performance](#performance)
8. [Data Protection & Backup](#data-protection--backup)
9. [Security](#security)
10. [Integration with AWS Services](#integration-with-aws-services)
11. [Pricing Model](#pricing-model)
12. [Common Use Cases](#common-use-cases)
13. [FSx vs EFS vs EBS vs S3](#fsx-vs-efs-vs-ebs-vs-s3)
14. [Getting Started](#getting-started)
15. [Best Practices](#best-practices)

---

## What is AWS FSx?

**Amazon FSx** is a fully managed, highly reliable, and scalable cloud file storage service provided by AWS. It makes it easy to launch, run, and scale feature-rich, high-performance file systems in the cloud with just a few clicks.

AWS FSx removes the complexity of setting up and maintaining file servers and storage volumes. AWS handles hardware provisioning, patching, replication, and backups, so you can focus entirely on your applications.

> **In short:** FSx = Managed file systems on AWS, available with multiple "flavors" to suit different workloads.

---

## Why Use AWS FSx?

| Benefit | Description |
|---|---|
| **Fully Managed** | No hardware to buy or manage; AWS handles infrastructure |
| **High Performance** | Sub-millisecond latencies and high throughput |
| **Broad Compatibility** | Supports Windows (SMB), Linux (NFS, Lustre), and POSIX workloads |
| **Elastic Scaling** | Scale capacity and throughput independently |
| **Native AWS Integration** | Works with IAM, CloudWatch, CloudTrail, VPC, KMS, and more |
| **Data Durability** | Multi-AZ deployments and automated backups |

---

## AWS FSx File System Types

AWS FSx offers **four** distinct file system options, each optimized for specific workloads:

---

### 1. Amazon FSx for Windows File Server

**Overview:**  
A fully managed native Windows file system built on Windows Server, accessed over the **SMB (Server Message Block)** protocol.

**Key Features:**
- Full Windows NTFS compatibility
- Supports Active Directory (AD) integration for user authentication and access control
- DFS (Distributed File System) Namespaces and Replication support
- Shadow Copies for point-in-time snapshots
- Multi-AZ high availability option
- Data deduplication to reduce storage costs

**Protocols:** SMB 2.0, 2.1, 3.0, 3.1.1

**Ideal For:**
- Windows-based enterprise applications
- Home directories and user profiles
- Microsoft SQL Server (user databases)
- SharePoint and other SMB workloads
- Applications that require Active Directory integration

**Storage Options:**
- SSD – for latency-sensitive workloads
- HDD – for broad set of workloads at lower cost

**Throughput:** Up to thousands of MB/s

---

### 2. Amazon FSx for Lustre

**Overview:**  
A fully managed high-performance parallel file system designed for workloads that require fast processing of large datasets. It is built on **Lustre**, an open-source parallel file system widely used in HPC environments.

**Key Features:**
- Sub-millisecond latencies
- Scales to hundreds of GB/s of throughput and millions of IOPS
- Native integration with Amazon S3 – link an S3 bucket to transparently read/write data
- Scratch and Persistent deployment options
- POSIX-compliant

**Protocols:** POSIX (NFS-like Lustre client)

**Ideal For:**
- High-Performance Computing (HPC)
- Machine Learning (ML) training and inference
- Financial modeling and simulations
- Video rendering and transcoding
- Genomics and life sciences

**Deployment Types:**

| Type | Use Case | Durability |
|---|---|---|
| **Scratch 1** | Temporary, short-term processing | Not replicated; optimized for cost |
| **Scratch 2** | Temporary workloads with higher throughput | Not replicated; 6× higher burst throughput than Scratch 1 |
| **Persistent 1** | Long-term workloads | Replicated within a single AZ |
| **Persistent 2** | Latency-sensitive, long-term workloads | Replicated within a single AZ; higher IOPS/throughput |

**S3 Integration:**
- Data repository associations allow FSx for Lustre to transparently read from S3
- Changes can be automatically exported back to S3

---

### 3. Amazon FSx for NetApp ONTAP

**Overview:**  
A fully managed file storage service built on **NetApp's ONTAP** file system, offering multi-protocol access and enterprise storage capabilities in the cloud.

**Key Features:**
- Multi-protocol support: NFS, SMB, and iSCSI
- Data tiering to S3 to reduce costs automatically
- SnapMirror for cross-region or cross-AZ replication
- FlexClone for near-instant, space-efficient clones
- Thin provisioning
- Automatic storage tiering (hot/cold data management)
- Multi-AZ and Single-AZ deployment

**Protocols:** NFS (v3, v4, v4.1), SMB (v2.0–3.1.1), iSCSI

**Ideal For:**
- Enterprises migrating from on-premises NetApp systems
- Workloads requiring multi-protocol access
- DevOps and CI/CD pipelines (fast clones)
- Databases requiring NFS or iSCSI
- Disaster recovery and replication scenarios

**Storage Hierarchy:**
```
FSx for ONTAP
└── Storage Virtual Machine (SVM)
    └── Volume (NFS/SMB/iSCSI)
        └── LUN (for iSCSI/SAN access)
```

---

### 4. Amazon FSx for OpenZFS

**Overview:**  
A fully managed file system built on the open-source **OpenZFS** file system, designed for Linux-based workloads that need high-performance NFS storage with rich data management features.

**Key Features:**
- NFS v3 and v4.x protocol support
- Up to 1 million IOPS with sub-millisecond latencies
- Built-in data compression (LZ4)
- Snapshots and clones with near-zero overhead
- Copies-on-write (CoW) for data integrity
- Point-in-time snapshots
- Multi-AZ support

**Protocols:** NFS (v3, v4, v4.1, v4.2)

**Ideal For:**
- Linux-based applications requiring high-performance NFS
- Web serving and content management
- Database workloads (MySQL, PostgreSQL)
- Application development and test environments
- DevOps and CI/CD workloads

---

## Key Concepts

### File System
The primary resource in FSx. When you create a file system, you specify the type (Windows, Lustre, ONTAP, OpenZFS), storage capacity, throughput, VPC, and subnet.

### Storage Virtual Machine (SVM)
Used in **FSx for ONTAP** only. An SVM is an isolated partition within the file system with its own network identity (IP), namespace, and administrative access. Multiple SVMs can reside in a single ONTAP file system.

### Volume
A logical unit of storage within a file system. In ONTAP, volumes are where data is actually stored and accessed. In OpenZFS, volumes map to ZFS datasets.

### Data Repository
In **FSx for Lustre**, a data repository is a linked S3 bucket. Files can be imported lazily from S3 or exported automatically back to S3.

### Deployment Type
Defines availability and durability:
- **Single-AZ** – Data stored in one Availability Zone
- **Multi-AZ** – Data replicated across two AZs for high availability and automatic failover

---

## Architecture Overview

```
┌────────────────────────────────────────────────────────────┐
│                          VPC                               │
│                                                            │
│  ┌──────────────────────┐     ┌──────────────────────┐    │
│  │   Availability Zone A │     │  Availability Zone B  │   │
│  │                       │     │                       │   │
│  │  ┌────────────────┐  │     │  ┌────────────────┐  │   │
│  │  │  EC2 Instances │  │     │  │  EC2 Instances │  │   │
│  │  └───────┬────────┘  │     │  └───────┬────────┘  │   │
│  │          │ mount      │     │          │ mount      │   │
│  │  ┌───────▼────────┐  │     │  ┌───────▼────────┐  │   │
│  │  │  FSx Primary   │◄─┼─────┼─►│  FSx Standby   │  │   │
│  │  │ (Active Node)  │  │repl │  │ (Standby Node) │  │   │
│  │  └───────┬────────┘  │     │  └────────────────┘  │   │
│  └──────────┼───────────┘     └──────────────────────┘   │
│             │                                              │
│    ┌────────▼────────┐                                     │
│    │  Amazon S3      │  (Optional – Lustre integration)    │
│    └─────────────────┘                                     │
└────────────────────────────────────────────────────────────┘
```

---

## Storage Types

| Storage Type | Latency | Throughput | Best For |
|---|---|---|---|
| **SSD** | Sub-millisecond | Very high | Latency-sensitive, IOPS-heavy workloads |
| **HDD** | Low | High | Throughput-heavy, large sequential reads |
| **Intelligent Tiering (ONTAP)** | Varies | Varies | Automatic hot/cold data tiering to S3 |

---

## Performance

### FSx for Windows File Server
- Throughput: 8 MB/s to 2,048 MB/s (scalable)
- Up to millions of IOPS with SSD storage

### FSx for Lustre
- Throughput: up to 1 TB/s aggregate
- IOPS: Millions
- Latency: sub-millisecond

### FSx for NetApp ONTAP
- Throughput: up to 4 GB/s
- IOPS: Hundreds of thousands
- Latency: sub-millisecond

### FSx for OpenZFS
- Throughput: up to 12.5 GB/s
- IOPS: Up to 1 million
- Latency: sub-millisecond (as low as 0.1 ms)

---

## Data Protection & Backup

### Automated Backups
- Daily automated backups retained for 0–90 days (configurable)
- Backups are incremental and stored in Amazon S3
- No impact on file system performance during backup

### Manual Backups
- Take point-in-time backups at any time
- Backups can be copied across AWS Regions and Accounts

### Snapshots (ONTAP & OpenZFS)
- Near-instant, space-efficient point-in-time snapshots
- Can be restored at the volume or file level
- No performance impact

### Multi-AZ Replication
- For Windows File Server and ONTAP, Multi-AZ deployment replicates data synchronously across two AZs
- Automatic failover with no data loss in case of AZ failure

### Replication (ONTAP)
- SnapMirror replication to another FSx for ONTAP file system
- Cross-region DR (Disaster Recovery) support

---

## Security

### Encryption
- **Encryption at rest** using AWS KMS (Key Management Service) – default AWS-managed key or customer-managed key (CMK)
- **Encryption in transit** using SMB encryption (Windows) or in-transit encryption with TLS (ONTAP, OpenZFS)

### Network Isolation
- File systems are deployed inside your **VPC**
- Access restricted to specific subnets and security groups

### Access Control
- **IAM** policies for API-level access (create, delete, describe file systems)
- **Active Directory (AD)** integration for user-level authentication on Windows and ONTAP
- **POSIX permissions** and **ACLs** for Linux workloads

### Compliance
- HIPAA eligible
- SOC 1, 2, 3
- PCI DSS
- ISO certifications

### Audit Logging
- File access auditing available for FSx for Windows File Server
- API calls logged in **AWS CloudTrail**

---

## Integration with AWS Services

| AWS Service | Integration |
|---|---|
| **Amazon EC2** | Mount FSx directly as a file system |
| **Amazon ECS / EKS** | Use FSx as persistent storage via CSI drivers |
| **AWS Lambda** | Access FSx via VPC-connected Lambda |
| **Amazon SageMaker** | Use FSx for Lustre as high-speed training data source |
| **AWS DataSync** | Migrate data to/from FSx |
| **AWS Backup** | Centralized backup management |
| **AWS CloudWatch** | Monitoring metrics, alarms, and dashboards |
| **AWS CloudTrail** | Audit API calls |
| **AWS IAM** | Access control for FSx API |
| **AWS KMS** | Encryption key management |
| **Amazon S3** | Data repository for Lustre; backup storage |
| **AWS Directory Service / AD** | User authentication for Windows/ONTAP |

---

## Pricing Model

AWS FSx pricing is based on the following dimensions (varies by file system type):

| Dimension | Description |
|---|---|
| **Storage Capacity** | Priced per GB-month provisioned |
| **Throughput Capacity** | Priced per MB/s-month provisioned (Windows, ONTAP) |
| **Backup Storage** | Priced per GB-month of backup data stored |
| **Data Transfer** | Standard AWS data transfer rates apply |
| **Requests (Lustre/S3)** | S3 API request charges apply for data repository operations |

> **Tip:** For FSx for ONTAP and OpenZFS, you only pay for the storage you provision, not the maximum capacity of the file system.

Refer to the [AWS FSx Pricing Page](https://aws.amazon.com/fsx/pricing/) for up-to-date pricing by region and file system type.

---

## Common Use Cases

### High-Performance Computing (HPC)
Use **FSx for Lustre** to run simulations, genomics workflows, seismic processing, or computational fluid dynamics at petabyte scale with sub-millisecond latencies.

### Machine Learning & AI
Use **FSx for Lustre** linked to an S3 data lake to feed training data to ML frameworks (TensorFlow, PyTorch) at high throughput, dramatically reducing training time.

### Windows Workloads
Use **FSx for Windows File Server** to lift-and-shift on-premises Windows file servers, home directories, or SQL Server databases to AWS with zero application changes.

### Enterprise Storage Migration
Use **FSx for NetApp ONTAP** to migrate from on-premises NetApp storage to AWS without changing applications, while gaining cloud agility and lower TCO.

### DevOps / CI/CD
Use **FSx for ONTAP or OpenZFS** FlexClone/Snapshot capabilities to spin up isolated dev/test environments from production snapshots in seconds.

### Containers
Use **FSx for ONTAP or OpenZFS** as persistent storage backends for containerized workloads running on Amazon EKS or ECS with the CSI driver.

### Media & Entertainment
Use **FSx for Lustre or OpenZFS** for video editing, rendering, and transcoding pipelines that require massive parallel I/O.

---

## FSx vs EFS vs EBS vs S3

| Feature | FSx for Windows | FSx for Lustre | FSx for ONTAP | FSx for OpenZFS | Amazon EFS | Amazon EBS | Amazon S3 |
|---|---|---|---|---|---|---|---|
| **Protocol** | SMB | Lustre (POSIX) | NFS/SMB/iSCSI | NFS | NFS | Block | Object (HTTP/S3 API) |
| **OS Support** | Windows | Linux | Linux/Windows | Linux | Linux | Linux/Windows | Any |
| **Scalability** | Up to 64 TB | Up to PB scale | Up to PB scale | Up to 512 TB | Elastic (auto) | Up to 64 TB | Unlimited |
| **Multi-AZ** | Yes | No (single AZ) | Yes | Yes | Yes | No | Yes (built-in) |
| **Performance** | High | Very High (HPC) | High | Very High | Moderate | Very High | Low (object) |
| **Use Case** | Windows apps | HPC / ML | Enterprise / Multi-protocol | Linux / NFS | Shared Linux FS | Single EC2 | Cold/warm data |
| **Latency** | ms | sub-ms | sub-ms | sub-ms | ms | sub-ms | ~100ms |

---

## Getting Started

### Step 1: Create an FSx File System (AWS Console)

1. Open the **Amazon FSx Console**: https://console.aws.amazon.com/fsx
2. Click **Create file system**
3. Choose your file system type (Windows, Lustre, ONTAP, or OpenZFS)
4. Configure storage capacity, throughput, VPC, and subnet
5. (Optional) Enable Multi-AZ for high availability
6. Review and click **Create file system**

### Step 2: Mount the File System

**FSx for Windows (Windows EC2 instance):**
```powershell
# Map the network drive
net use Z: \\fs-xxxxxxxxxxxxxxxxx.fsx.us-east-1.amazonaws.com\share
```

**FSx for Lustre (Linux EC2 instance):**
```bash
# Install the Lustre client
sudo amazon-linux-extras install -y lustre

# Mount the file system
sudo mkdir -p /mnt/fsx
sudo mount -t lustre -o relatime,flock \
  fs-xxxxxxxxxxxxxxxxx.fsx.us-east-1.amazonaws.com@tcp:/fsx \
  /mnt/fsx
```

**FSx for ONTAP via NFS (Linux EC2 instance):**
```bash
sudo mkdir -p /mnt/ontap
sudo mount -t nfs \
  svm-xxxxxxxxxxxxxxxxx.fs-xxxxxxxxxxxxxxxxx.fsx.us-east-1.amazonaws.com:/vol1 \
  /mnt/ontap
```

**FSx for OpenZFS via NFS (Linux EC2 instance):**
```bash
sudo mkdir -p /mnt/openzfs
sudo mount -t nfs \
  fs-xxxxxxxxxxxxxxxxx.fsx.us-east-1.amazonaws.com:/fsx \
  /mnt/openzfs
```

### Step 3: Use AWS CLI

```bash
# List all FSx file systems
aws fsx describe-file-systems

# Create an FSx for Lustre file system
aws fsx create-file-system \
  --file-system-type LUSTRE \
  --storage-capacity 1200 \
  --storage-type SSD \
  --subnet-ids subnet-xxxxxxxxxxxxxxxxx \
  --lustre-configuration DeploymentType=SCRATCH_2

# Create a backup
aws fsx create-backup \
  --file-system-id fs-xxxxxxxxxxxxxxxxx

# Delete a file system
aws fsx delete-file-system \
  --file-system-id fs-xxxxxxxxxxxxxxxxx
```

---

## Best Practices

### Performance
- Use **SSD storage** for latency-sensitive and IOPS-intensive workloads.
- For Lustre, choose **Persistent 2** for production workloads that need both high throughput and durability.
- Right-size throughput capacity – it can be scaled up after creation without downtime.
- For large parallel workloads, use **striped access patterns** with FSx for Lustre.

### Cost Optimization
- Use **HDD storage** (Windows File Server) for throughput-oriented, less latency-sensitive workloads.
- Enable **data deduplication** on FSx for Windows File Server for user file shares and general-purpose file shares.
- For ONTAP, enable **auto-tiering** to move cold data to S3 automatically.
- Use **Scratch deployment** for FSx Lustre in short-lived HPC jobs.
- Delete unused backups and snapshots regularly.

### Security
- Always enable **encryption at rest** using a customer-managed KMS key (CMK) for sensitive data.
- Restrict file system access using **VPC Security Groups** and avoid wide-open ingress rules.
- Enable **CloudTrail logging** to audit all FSx API calls.
- Use **AWS Backup** with vault lock for immutable backup retention.
- Integrate with **AWS IAM** for least-privilege access to the FSx API.

### High Availability
- Use **Multi-AZ** deployment for production Windows File Server and ONTAP workloads.
- Define **maintenance windows** during off-peak hours for system updates.
- Test **failover procedures** periodically to validate RTO/RPO.

### Monitoring
- Use **Amazon CloudWatch** to monitor throughput, IOPS, storage, and free capacity.
- Set up **CloudWatch Alarms** for low free storage capacity or high latency.
- Enable **file access auditing** (Windows File Server) for compliance requirements.

---

## Summary Table – Which FSx to Choose?

| If you need… | Use |
|---|---|
| Native Windows SMB with Active Directory | FSx for Windows File Server |
| Fastest possible parallel I/O for HPC or ML | FSx for Lustre |
| Multi-protocol (NFS + SMB + iSCSI) enterprise storage | FSx for NetApp ONTAP |
| High-performance Linux NFS with ZFS features | FSx for OpenZFS |

---

## References

- [Amazon FSx Official Documentation](https://docs.aws.amazon.com/fsx/)
- [FSx for Windows File Server Docs](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/)
- [FSx for Lustre Docs](https://docs.aws.amazon.com/fsx/latest/LustreGuide/)
- [FSx for NetApp ONTAP Docs](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/)
- [FSx for OpenZFS Docs](https://docs.aws.amazon.com/fsx/latest/OpenZFSGuide/)
- [AWS FSx Pricing](https://aws.amazon.com/fsx/pricing/)
- [AWS FSx FAQs](https://aws.amazon.com/fsx/faqs/)
