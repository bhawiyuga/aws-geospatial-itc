---
title: Introdution to Cloud Computing - Big Geodata Processing course (UT-ITC)
layout: default
---

# Introdution to Cloud Computing 
This pre-class reading note is part of Big Geodata Processing course of master program teached in Faculty of ITC, University of Twente.

---

## Table of Contents
1. [Introduction to Cloud Computing](#introduction-to-cloud-computing)
2. [Why Cloud for Geodata?](#why-cloud-for-geodata)
3. [Cloud Service Models](#cloud-service-models)
4. [AWS Core Services for Geoinformation](#aws-core-services-for-geoinformation)
   - **Compute Services**: EC2, Lambda, Containers (ECS/Fargate/EKS/Batch), ECR
   - **Storage Services**: S3, EBS, EFS
   - **Database Services**: RDS (PostGIS), DynamoDB
   - **Additional Services**: Athena, EMR, SageMaker, CloudWatch
5. [Pricing Overview](#pricing-overview)
6. [Best Practices](#best-practices)
7. [Further Reading & Resources](#further-reading--resources)

---

## Introduction to Cloud Computing

**Cloud computing** is the delivery of computing services, including servers, storage, databases, networking, software, and analytics, over the internet ("the cloud") to offer faster innovation, flexible resources, and economies of scale.

### Key Characteristics

- **On-demand self-service**: Provision resources automatically without human interaction
- **Broad network access**: Available over the network via standard mechanisms
- **Resource pooling**: Provider's resources serve multiple customers with dynamic assignment
- **Rapid elasticity**: Scale up or down quickly based on demand
- **Pay-as-you-go**: Pay only for what you use. As analogy, think of it like renting electricity. 

### Deployment Models

- **Public Cloud**: Resources owned and operated by third-party provider (AWS, Google Cloud, Azure)
- **Private Cloud**: Dedicated infrastructure for single organization (CRIB)
- **Hybrid Cloud**: Combination of public and private clouds

---

## Why Cloud for Geodata?

Geospatial data processing presents unique challenges that cloud computing addresses effectively:

1. **Scale of Data**
- Satellite imagery: Petabytes of data (Landsat archive alone exceeds 7 PB)
- LiDAR point clouds: Billions of points per survey
- Global datasets: OpenStreetMap, DEM, climate models

2. **Computational Intensity**
- Image classification and object detection
- Terrain analysis and 3D modeling
- Time-series analysis of multi-temporal imagery
- Machine learning on spatial data

3. **Data Co-location**
- Process data where it's stored (avoid large transfers)
- Public datasets already hosted on cloud (AWS Open Data, Google Earth Engine)
- Example: Landsat and Sentinel data on AWS S3

4. **Cost Efficiency**
- No upfront hardware investment
- Pay only during processing periods
- Scale to zero when not in use

5. **Collaboration**
- Share results and workflows globally
- Reproducible research environments
- Version control and data provenance

---

## Cloud Service Models

Understanding service models helps you choose the right abstraction level:

### Infrastructure as a Service (IaaS)
- **What you get**: Virtual machines, storage, networks
- **What you manage**: OS, runtime, applications, data
- **Examples**: AWS EC2, S3, VPC
- **Best for**: Custom processing pipelines, full control needed
- **Geodata use case**: Setting up custom Dask cluster (Day 2 hands-on!)

### Platform as a Service (PaaS)
- **What you get**: Managed runtime environment
- **What you manage**: Applications and data
- **Examples**: AWS Elastic Beanstalk, Lambda
- **Best for**: Focus on code, not infrastructure
- **Geodata use case**: Deploying web mapping applications

### Software as a Service (SaaS)
- **What you get**: Complete application
- **What you manage**: Just your data and settings
- **Examples**: Google Earth Engine, ArcGIS Online
- **Best for**: Immediate use without setup
- **Geodata use case**: Quick analysis of satellite imagery (you already know GEE!)

**Comparison for Geodata Processing:**

| Model | Control | Complexity | Setup Time | Example Task |
|-------|---------|------------|------------|--------------|
| SaaS | Low | Low | Minutes | Analyze NDVI in GEE |
| PaaS | Medium | Medium | Hours | Deploy tiling server |
| IaaS | High | High | Days | Custom ML pipeline on EC2 |

---

## AWS Core Services for Geoinformation

AWS services are organized into categories based on their primary function. Understanding these categories helps you choose the right service for your geodata workflows.

**Typical Cloud-Based Geodata Workflow**:
1. **Search & Discovery**: Query **STAC catalogs** to find satellite imagery from object storage (e.g., **AWS S3**, Google Cloud Storage)
2. **Ingest**: Access data directly from cloud object storage or stream subsets using **cloud-optimized formats** (COG, Zarr)
3. **Compute Environment**: Launch **Jupyter notebooks** or processing scripts on:
   - **Virtual machine clusters** (e.g., **AWS EC2**, Google Compute Engine)
   - **Container orchestration platforms** (e.g., **AWS EKS/ECS**, Kubernetes)
   - **Serverless compute** for event-driven tasks (e.g., **AWS Lambda**, Cloud Functions)
4. **Processing**: Parallel/distributed processing using frameworks like **Dask**, Spark, or Ray across compute cluster
5. **Temporary Storage**: Use **block storage** (e.g., **AWS EBS**) or shared file systems for intermediate working data
6. **Store Results**: Save processed outputs to **object storage** (e.g., **AWS S3**) in cloud-optimized formats
7. **Catalog & Share**: Index results in **spatial databases** (e.g., **AWS RDS** with PostGIS) or **NoSQL databases** (e.g., **AWS DynamoDB**), publish as STAC items

---

## Compute Services

Provide processing power to run your applications and analysis. In geodata processing, compute is where your code executes, whether that's image classification, terrain analysis, or spatial statistics.

### Virtual Machine (EC2)

Virtual servers (virtual machines) in the cloud. EC2 (Elastic Compute Cloud) provides resizable compute capacity that you can launch in minutes. Think of it as renting a computer in AWS's data center. You get full control over the operating system and can install any software you need.

**Why for geodata**:
- **Full control**: Install any geospatial software (GDAL, QGIS, Python libraries, custom tools)
- **Scale compute power**: From tiny instances to machines with 96+ vCPUs and terabytes of RAM
- **Choose GPU instances**: Accelerate deep learning model training for satellite image classification
- **Parallel processing**: Launch hundreds of instances to process large datasets simultaneously
- **Flexibility**: Start with small instance, scale up when needed, scale down when done
- **Pre-configured environments**: Use AMIs with geo-software already installed (like those from OSGeo)

**Key Concepts**:

- **Instance types**: Optimized for different workloads (over 400+ combinations!)
  - **General purpose**: `t3.medium`, `m5.large` - Balanced CPU/memory for general workloads
  - **Compute optimized**: `c5.xlarge`, `c6i.4xlarge` - High CPU for intensive processing (orthorectification, feature extraction)
  - **Memory optimized**: `r5.2xlarge`, `r6i.8xlarge` - Large RAM for in-memory raster operations, large point clouds
  - **GPU instances**: `p3.2xlarge`, `g4dn.xlarge` - Deep learning, CNN-based classification, GPU-accelerated processing
  - **Storage optimized**: `i3.xlarge` - High-speed local NVMe storage for temporary processing
  
- **Instance lifecycle**:
  - **Launch**: Start a new instance from AMI (takes 1-2 minutes)
  - **Running**: You pay per second while instance runs
  - **Stop**: Halt instance, EBS data persists, no compute charges (only storage)
  - **Terminate**: Delete instance permanently, EBS volumes deleted (unless configured otherwise)
  - **Important**: Stopped instances still incur EBS storage costs!

- **AMI (Amazon Machine Image)**: Pre-configured OS template
  - Public AMIs: Ubuntu, Amazon Linux, Windows Server
  - Community AMIs: Pre-installed geo-software (search: "gis", "gdal", "qgis")
  - Custom AMIs: Create your own with exact software configuration
  - Marketplace AMIs: Commercial software (ArcGIS, ENVI)
  
- **Security Groups**: Virtual firewall rules for instances
  - Control inbound/outbound traffic
  - Example: Allow SSH (port 22) only from your IP
  - Allow HTTP/HTTPS for web services
  - Block all other traffic by default
  
- **Key Pairs**: SSH access credentials for Linux instances
  - Public key stored on EC2
  - Private key (.pem file) stays with you - NEVER share this!
  - Used to SSH into instance: `ssh -i mykey.pem ec2-user@<instance-ip>`
  
- **Elastic IP**: Static public IP address
  - Normal EC2 IPs change on stop/start
  - Elastic IP remains constant (useful for servers)
  - Free while attached to running instance
  - Cost: $0.005/hour when NOT attached (to discourage waste)

- **Instance Store**: Temporary local storage
  - Very fast NVMe SSD directly attached
  - Data lost on stop/terminate (ephemeral)
  - Free (included in instance price)
  - Use for: Temporary processing cache, swap space

- **Placement Groups**: Control physical placement of instances
  - **Cluster**: Low-latency network between instances (same rack)
  - **Spread**: Instances on different hardware (high availability)
  - **Partition**: Groups of instances isolated from failures

**Example of Instance Selection for Geodata Tasks**:

| Task | Recommended Instance | Reasoning |
|------|---------------------|-----------|
| GDAL raster processing | `c5.4xlarge` (16 vCPU) | CPU-intensive, benefits from multiple cores |
| Large point cloud processing | `r5.4xlarge` (128 GB RAM) | Needs lots of memory to load data |
| Deep learning training | `p3.2xlarge` (1 GPU) | GPU acceleration essential |
| Deep learning inference | `g4dn.xlarge` (1 GPU) | Cheaper GPU option for prediction |
| Batch imagery preprocessing | `c5.xlarge` + Spot | CPU tasks, interruptible OK |
| PostGIS database | `r5.xlarge` (32 GB RAM) | Memory for database caching |
| Development/testing | `t3.medium` (2 vCPU) | Burstable, cheap for light workloads |

**Pricing** (US East region, On-Demand):
- `t3.medium` (2 vCPU, 4 GB): $0.0416/hour (~$30/month if running 24/7)
- `c5.xlarge` (4 vCPU, 8 GB): $0.17/hour (~$124/month)
- `c5.4xlarge` (16 vCPU, 32 GB): $0.68/hour (~$496/month)
- `r5.2xlarge` (8 vCPU, 64 GB): $0.504/hour (~$368/month)
- `p3.2xlarge` (8 vCPU, 61 GB, 1 V100 GPU): $3.06/hour (~$2,234/month)
- **Spot Instances**: Same hardware at 50-90% discount (prices fluctuate)
    - Up to 90% discount compared to On-Demand
    - AWS can interrupt with 2-minute warning
    - Best for: Fault-tolerant, flexible tasks (batch processing)
    - Not for: Critical services, databases
    - **Geodata perfect fit**: Processing 1000 scenes? Use Spot! If interrupted, just resume.

**Cost Optimization**:
- **Stop instances when not in use**: Zero compute charges!
- **Reserved Instances**: 1-year commitment = 30-40% savings, 3-year = 60% savings
- **Savings Plans**: Flexible commitment to compute usage
- **Right-sizing**: Monitor actual usage, downsize if underutilized

**Best Practices for EC2 in Geodata Workflows**:

1. **Start small, scale up**: Begin with `t3.medium` for testing, then use larger instances for production
2. **Use Auto Scaling**: Automatically adjust number of instances based on workload
3. **Tag everything**: Organize instances by project, purpose, cost center
4. **Set termination protection**: Prevent accidental deletion of critical instances
5. **Enable CloudWatch monitoring**: Track CPU, memory, disk usage
6. **Create AMI snapshots**: Save configured environments for future use
7. **Use EBS snapshots**: Regular backups before risky operations
8. **Automate shutdown**: Use Lambda or scripts to stop instances after processing
9. **Data gravity**: Keep compute near data (same region as S3) to avoid transfer costs
10. **Test with Spot**: Most geodata tasks are fault-tolerant—save money with Spot!

---

### Container Services

Lightweight, portable packages containing application code and all dependencies. Think of them as "lighter" virtual machines that start in seconds instead of minutes. Imagine you have a Python script that processes satellite imagery using rasterio 1.3.5 and numpy 1.24.0. On your laptop it works perfectly. But on a colleague's machine or on EC2, it might fail because they have different versions or missing libraries. Containers solve this by **packaging your code and its entire environment into a single, portable unit**. Docker is the most popular container platform.

**Container vs Virtual Machine**:
- **VM**: Includes full OS (gigabytes), boots in minutes, heavier overhead
- **Container**: Shares host OS kernel, includes only app + libraries (megabytes), starts in seconds, lighter overhead
- **Result**: Run 100 containers on hardware that might only support 10 VMs

**Basic terminology**:
- **Image**: Read-only template (like an AMI for containers) containing your application and dependencies. Example: `osgeo/gdal:ubuntu-full-3.6.2` (official GDAL image)
- **Container**: Running instance of an image (like an EC2 instance is a running AMI)
- **Dockerfile**: Recipe to build an image (text file with instructions)
- **Registry**: Repository for storing and sharing images (Docker Hub, AWS ECR)
- **Orchestration**: Managing multiple containers across multiple hosts

**Why containers for geodata**:
- **Reproducibility**: Package your exact environment (GDAL 3.6.2, Python 3.11, specific libraries). You can publish alongside research paper for reproducibility.
- **Portability**: Same container runs on your laptop, AWS, Google Cloud, on-premises cluster, anywhere.
- **Consistency**: Everyone uses identical environment. Development, testing, production use same container.
- **Efficiency**: Better resource utilization than full VMs. Start 100 processing tasks in seconds.
- **Isolation**: Each container runs independently. Multiple Python versions on same host.
- **Version control**: Image tags track versions. Tag: `my-processing:v1.0`, `my-processing:v1.1`.

---

#### ECS (Elastic Container Service)

AWS's native container orchestration service—manages running many containers across multiple EC2 instances. A smart scheduler that decides which EC2 instance should run which container, monitors health, restarts failed containers, and scales automatically.

**Key Concepts**:
- **Task Definition**: Blueprint for your application (which image, CPU, memory)
- **Service**: Maintains desired number of running tasks
- **Cluster**: Logical grouping of EC2 instances or Fargate capacity
- **Launch Types**: EC2 (you manage instances) or Fargate (serverless)

**Pricing**:
- **EC2 Launch Type**: 
  - No additional ECS charge (ECS itself is free!)
  - Pay only for underlying EC2 instances
  - Example: c5.2xlarge at $0.34/hour, run 10 containers on it
- **Fargate Launch Type**: Pay per task resources
  - vCPU: $0.04048/hour
  - Memory: $0.004445/GB-hour
  - Example: 2 vCPU, 4 GB for 1 hour = $0.08 + (4 × $0.004445) = $0.098

**Cost Comparison Example** (100 tasks, 1 hour each):
- **Fargate**: 100 tasks × 2 vCPU × 1 hr × $0.04048 = $8.10
- **EC2 (c5.2xlarge)**: Run ~8 tasks concurrently = 12.5 hours × $0.34 = $4.25
- **Winner**: EC2 cheaper for sustained workloads, Fargate better for sporadic

**Best Practices for ECS**:
1. **Use task IAM roles**: Don't embed AWS credentials in images
2. **Enable container insights**: Monitor CPU, memory per container
3. **Use CloudWatch Logs**: Centralized logging from all containers
4. **Auto-scaling policies**: Scale based on metrics (CPU, memory, SQS queue)
5. **Health checks**: Let ECS restart unhealthy containers
6. **Spot instances for EC2**: Save 50-90% on fault-tolerant workloads
7. **Task placement strategies**: Spread across AZs for reliability
8. **Use secrets manager**: For database passwords, API keys

---

#### Fargate (Serverless Containers)
Run containers without managing servers—you just define the container, AWS handles everything else. Lambda for containers. No EC2 instances to manage, no cluster capacity planning, no patching. Just run your container.

**Fargate Task Sizes**:
- CPU: 0.25 to 4 vCPU
- Memory: 0.5 GB to 30 GB (certain combinations only)
- Example valid sizes:
  - 0.25 vCPU: 0.5 GB, 1 GB, 2 GB
  - 1 vCPU: 2 GB, 3 GB, 4 GB, ... 8 GB
  - 2 vCPU: 4 GB, 5 GB, ... 16 GB
  - 4 vCPU: 8 GB, 9 GB, ... 30 GB

**Pricing** (Linux, US East):
- vCPU: $0.04048/hour
- Memory: $0.004445/GB-hour
- **No charge when not running!**

---

#### EKS (Elastic Kubernetes Service)

Managed Kubernetes service. AWS runs the Kubernetes control plane, you run workloads. Kubernetes is open-source container orchestration platform (like ECS but industry-standard, more complex, more powerful)

**Kubernetes Basics**:
- **Pod**: Group of one or more containers (like ECS task)
- **Deployment**: Manages replicas of pods
- **Service**: Network endpoint to access pods
- **Node**: Worker machine (EC2 instance or Fargate)
- **Namespace**: Logical isolation within cluster

**When to choose EKS over ECS**:
- Need Kubernetes-specific features
- Want portability (same config on AWS, GCP, on-prem)
- Team has Kubernetes expertise
- Using ecosystem tools (Dask-kubernetes, JupyterHub)
- Complex microservices architecture

**Pricing**:
- **Control plane**: $0.10/hour = **$73/month** (per cluster)
- **Worker nodes**: 
  - EC2: Pay for instances
  - Fargate: Same Fargate pricing as ECS

---

### Serverless Compute (Lambda)

Run code without managing servers. You just upload your function and cloud provider handles everything. Lambda executes your code only when triggered (events like S3 uploads, API calls, schedules), automatically scales from zero to thousands of executions, and charges only for actual compute time (measured in milliseconds). When idle, you pay nothing. Think of it as "functions as a service" for event-driven, short-duration tasks.

**Why for geodata**:
- Event-driven processing (new file uploaded → trigger processing)
- Auto-scaling from zero
- Pay only for execution time (measured in milliseconds!)
- Perfect for lightweight, quick tasks

**Key Concepts**:
- **Function**: Your code (Python, Node.js, etc.)
- **Trigger**: S3 upload, API call, schedule
- **Runtime limit**: Max 15 minutes execution
- **Memory**: 128 MB to 10 GB

**Pricing**:
- Free tier: 1M requests/month + 400,000 GB-seconds compute
- After: $0.20 per 1M requests
- Compute: $0.0000166667/GB-second

### Which Compute Service Should I Use?

Choose based on your task characteristics:

| Your Need | Choose This | Why |
|-----------|-------------|-----|
| Full control, custom software, GPU | **EC2** | Complete flexibility |
| Quick task < 15 min, event-driven | **Lambda** | Simplest, cheapest for short tasks |
| Long-running task, want serverless | **Fargate** | No server management |
| Reproducible environment | **Containers (ECS/Fargate)** | Package everything together |
| Thousands of batch jobs | **AWS Batch** | Handles complexity for you |
| Already use Kubernetes | **EKS** | Stick with what you know |
| Need portability across clouds | **Containers (ECS/EKS)** | Industry standard |

**Decision Tree for Geodata Processing**:

Decision flow: If you need GPU or very specific hardware, use EC2. Otherwise, check if task takes < 15 minutes - if yes and event-driven use Lambda, if not event-driven and need container use Fargate, otherwise Lambda or EC2. For tasks > 15 min, if you have thousands of similar jobs use AWS Batch, otherwise check if you need reproducibility - if yes use Containers (ECS/Fargate), if no use EC2 or Fargate.

**Cost Comparison Example**: Processing 100 Landsat scenes (30 min each)

| Service | Setup | Cost | Notes |
|---------|-------|------|-------|
| EC2 (c5.xlarge) | Manual | ~$85 | 50 hours × $0.17/hr |
| EC2 Spot | Manual | ~$25 | 50 hours × ~$0.05/hr (70% discount) |
| Fargate | Managed | ~$100 | Per vCPU/memory pricing |
| AWS Batch + Spot | Automated | ~$30 | Auto-scheduling + spot pricing |
| Lambda | N/A | Not suitable | 15 min limit per function |

---

## Storage Services

Store your data like raw imagery, processed results, metadata, and intermediate files. In geodata workflows, you need different types of storage: object storage for imagery, block storage for working datasets, and archives for long-term preservation.

### S3 (Simple Storage Service)

Object storage for any type of data. Unlike traditional file systems (with hierarchical folders) or block storage (like hard drives), S3 stores data as **objects** in a flat namespace. Each object consists of the data itself, metadata (descriptive information), and a unique identifier (key). User retrieve object directly as if you are downloading data from a website (without the needs to use VM).

**Why for geodata**:
- **Unlimited capacity**: Store petabytes of imagery
- **Durability**: 99.999999999% (11 nines) - virtually never lose data
- **Integration**: Works seamlessly with all AWS services
- **Public datasets**: Landsat, Sentinel-2, NAIP already on S3

**Key Concepts**:
- **Buckets**: Container for objects (globally unique name)
- **Objects**: Individual files with metadata (images, vectors, documents)
- **S3 paths**: `s3://bucket-name/path/to/file.tif`
- **Storage classes**: Different cost/performance tiers

**Storage Classes** (per GB/month):
- **S3 Standard**: $0.023 (frequent access, millisecond latency)
- **S3 Intelligent-Tiering**: $0.023 + monitoring (automatic optimization)
- **S3 Standard-IA**: $0.0125 (infrequent access, retrieval fee)
- **S3 Glacier Instant Retrieval**: $0.004 (archive, millisecond access)
- **S3 Glacier Flexible Retrieval**: $0.0036 (archive, minutes-hours retrieval)
- **S3 Glacier Deep Archive**: $0.00099 (long-term archive, 12 hours retrieval)

**Transfer Costs**:
- Upload to S3: **Free**
- Download from S3: First 100 GB/month free, then $0.09/GB
- Same-region transfer (S3 to EC2): **Free** (important!)

**Geodata Usage Example**:

Using libraries like rasterio, you can read directly from S3 without downloading (no local storage needed), and write processed results back to S3 seamlessly. S3 also stores STAC catalogs and assets.

---

### EBS (Elastic Block Store)

Block storage volumes that attach to EC2 instances (like an external hard drive)

**Why for geodata**:
- **Persistence**: Data survives instance stop/start
- **Performance**: High-speed SSD options
- **Snapshots**: Point-in-time backups
- **Flexibility**: Attach, detach, resize as needed

**Volume Types**:
- **gp3 (General Purpose SSD)**: $0.08/GB-month, 3000 IOPS baseline
  - Best for: Most workloads, good balance
- **io2 (Provisioned IOPS SSD)**: $0.125/GB-month + IOPS cost
  - Best for: Database workloads, highest performance
- **st1 (Throughput Optimized HDD)**: $0.045/GB-month
  - Best for: Large sequential reads (big raster files)
- **sc1 (Cold HDD)**: $0.015/GB-month
  - Best for: Infrequent access, archival

**Geodata Usage Example**:

Typical workflow: Attach a 500 GB gp3 volume to EC2 instance, download data from S3 to EBS for fast local access, process on EBS with high IOPS, upload results to S3 for permanent storage, then delete EBS volume to save costs.

---

### EFS (Elastic File System)

**What it is**: Managed network file system (NFS) that can be mounted by multiple EC2 instances

**Why for geodata**:
- **Shared storage**: Multiple instances access same files
- **Auto-scaling**: Grows and shrinks automatically
- **No capacity planning**: Pay only for what you use

**Pricing**:
- Standard: $0.30/GB-month
- Infrequent Access: $0.025/GB-month

**Geodata Usage Example**:

EFS can be mounted on multiple EC2 instances, allowing all instances to read/write shared datasets. This is ideal for shared reference data or collaborative processing.

---

### Which Storage Service Should I Use?

Choose based on your data access patterns and requirements:

| Your Need | Choose This | Why |
|-----------|-------------|-----|
| Long-term storage of imagery/results | **S3 Standard** | Durable, accessible, cost-effective |
| Archive data (rare access) | **S3 Glacier** | Very cheap ($0.00099/GB) |
| Working data for EC2 processing | **EBS** | Fast local access |
| Shared data across multiple EC2 | **EFS** | Network file system |
| Data accessed < 1/month | **S3 Intelligent-Tiering** | Auto-optimizes costs |
| Intermediate processing files | **EBS** then delete | Fast, then clean up |

**Storage Decision Tree for Geodata**:

Decision flow: If data is needed long-term, consider access frequency - frequently (>1/month) use S3 Standard, sometimes (1/month-1/year) use S3 Intelligent-Tiering, rarely (<1/year) use S3 Glacier Deep Archive. If not long-term, check if it's temporary processing data - if yes and need high IOPS use EBS SSD (gp3 or io2), if don't need high IOPS use EBS HDD (st1). If not temporary, check if need shared across instances - if yes use EFS, if no use EBS.

**Cost Optimization Strategy**:

1. **Active processing**: S3 → EBS → Process → S3
2. **Long-term**: Use S3 Lifecycle Policies
   - 0-30 days: S3 Standard
   - 30-90 days: S3 Intelligent-Tiering
   - 90+ days: S3 Glacier
3. **Cleanup**: Delete EBS volumes when not in use!

**Example Cost Comparison** (1 TB for 1 year):

| Service | Access Pattern | Annual Cost |
|---------|---------------|-------------|
| S3 Standard | Frequent | $276 |
| S3 Intelligent-Tiering | Mixed | $276 + monitoring |
| S3 Glacier | Rare | $48 |
| S3 Deep Archive | Archive | $11.88 |
| EBS gp3 | Always attached | $960 |

---

## Database Services

Store and query structured data—vector geometries, metadata, attributes, and relationships. Unlike file storage (S3/EBS), databases provide indexing, transactions, and complex queries.

### RDS (Relational Database Service)

Managed relational databases (PostgreSQL, MySQL, etc.) without server management

**Why for geodata**:
- **PostGIS support**: Full spatial database capabilities
- **Managed**: Automatic backups, updates, monitoring
- **Scalable**: Read replicas, vertical scaling
- **High availability**: Multi-AZ deployment option

**Key Features**:
- **PostGIS**: PostgreSQL extension for spatial operations
- **Automated backups**: Point-in-time recovery (35 days)
- **Multi-AZ deployment**: Automatic failover for high availability
- **Read replicas**: Scale read operations across regions

**Database Engines**:
- **PostgreSQL with PostGIS**: Best for geodata (spatial queries, topology)
- **MySQL**: Alternative option
- **Amazon Aurora PostgreSQL**: PostgreSQL-compatible, 3x faster, auto-scaling

**Pricing** (PostgreSQL, US East):
- `db.t3.micro` (1 vCPU, 1 GB): $0.018/hour (~$13/month)
- `db.t3.medium` (2 vCPU, 4 GB): $0.073/hour (~$53/month)
- `db.r5.xlarge` (4 vCPU, 32 GB): $0.36/hour (~$262/month)
- Storage: $0.115/GB-month (SSD)
- Backup storage: First 100% of DB storage is free

**Geodata Example**:

With PostGIS on RDS, you can store vector data in spatial tables, create spatial indexes for fast queries, perform spatial operations like intersections and buffers, and join tables based on spatial relationships like containment.

---

### DynamoDB (NoSQL Database)

Fully managed NoSQL database for key-value and document data

**Why for geodata**:
- **Serverless**: No server management
- **Performance**: Single-digit millisecond latency at any scale
- **Flexible schema**: Store varying metadata structures
- **Auto-scaling**: Handles traffic spikes

**Best for geodata**:
- STAC metadata storage
- Tile index/catalog
- Processing job status tracking
- Not for: Complex spatial queries (use RDS/PostGIS for that)

**Pricing**:
- On-demand: $1.25/million writes, $0.25/million reads
- Storage: $0.25/GB-month
- Free tier: 25 GB storage, 25 WCU, 25 RCU

**Geodata Example**:

Using boto3, you can store STAC item metadata with attributes like collection, datetime, bounding box, and assets, then efficiently query by collection or other indexed fields.

---

### Which Database Service Should I Use?

Choose based on your data structure and query needs:

| Your Need | Choose This | Why |
|-----------|-------------|-----|
| Vector data with spatial queries | **RDS (PostGIS)** | Full spatial SQL support |
| Complex relationships, transactions | **RDS** | ACID compliance, joins |
| Simple metadata, high throughput | **DynamoDB** | Serverless, scalable |
| STAC catalog | **DynamoDB or RDS** | Both work, DynamoDB simpler |
| Attributes + geometry analysis | **RDS (PostGIS)** | ST_* functions |
| Need GeoJSON queries | **RDS (PostGIS)** | Native support |

**Database Decision Tree**:

Decision flow: If you need spatial operations (buffers, intersections, etc.), use RDS with PostGIS. If not, check if your data is structured (fixed schema) - if yes and need transactions/joins use RDS (PostgreSQL/MySQL), if yes but don't need transactions use DynamoDB (simpler, cheaper). If data has flexible schema, use DynamoDB.

**When to combine both**:
- **RDS**: Vector data, complex spatial queries
- **DynamoDB**: Metadata catalog, job tracking
- **S3**: Raster imagery, large files

**Example Architecture**:

For satellite imagery storage, use S3 for raster data as Cloud-Optimized GeoTIFF, store metadata in DynamoDB (STAC items), and store vector results in RDS PostGIS (polygons, analysis).

---

## Additional Services for Geodata

#### Amazon SageMaker
- Machine learning platform
- Built-in algorithms, Jupyter notebooks
- Model training and deployment
- Good for: Land use classification, object detection in imagery

#### Amazon EMR (Elastic MapReduce)
- Managed Hadoop/Spark clusters
- Process vast amounts of data
- Good for: Large-scale raster analysis, global land cover classification

#### CloudWatch
- Monitoring and logging
- Track EC2 performance, set alarms
- Essential for: Cost monitoring, debugging

---

## Pricing Overview

### Cost Structure

AWS pricing has three main components:

1. **Compute**: Time your instances run
2. **Storage**: Amount of data stored
3. **Transfer**: Data moved out of AWS

### Key Pricing Principles

- **Pay-as-you-go**: No upfront costs
- **Save with commitment**: Reserved instances (1-3 year term) save 30-75%
- **Scale down to save**: Stop instances when not needed
- **Free tier**: 12 months free access to many services

### Free Tier Highlights (First 12 Months)

- **EC2**: 750 hours/month of `t2.micro` or `t3.micro`
- **S3**: 5 GB standard storage
- **RDS**: 750 hours/month of `db.t2.micro`, 20 GB storage
- **Lambda**: 1M requests/month, 400,000 GB-seconds
- **Data transfer**: 15 GB/month out to internet

### Example Scenario: Processing 1000 Landsat Scenes

**Option 1: Single EC2 Instance**
- Instance: `c5.4xlarge` (16 vCPU, 32 GB)
- Duration: 100 hours
- Cost: 100 × $0.68 = $68
- Storage: 500 GB EBS = $40/month (for 1 month ≈ $40)
- **Total**: ~$108

**Option 2: 10 Spot Instances (Parallel)**
- Instance: `c5.4xlarge` spot price ~$0.20/hour
- Duration: 10 hours (10x faster!)
- Cost: 10 instances × 10 hours × $0.20 = $20
- **Total**: ~$60 + storage

**Cost Saving Tips**:
1. Use spot instances (up to 90% discount)
2. Stop instances when not in use
3. Use S3 Intelligent-Tiering for storage
4. Set billing alarms in CloudWatch
5. Delete snapshots you no longer need

### Pricing Calculator

**AWS Pricing Calculator**: https://calculator.aws/
- Estimate your monthly costs
- Compare different configurations
- Plan your budget before starting

---

## Best Practices

### 1. Security

- **Never share AWS credentials**: Use IAM roles for EC2
- **Use security groups**: Limit SSH access to your IP
- **Enable MFA**: Multi-factor authentication for AWS console
- **Encrypt data**: S3 encryption, EBS encryption

### 2. Cost Management

- **Set billing alarms**: Get alerts before overspending
- **Tag resources**: Track costs by project
- **Use auto-shutdown**: Stop instances after processing
- **Monitor usage**: CloudWatch dashboards

### 3. Data Management

- **Lifecycle policies**: Auto-move S3 data to cheaper storage
- **Versioning**: Enable for critical datasets
- **Backup strategy**: Regular EBS snapshots
- **Data locality**: Keep data and compute in same region

### 4. Performance

- **Choose right instance type**: CPU vs memory vs GPU
- **Use placement groups**: Low-latency inter-instance communication
- **Optimize storage**: SSD for random access, HDD for sequential
- **Parallel processing**: Multiple smaller instances often better than one large

### 5. For Geodata Specifically

- **Use Cloud-Optimized GeoTIFF (COG)**: Efficient partial reads from S3
- **STAC for metadata**: Standard for organizing geospatial assets
- **Tiles over full images**: Process spatial subsets
- **Compress data**: Reduce storage and transfer costs

---

## Further Reading & Resources

### Official AWS Documentation

1. **AWS Getting Started**: https://aws.amazon.com/getting-started/
   - Free tutorials and hands-on labs
   
2. **AWS EC2 User Guide**: https://docs.aws.amazon.com/ec2/
   - Complete reference for EC2 instances
   
3. **AWS S3 Documentation**: https://docs.aws.amazon.com/s3/
   - Object storage best practices

4. **AWS Pricing**: https://aws.amazon.com/pricing/
   - Detailed pricing for all services

5. **AWS Free Tier**: https://aws.amazon.com/free/
   - What's included in free tier

### Geospatial on AWS

1. **AWS Earth on AWS**: https://aws.amazon.com/earth/
   - Geospatial datasets and case studies
   
2. **Registry of Open Data on AWS**: https://registry.opendata.aws/
   - Free datasets: Landsat, Sentinel, NAIP, and more
   - Tag: "earth observation"

3. **Landsat on AWS**: https://registry.opendata.aws/landsat-8/
   - Direct S3 access to entire Landsat archive

4. **Sentinel-2 on AWS**: https://registry.opendata.aws/sentinel-2/
   - Cloud-optimized GeoTIFFs on S3

### Containers and Docker

1. **Docker Documentation**: https://docs.docker.com/
   - Official Docker guide and tutorials
   
2. **Docker Hub - Geospatial Images**: https://hub.docker.com/
   - Search: "gdal", "postgis", "qgis"
   - OSGeo official images

3. **AWS ECS Documentation**: https://docs.aws.amazon.com/ecs/
   - Complete guide to ECS and Fargate

4. **AWS Batch User Guide**: https://docs.aws.amazon.com/batch/
   - Batch processing best practices

5. **Kubernetes Documentation**: https://kubernetes.io/docs/
   - For EKS users

6. **Dask on Kubernetes**: https://kubernetes.dask.org/
   - Deploy Dask clusters on EKS

### Cloud-Optimized Formats

1. **Cloud-Optimized GeoTIFF (COG)**: https://www.cogeo.org/
   - Why and how to create COGs
   
2. **STAC Specification**: https://stacspec.org/
   - SpatioTemporal Asset Catalogs

### Tutorials and Courses

1. **AWS Training and Certification**: https://aws.amazon.com/training/
   - Free digital courses
   - Cloud Practitioner Essentials (recommended)

2. **Awesome AWS**: https://github.com/donnemartin/awesome-aws
   - Curated list of AWS libraries, tools, and guides

3. **Dask Documentation**: https://docs.dask.org/
   - Parallel computing with Dask (relevant for Day 2!)

4. **Pangeo Project**: https://pangeo.io/
   - Cloud-native geoscience workflows
   - Examples with Dask on cloud

### Community and Support

1. **AWS re:Post**: https://repost.aws/
   - Community Q&A forum
   
2. **GIS Stack Exchange**: https://gis.stackexchange.com/
   - Tag your questions with 'amazon-web-services'

3. **r/aws** (Reddit): https://reddit.com/r/aws
   - Active community discussions

---

## Questions?

Come to class with questions! Here are some thought starters:

1. When would you choose cloud over local processing?
2. How does AWS compare to Google Cloud (where GEE runs)?
3. What's the most cost-effective way to process your research data?
4. How can you ensure reproducibility when using cloud?

See you in class!

---
  
*Last updated: January 2026*