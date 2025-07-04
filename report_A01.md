## Technical Architecture

---

### System Overview

<details>
<summary>Comprehensive description of the AWS Data Platform foundation</summary>

---


* **Purpose**: <em>Describe the high‑level objectives of the data platform (analytics, ML, BI, etc.).</em>
* **High‑level Infrastructure Diagram**: Diagram summarizing core services.
  
  ```mermaid
  graph TB
    subgraph "🌐 User Connectivity"
        Users[👥 Business Users<br/>Data Scientists, Analysts, Engineers]
        AdminAccess[🔐 Secure Access<br/>AWS Session Manager<br/>No VPN Required]
    end
    
    subgraph "☁️ AWS Cloud Infrastructure"
        subgraph "🏢 Primary Datacenter (Singapore)"
            subgraph "🔒 Private Secure Zone (Private Network)"
                subgraph "📊 Data Processing Layer"
                    NLB[⚖️ Load Balancer<br/>Network Load Balancer<br/>📈 Auto-distribute traffic]
                    
                    subgraph "🏗️ Server Cluster Zone A"
                        Workers1[💻 Data Processing Servers<br/>Auto Scaling Group: 2-10 instances<br/>🚀 Auto-scale based on demand]
                        Auth1[🔐 Authentication Server A<br/>FreeIPA Master<br/>👤 Employee account management]
                    end
                    
                    subgraph "🏗️ Server Cluster Zone B"
                        Workers2[💻 Data Processing Servers<br/>Auto Scaling Group: 2-10 instances<br/>🚀 Auto-scale based on demand]
                        Auth2[🔐 Authentication Server B<br/>FreeIPA Replica<br/>🛡️ Authentication system backup]
                    end
                end
                
                subgraph "💾 Data Storage Layer"
                    EFS[📁 Shared File System<br/>Amazon EFS<br/>💰 Pay-as-you-use<br/>🔄 Auto-scale from GB to PB]
                end
            end
            
            subgraph "🛡️ Security & Operations Services"
                SecMgr[🔑 Password Management<br/>AWS Secrets Manager<br/>🔐 Secure credential storage]
                Monitoring[📈 System Monitoring<br/>CloudWatch<br/>📊 Dashboards & Alerts]
                Backup[💿 Automated Backup<br/>AWS Backup<br/>📅 Daily scheduled backups]
            end
        end
    end
    
    %% User Flow
    Users --> AdminAccess
    AdminAccess --> NLB
    
    %% Load Distribution
    NLB --> Workers1
    NLB --> Workers2
    NLB --> Auth1
    NLB --> Auth2
    
    %% Authentication Flow
    Workers1 --> Auth1
    Workers2 --> Auth2
    Auth1 -.->|🔄 Sync| Auth2
    
    %% Data Access
    Workers1 --> EFS
    Workers2 --> EFS
    
    %% Security & Operations
    Workers1 --> SecMgr
    Workers2 --> SecMgr
    Workers1 --> Monitoring
    Workers2 --> Monitoring
    EFS --> Backup
    
    %% Styling
    classDef userClass fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef computeClass fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef storageClass fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef securityClass fill:#fff3e0,stroke:#e65100,stroke-width:2px
    
    class Users,AdminAccess userClass
    class NLB,Workers1,Workers2,Auth1,Auth2 computeClass
    class EFS storageClass
    class SecMgr,Monitoring,Backup securityClass
  ```
* **High‑level Technical-Detailed Diagram**: Diagram for technical detail design
  ```mermaid
    graph TB
    subgraph "AWS Region: ap-southeast-1"
        subgraph "VPC: 10.0.0.0/16"
            subgraph "Public Subnets"
                subgraph "AZ-1a: 10.0.1.0/24"
                    NLB-1a[Network LB<br/>Port: 80,443,22,389]
                    NAT-1a[NAT Gateway<br/>For outbound traffic]
                end
                subgraph "AZ-1b: 10.0.2.0/24" 
                    NLB-1b[Network LB Target<br/>Cross-AZ redundancy]
                    NAT-1b[NAT Gateway<br/>Backup outbound]
                end
            end
            
            subgraph "Private App Subnets"
                subgraph "AZ-1a: 10.0.10.0/24"
                    ASG-1a[Auto Scaling Group<br/>Min:2, Max:10, Desired:4]
                    EC2-1a[m5.xlarge instances<br/>4vCPU, 16GB RAM<br/>SG: app-servers-sg]
                    FreeIPA-1a[c5.large instance<br/>2vCPU, 4GB RAM<br/>SG: freeipa-master-sg<br/>Ports: 53,88,389,636,464]
                end
                subgraph "AZ-1b: 10.0.20.0/24"
                    ASG-1b[Auto Scaling Group<br/>Min:2, Max:10, Desired:4]
                    EC2-1b[m5.xlarge instances<br/>4vCPU, 16GB RAM<br/>SG: app-servers-sg]
                    FreeIPA-1b[c5.large instance<br/>2vCPU, 4GB RAM<br/>SG: freeipa-replica-sg<br/>Ports: 53,88,389,636]
                end
            end
            
            subgraph "Private Data Subnets"
                subgraph "AZ-1a: 10.0.30.0/24"
                    EFS-MT-1a[EFS Mount Target<br/>SG: efs-sg<br/>Port: 2049/TCP]
                end
                subgraph "AZ-1b: 10.0.40.0/24"
                    EFS-MT-1b[EFS Mount Target<br/>SG: efs-sg<br/>Port: 2049/TCP]
                end
                EFS[Amazon EFS<br/>Encryption: AES-256<br/>Performance: Provisioned<br/>Throughput: 500 MiB/s]
            end
            
            subgraph "VPC Endpoints Subnet: 10.0.50.0/24"
                VPC-EP-SSM[SSM VPC Endpoint<br/>com.amazonaws.region.ssm]
                VPC-EP-S3[S3 Gateway Endpoint<br/>com.amazonaws.region.s3]
                VPC-EP-SM[Secrets Manager Endpoint<br/>com.amazonaws.region.secretsmanager]
            end
        end
    end
    
    subgraph "AWS Managed Services"
        SM[AWS Secrets Manager<br/>KMS Encrypted<br/>Auto-rotation: 30 days]
        CW[CloudWatch<br/>Metrics + Logs + Alarms<br/>Retention: 90 days]
        BACKUP[AWS Backup<br/>Daily: 7 days retention<br/>Weekly: 4 weeks retention]
        SSM[Systems Manager<br/>Session Manager<br/>Patch Manager]
    end
    
    subgraph "External"
        Users[👥 Users<br/>SSH via Session Manager]
        Internet[🌐 Internet]
    end
    
    %% Network Flow - Detailed
    Users -->|HTTPS/443| NLB-1a
    NLB-1a -->|TCP/22| EC2-1a
    NLB-1a -->|TCP/389,636| FreeIPA-1a
    NLB-1a -.->|Health Check| NLB-1b
    
    %% Cross-AZ Replication
    EC2-1a -->|NFS/2049| EFS-MT-1a
    EC2-1b -->|NFS/2049| EFS-MT-1b
    EFS --> EFS-MT-1a
    EFS --> EFS-MT-1b
    
    %% Authentication Flow
    EC2-1a -->|LDAP/389| FreeIPA-1a
    EC2-1b -->|LDAP/389| FreeIPA-1b
    FreeIPA-1a -.->|Replication/389| FreeIPA-1b
    
    %% Auto Scaling
    ASG-1a --> EC2-1a
    ASG-1b --> EC2-1b
    
    %% Security & Monitoring
    EC2-1a -->|HTTPS/443| VPC-EP-SSM
    EC2-1b -->|HTTPS/443| VPC-EP-SSM
    FreeIPA-1a -->|HTTPS/443| VPC-EP-SM
    VPC-EP-SSM --> SSM
    VPC-EP-SM --> SM
    
    %% Outbound Internet
    EC2-1a --> NAT-1a
    EC2-1b --> NAT-1b
    NAT-1a --> Internet
    NAT-1b --> Internet
    
    %% Monitoring & Backup
    EC2-1a -->|CloudWatch Agent| CW
    EC2-1b -->|CloudWatch Agent| CW
    EFS --> BACKUP
    FreeIPA-1a --> BACKUP
  ```
* **Core AWS services**: EC2, EFS/NFS, IAM, FreeIPA, VPC, Security Groups, CloudWatch.
* **Data flow outline**: Bullet list of ingestion → processing → storage → consumption paths.
* **Scalability & HA strategy**: Auto Scaling groups, Multi‑AZ design, fault‑tolerant components.

---

</details>

### Business Perspective
<details>
<summary>High-level value and risk view for stakeholders</summary>

---

- **Foundation for data-driven growth**
  - Central platform enables rapid delivery of analytics, ML, and reporting use-cases without each project building bespoke infrastructure.
- **Cost-aligned elasticity**
  - Auto-scaling workloads and pay-per-use storage (`EFS`, `EC2 ASG`) avoid sunk hardware costs while scaling up for peak campaigns.
- **Security & compliance by design**
  - Private subnets, IAM-least-privilege, FreeIPA SSO, and KMS encryption mitigate data-breach, insider, and regulatory risks.
- **Operational resilience**
  - Multi-AZ deployment, managed backups, and health-checked load balancers minimize downtime that could delay business insights.
- **Fast developer onboarding**
  - Session Manager browser SSH removes VPN friction; central secrets rotation and standard images let engineers ship faster.
- **Key business risks**
  - Cloud cost overrun if auto-scaling limits mis-tuned.
  - Talent dependency on specialised AWS skills; mitigation: IaC & runbooks.

---
</details>

### Technical Perspective
<details>
<summary>Logical layers, traffic flow, and component roles</summary>

---

#### Presentation / Access Layer
- **Network Load Balancer (NLB)**
  - Terminates external `443/HTTPS`, `22/SSH`, `389/LDAP`, distributes to private subnets.
- **AWS Session Manager**
  - Browser-based bastion; no inbound SG rules required.

---
#### Computation Layer
- **EC2 Auto Scaling Groups**
  - `m5.xlarge` worker fleet in AZ-1a & AZ-1b; host Spark, dbt, Airflow, or custom pipelines.
  - User-data bootstraps from hardened AMI; registered in FreeIPA.
- **FreeIPA Directory**
  - Master/replica pair provides LDAP/Kerberos auth & sudo policies.

---
#### Storage Layer
- **Amazon EFS**
  - Multi-AZ, AES-256 encrypted, burst or provisioned throughput NFS share for pipeline staging & shared datasets.
- **S3 (via gateway endpoint)**
  - Raw & curated data lake buckets (not shown on diagram for brevity).

---
#### Security & Operations Layer
- **AWS Secrets Manager**
  - Centralised credential store with 30-day rotation.
- **CloudWatch & CloudWatch Logs**
  - Metric/alarm dashboards, log retention 90 days; events forward to PagerDuty.
- **AWS Backup**
  - Daily backups (7 days) + weekly (4 weeks) for EFS & FreeIPA EBS.
- **Systems Manager Patch Manager**
  - Automatic CVE patch waves (dev → prod) with change calendar.

---
#### Network & Connectivity
- **VPC (10.0.0.0/16)**
  - Public subnets (NLB + NAT), private app subnets (workers, FreeIPA), private data subnets (EFS MT).
- **VPC Private Endpoints**
  - SSM, S3, SecretsManager eliminate NAT cost for control-plane calls.
- **NAT Gateway**
  - Outbound internet for package mirrors and third-party APIs; one per AZ for HA.

---
</details>

---

## Technical Analysis

---

### Component Rationale
<details>
<summary>Why each service was selected and trade-offs</summary>

---

#### Compute (`EC2 + ASG`)
- *Pros*: Full OS control (POSIX ACLs, GPU option), mature FreeIPA integration.
- *Cons*: Ops overhead vs. managed EMR/EKS; step-functionless recovery scripts needed.

---
#### Directory (FreeIPA on EC2)
- *Pros*: Open-source, multi-protocol (LDAP/Kerberos), integrates with on-prem AD.
- *Cons*: Self-managed patching; consider AWS Managed Microsoft AD for pure AD shops.

---
#### Storage (`EFS`)
- *Pros*: Shared POSIX file-system semantics, sub-second fail-over across AZs.
- *Cons*: Pricey at scale; maximum 10 GB/s; metadata-heavy workloads may hit IOP caps.

---
#### Network Load Balancer
- *Pros*: L4 performance, static IPs, 20 ms TLS termination.
- *Cons*: No WAF/L7 rules—pair with Network Firewall if needed.

---
#### Secrets Manager vs. Parameter Store
- *Pros*: Native rotation lambda, KMS envelope encryption.
- *Cons*: Higher $ per secret; small secrets cache may incur latency.

---
#### CloudWatch
- *Pros*: Native agent, cross-service correlations, single bill.
- *Cons*: 5 GB free logs only; retention fees accumulate—plan log lifecycle.

---
</details>

---
### NFS Alternative Proposal
---

<details>
<summary>When EFS Caps Are Hit. Need higher throughput, bigger dataset, or global access needs</summary>

---

#### Alternative 1 – Amazon FSx for Lustre
- *Why*: Up to `100 GB/s` & millions IOPS; tight S3 integration for lakehouse exports.
- *Impact*: Requires client driver; bursty cost model per GB / throughput unit.

---
#### Alternative 2 – Delta Lake on Amazon S3 + EMR Serverless
- *Why*: Object storage scales virtually unlimited; ACID via `delta-spark` without shared NFS.
- *Impact*: Migration of pipeline code to Spark/DataFrames; no POSIX file locks.

---
</details>

---

## Cost Estimation

---

### Indicative Monthly Spend (ap-southeast-1)
<details>
<summary>On-demand with 1-year Standard RI where noted</summary>

---

- **Compute**
  - Assume 8 × `m5.xlarge` (50 % RI) → `$2,600`
  - 2 × `c5.large` FreeIPA → `$134`
- **EFS**
  - 5 TB standard + 500 MiB/s provisioned → `$1,350`
- **NLB (2 AZ)**
  - LCU & data → `$120`
- **NAT Gateways (2)**
  - 2 × `$32.40` + 1 TB data → `$190`
- **CloudWatch**
  - 200 GB logs ingested + 90 days retention → `$70`
- **Secrets Manager**
  - 50 active secrets → `$60`
- **AWS Backup**
  - 5 TB EFS snapshots (incremental) → `$100`
- **SSM & Endpoints**
  - Sessions & endpoints (minimal) → `$20`
- **Total (approx.)**
  - **`$4,644`/month**  
    - *50 % savings potential* via Graviton instances, Smart Tier on EFS, and Compute Savings Plans.

---
</details>

---

## Deployment chronology
<details>
<summary>project deployment chronology step by step</summary>
```mermaid
gantt
    title AWS Data Platform – Deployment Chronology (Assume kick off date is 7-7-2025)
    dateFormat  YYYY-MM-DD
    excludes    weekends

    section Blockers & Foundations
    Repo / CI-CD pipeline [E1]               :active, task_repo, 2025-07-07, 3d
    IAM baseline roles & policies [E2]       :task_iam,  after task_repo, 2d
    VPC & Subnet provisioning [E3]           :task_vpc,  after task_repo, 3d
    Security groups & KMS keys [E4]          :task_sg,   after task_iam, 2d

    section Core Networking
    VPC Endpoints (SSM,S3,SM) [E3]           :task_vpcep, after task_vpc, 2d
    NAT Gateways (HA) [E3]                   :task_nat,   after task_vpc, 1d

    section Storage
    EFS + Mount Targets (Multi-AZ) [E4]      :task_efs,   after task_sg, 2d

    section Identity (critical)
    FreeIPA Master (AZ-1a) [E1]              :crit, task_ipa_master, after task_vpcep, 2d
    FreeIPA Replica (AZ-1b) [E1]             :crit, task_ipa_replica, after task_ipa_master, 2d
    Replication validation [E1,E2]           :task_ipa_validate, after task_ipa_replica, 1d

    section Compute Fleet
    Hardened AMI build [E2]                  :task_ami,  after task_ipa_validate, 3d
    Auto Scaling Group config [E2]           :task_asg,  after task_ami, 1d
    Worker nodes launch [E2]                 :task_workers, after task_asg, 2d
    Network Load Balancer & TGs [E3]         :task_nlb,  after task_asg, 2d
    Join workers to FreeIPA [E1,E2]          :task_join, after task_workers, 1d

    section Operations & Security
    Session Manager baseline [E3]            :task_ssm,   after task_nlb, 1d
    CloudWatch agent + dashboards [E4]       :task_cw,    after task_ssm, 2d
    Secrets Manager + rotation [E4]          :task_sm,    after task_ssm, 2d
    AWS Backup plans (EFS & EBS) [E4]        :task_backup, after task_sm, 1d

    section Validation & Hardening
    Integration tests (pipeline smoke) [E2]  :task_test,   after task_backup, 2d
    Security assessment / pen-test [E3]      :task_secassess, after task_test, 2d
    Performance & load tests [All]           :task_perf,  after task_test, 3d

    section Closure
    Cost-monitoring baseline [E4]            :task_cost,  after task_perf, 1d
    Documentation & Knowledge Transfer [All] :milestone, task_docs, after task_cost, 2d

```
</details>
## Infrastructure as Code (Terraform)

---

### terraform\_component\_list

<details>
<summary>All resources provisioned via Terraform</summary>

---

* **AWS Identity & Access Management (IAM)**

  * `service roles`, `instance profiles`, `inline policies`, `managed policies`, `IAM groups/users`
* **Networking**

  * `VPC`, `public-subnets`, `private-app-subnets`, `private-data-subnets`, `endpoint-subnet`, `route-tables`, `NACLs`
  * `Internet Gateway`, `NAT Gateways (per-AZ)`
  * `VPC Interface/Gateway Endpoints` for `SSM`, `Secrets Manager`, `S3`
* **Security**

  * all `Security Groups` and rules
  * customer-managed `KMS keys` for `EFS`, `Secrets Manager`, `Backup`, `S3`
  * `AWS Config`, `GuardDuty`, `CloudTrail` trails with `S3` + `CloudWatch Logs` destinations
* **Compute & Scaling**

  * `EC2 Launch Templates`, `Auto Scaling Groups` for worker fleet
  * `EC2 instances` for `FreeIPA (master & replica)` with `EBS` volumes and tags
* **Load Balancing & Networking Extras**

  * `Network Load Balancers` (multi-AZ listeners, target groups, health checks)
  * `Route 53` private hosted-zone records (optional internal DNS)
* **Storage & Data Services**

  * `Amazon EFS` file system + mount targets + performance & throughput configuration
  * `S3 buckets` for `raw`, `staged`, `curated` data lake & for `logs/artifacts`
  * `ECR repositories` for container images
* **Operations & Resilience**

  * `CloudWatch Log Groups`, `metric alarms`, `dashboards`
  * `AWS Backup` vaults & backup plans (`EFS`, `FreeIPA EBS`)
  * `EventBridge rules` for compliance / backup notifications
* **CI/CD Foundations**

  * `CodeBuild projects`, `CodePipeline pipelines` for  *image build/push* & *Terraform automation*
  * `SSM Maintenance Windows` / `Patch Manager` baseline definitions
* **Governance**

  * comprehensive `tags` & `resource groups` for cost allocation and reporting

---

</details>

---


### ansible\_component\_list

<details>
<summary>All configuration and operations handled by Ansible</summary>

---

* **OS Baseline & Hardening**

  * disable unnecessary services, apply `CIS hardening`, configure `chrony/NTP`, enable `auditd`
* **Package & Runtime Installation**

  * install `Docker/Podman/containerd`, `Java`, `Python`, `Scala`, and system libraries for `Spark`, `dbt`, `Airflow`
  * configure `CloudWatch Agent` & `SSM Agent` (if not pre-baked in AMI)
* **Application Layer**

  * deploy & update container images: `spark-runtime`, `dbt-runner`, `airflow`, `data-pipeline-custom`
  * create `systemd` units or `Docker-Compose / ECS-Anywhere` manifests
* **FreeIPA Setup & Management**

  * install `FreeIPA` server/client, provision domain, configure replica & replication checks
  * import bootstrap `users/groups`, apply `HBAC` & `sudo` policies
* **Secrets & Credentials Handling**

  * bootstrap scripts to pull secrets from `AWS Secrets Manager` and inject into configs / env-vars
* **Logging & Monitoring Agents**

  * configure `CloudWatch Agent` JSON, log files & custom metrics
  * optionally install `Prometheus node exporter`
* **Continuous Deployment Hooks**

  * rolling or blue-green updates for worker `AMIs` / containers
  * register new EC2 nodes with `FreeIPA`, deregister terminated nodes
* **Day-2 Operations & Maintenance**

  * patching playbooks (triggered via `SSM` or `Ansible AWX`)
  * backup verification tasks, restore drills, security-baseline drift checks
* **AMI Bake Pipeline (optional)**

  * use `Packer + Ansible` to build hardened golden `AMIs` referenced by Launch Templates

---

</details>


### terraform\_deployment\_groups

---

#### group\_1\_identity\_and\_access

<details>
<summary>Provision shared security primitives first</summary>

---

* create backend `S3` bucket + `DynamoDB` lock table for remote state
* deploy organisation or account-level `IAM` roles, instance profiles, and least-privilege `policies`
* generate customer-managed `KMS` keys (`alias/data-efs`, `alias/data-s3`, `alias/data-backup`) with key-admin and key-usage roles
* enable `CloudTrail`, central `S3` logging bucket, and org-wide `GuardDuty` + `Security Hub`
* output `kms_key_arns`, `iam_role_arns`, and `state_bucket` for downstream modules

---

</details>

#### group\_2\_stable\_core\_infrastructure

<details>
<summary>Lay down networking and other rarely-changing foundations</summary>

---

* create single `VPC (10.0.0.0/16)` with four application sub-nets + one endpoints sub-net
* attach `Internet Gateway`, route tables, `NAT Gateways` (one per AZ)
* provision `VPC Interface/Gateway Endpoints` for `SSM`, `Secrets Manager`, `S3`
* define all baseline `Security Groups` (ingress/egress only, no instance IDs yet)
* allocate `Amazon EFS` + mount targets in each AZ, encrypted with `KMS` from Group 1
* register `Route 53` private hosted zone (optional) and seed base records
* export subnet IDs, security-group IDs, and EFS file-system ID for Group 3

---

</details>

#### group\_3\_change\_prone\_compute\_and\_services

<details>
<summary>Spin up compute, load-balancing, and ops resources</summary>

---

* build `EC2 Launch Templates` referencing golden AMIs and `iam_instance_profiles` from Group 1
* create `Auto Scaling Groups` for data-worker fleet across private-app sub-nets
* launch `FreeIPA Master` and `Replica` instances with stitched‐in `user-data`
* deploy `Network Load Balancers`, listeners, target groups, and health checks
* build `AWS Backup` vault + plans targeting EFS and FreeIPA EBS volumes
* create `CloudWatch` log groups, metric alarms, dashboards, and `EventBridge` rules
* stand-up `ECR` repos, `CodeBuild` projects, and `CodePipeline` for CI/CD workflows
* outputs feed Ansible inventory: `worker_private_ips`, `freeipa_master_ip`, `efs_dns`

---

</details>

---

### ansible\_deployment\_phases

---

#### phase\_a\_freeipa\_bootstrap

<details>
<summary>Configure identity backbone before touching workers</summary>

---

* harden OS, install `freeipa-server` packages on master node
* initialise FreeIPA domain, enable replication ports, create admin service accounts
* install `freeipa-server` on replica and join to master with replication checks

---

</details>

#### phase\_b\_worker\_os\_baseline

<details>
<summary>Prepare all worker nodes for platform runtimes</summary>

---

* apply CIS level-1 hardening, configure `chrony`, `auditd`, and required kernel params
* install `Docker` (or `containerd`), `Java 11`, `Python 3.x`, `Scala`, and common libs
* enrol each node into FreeIPA (`ipa-client-install`) for LDAP/Kerberos auth
* deploy and start `CloudWatch Agent` & validate log/metric flow to CloudWatch

---

</details>

#### phase\_c\_platform\_runtime\_deploy

<details>
<summary>Lay down Spark, dbt, Airflow, and custom pipelines</summary>

---

* pull signed images from `ECR` and load into local container runtime
* render config files from templates, injecting secrets via `aws-secretsmanager` lookup
* create `systemd` units or `compose` stacks; validate service health locally
* mount shared datasets from `EFS` and run smoke tests against sample data

---

</details>

#### phase\_d\_day\_2\_operations

<details>
<summary>Enable ongoing maintenance and updates</summary>

---

* configure `SSM Patch Manager` baseline tags and Ansible playbook hooks
* schedule rolling AMI or container updates through Ansible AWX pipelines
* run backup validation playbooks, restore drills, and security-drift scans
* export compliance reports to `S3` and notify `PagerDuty` via EventBridge rule

---

</details>

---

### orchestration\_flow

---

#### workflow\_summary

<details>
<summary>End-to-end execution order and gating logic</summary>

---

* run `terraform init/plan/apply` for **Group 1** → obtain remote-state backend & `kms` keys
* run `terraform apply` for **Group 2** (depends on Group 1 outputs)
* run `terraform apply` for **Group 3** (references IDs from Group 2)
* collect dynamic inventory from Terraform state; feed into Ansible controller
* execute Ansible **Phase A → B → C → D** in sequence, halting on any failure
* guard each stage with `environment` and `sensitive` approver blocks inside the CI/CD pipeline

---

</details>

---

## repo\_structure

---

### terraform\_repository

<details>
<summary>Directory layout and content owned by Terraform</summary>

---

* **Directory tree**

  ```bash
  terraform/
  ├── globals/            # IAM, KMS, CloudTrail, remote-state backend
  ├── networking/         # VPC, subnets, IGW, NAT, endpoints
  ├── platform/           # ASG, EFS, FreeIPA, NLB, ECR, Backup
  ├── modules/            # Re-usable opinionated TF modules
  │   ├── vpc/
  │   ├── kms/
  │   ├── sg/
  │   ├── efs/
  │   ├── asg/
  │   └── nlb/
  ├── environments/       # Layered workspaces
  │   ├── dev/
  │   ├── staging/
  │   └── prod/
  ├── pipelines/          # CodeBuild ⟶ CodePipeline definitions
  ├── scripts/            # Helper wrappers (fmt/lint/plan/apply)
  └── README.md
  ```
* **State strategy**

  * `globals/` creates remote-state `S3` bucket + `DynamoDB` lock
  * each folder keeps its own backend stanza (`key = env/name.tfstate`)
* **Module conventions**

  * version-pinned providers via `terraform.tf` in root
  * `modules/*` expose variables with sane defaults; no provider blocks
* **CI hooks**

  * pre-commit running `tflint`, `tfsec`, `terraform validate`, `checkov`

---

#### modules\_breakdown

* `modules/vpc` – parametrised multi-AZ VPC with route-tables & NACLs
* `modules/sg` – opinionated security-group factory with rule lists
* `modules/efs` – EFS, mount targets, SG, lifecycle policies
* `modules/asg` – launch template + mixed-instances policy + scaling policies
* `modules/nlb` – cross-AZ Network LB, listeners, target groups, HC’s

---

</details>

### ansible\_repository

<details>
<summary>Playbooks, roles, inventory, and AMI bake pipeline</summary>

---

* **Directory tree**

  ```bash
  ansible/
  ├── inventories/
  │   ├── dynamic_terraform.yml   # Pulls hosts/vars from TF state
  │   └── group_vars/
  ├── roles/
  │   ├── os_hardening/
  │   ├── freeipa_server/
  │   ├── freeipa_client/
  │   ├── docker_runtime/
  │   ├── spark_runtime/
  │   ├── airflow/
  │   ├── dbt_runner/
  │   └── logging_agent/
  ├── playbooks/
  │   ├── 10_freeipa_bootstrap.yml
  │   ├── 20_worker_baseline.yml
  │   ├── 30_platform_deploy.yml
  │   └── 40_day2_ops.yml
  ├── packer/
  │   └── freeipa_ami.pkr.hcl
  ├── files/               # Static artefacts
  ├── templates/           # Jinja2 configs
  ├── scripts/             # Utility wrappers & Molecule tests
  └── README.md
  ```
* **Inventory**

  * dynamic plugin reads Terraform outputs; tag-based grouping (`role`, `env`)
* **Vault & secrets**

  * `ansible-vault` encrypted vars for bootstrap secrets & tokens
* **CI pipeline**

  * GitHub Actions workflow: `ansible-lint`, `molecule`, then AWX job-template trigger

---

#### role\_conventions

* self-contained roles with `tasks/handlers/defaults/vars/templates/files`
* idempotency validated in `check_mode` pipelines
* role tags follow `role:<name>` + `tier:<layer>` + `env:<workspace>` patterns

---

</details>

### shared\_ci\_cd

<details>
<summary>Cross-tooling pipelines and developer ergonomics</summary>

---

* **`.github/workflows/`**

  * `terraform-ci.yml` – fmt ➜ lint ➜ plan ➜ apply (protected branch, manual approval)
  * `ansible-ci.yml` – lint ➜ molecule ➜ deploy via AWX API
* **`scripts/`** (repo-root)

  * `terraform-wrapper.sh` – standardises backend selection and var-file injection
  * `ansible-wrapper.sh` – dynamic-inventory generator + vault helpers
* **Pre-commit config** applies to both `/terraform` and `/ansible` paths

---

</details>

---

## Access Control & Security Architecture


---

### iam\_role\_catalog

<details>
<summary>Key IAM roles & purpose</summary>

---

* **`AdminRole`** — break-glass privileges; MFA enforced; no persistent keys
* **`TechLeadRole`** — change-management approve/merge; read/write TF state; CloudWatch dashboards
* **`DevOpsRole`** — full CRUD on Terraform state bucket, CodeBuild/CodePipeline, ECR; no direct data-lake access
* **`DataEngineerRole`** — read/write S3 data-lake buckets; start/stop Airflow & Spark jobs; assume via FreeIPA + SSO
* **`SecurityEngineerRole`** — GuardDuty, Config, IAM Access Analyzer, KMS key-admin; cannot alter workload resources
* **`AuditorRole`** — CloudTrail, Config read-only; cannot change encryption keys or SGs
* **`EC2WorkerInstanceProfile`** — scoped to S3 path-mappings, EFS mount, SecretsMgr read, CloudWatch putMetrics
* **`FreeIPAInstanceProfile`** — SecretsMgr read (directory bootstrap), KMS decrypt on FreeIPA-EBS key
* **`CICDPipelineRole`** — assume by CodeBuild; limited to ECR push, Terraform plan/apply via OIDC trust
* **`BackupServiceRole`** — managed by AWS Backup; scoped to copy EFS & EBS snapshots to vault
* **`SessionManagerRole`** — SSM core + ssm\:StartSession only on tagged bastionless instances

---

</details>

---

### iam\_principal\_policies

---

<details>
<summary>Least-privilege boundaries & rotation controls</summary>

---

* **Access boundaries** on human roles → deny outside `ap-southeast-1`, production VPC tag safeguard
* **Managed policies** version-pinned; no wildcard `*` actions except AWS-managed `ReadOnlyAccess`
* **KMS key policies** delegate to roles, not users; mandatory encryption on S3/EFS/EBS
* **Secrets rotation** → Secrets Manager 30-day schedule; Lambda rotation function via `CICDPipelineRole`
* **IAM creds** — no IAM users; all CLI/API via federated SSO (AWS IAM Identity Center) + MFA
* **Session policies** — 8-hour duration max; condition keys `aws:RequestedRegion`, `aws:TagKeys`
* **SCPs** at OU level — deny `iam:CreateAccessKey`, enforce `kms:Encrypt` on all `s3:PutObject`

---

</details>

---

### raci\_matrix

---

| Task / Artifact                     | Project Manager | Tech Lead | DevOps Engineer | Data Engineer | Security Engineer | Auditor |
| ----------------------------------- | :-------------: | :-------: | :-------------: | :-----------: | :---------------: | :-----: |
| Define IAM roles & policies         |        A        |     R     |        C        |       C       |         I         |    I    |
| Implement Terraform IAM modules     |        I        |     C     |        R        |       I       |         C         |    A    |
| Approve production policy changes   |        C        |     R     |        I        |       I       |         A         |    I    |
| Review CloudTrail / Config findings |        I        |     I     |        C        |       I       |         R         |    A    |
| SG rule change requests             |        I        |     R     |        C        |       C       |         A         |    I    |
| Secrets rotation verification       |        I        |     C     |        R        |       I       |         A         |    I    |
| Audit access reports & evidence     |        I        |     I     |        C        |       I       |         R         |    A    |

*R = Responsible A = Accountable C = Consulted I = Informed*

---

### security\_group\_strategy

---

* **Tiered SG model**

  * **edge-sg** → NLB listeners; ingress only 443/22/389 from `0.0.0.0/0` with AWS WAF future hook
  * **app-sg** → worker ASG; allow NFS 2049 from `data-sg`, LDAP 389 from **auth-sg**, egress 443 to VPC endpoints
  * **auth-sg** → FreeIPA; ingress 22/389/636 from **app-sg**, replication 389 from peers, egress 443 to SecretsMgr
  * **data-sg** → EFS mount targets; ingress 2049 from **app-sg** only
* **No public inbound** to EC2; admin access via SSM Session Manager tunnel
* **Self-service rules** denied; changes via Terraform MR + security review
* **Default-deny** outbound except required AWS services (VPC endpoints, NAT range)

---

### authentication\_flow

---

* **User login**: IAM Identity Center (SAML/OIDC) → assume `TechLeadRole` / `DevOpsRole` / `DataEngineerRole` with MFA
* **Bastionless access**: AWS SSM Session Manager → attach `SessionManagerRole` → start session within worker VPC
* **Service auth**:

  * EC2 userdata fetches temporary creds via instance profile (`EC2WorkerInstanceProfile`)
  * Application containers pull secrets (`db.password`, `api_key`) from Secrets Manager with role-based Decrypt
  * Apps mount EFS using TLS + IAM POSIX UID mapping
* **Inter-service auth**:

  * Workers authenticate to FreeIPA using Kerberos tickets (forwardable)
  * FreeIPA replication secured by GSS-API over 389/TCP
* **Data access**: Spark/dbt jobs assume `DataPipelineExecutionRole` (derived from worker profile) → S3 access with IAM condition keys `s3:prefix=${project}/`
* **Monitoring & backup**: CloudWatch agent uses signed OpenTelemetry endpoint; Backup Service Role snapshots EFS/EBS encrypted with KMS key
* **Audit trail**: CloudTrail + Session Manager logs → centralized S3, immutable via Object Lock, analyzed by Athena

---

### tool\_reference

---

* **AWS Secrets Manager** — credential store & rotation
* **AWS Session Manager** — bastionless SSH/RDP; logging to CloudWatch & S3
* **FreeIPA** — centralized LDAP/Kerberos + HBAC policies
* **IAM Access Analyzer** — detect public or cross-account access drift
* **AWS Config** — conformance packs checking SG open ports, unencrypted resources

---

## System Integration

<details>
<summary>System integration: connectivity, data flows, and network architecture</summary>

---

### Core system components

* **Compute nodes** – `EC2 Auto Scaling Workers`, `FreeIPA Master`, `FreeIPA Replica`, `CodeBuild / CodePipeline` containers
* **Networking** – `Network Load Balancers`, `Internet Gateway`, `NAT Gateways`, `VPC Interface Endpoints (SSM, Secrets Manager)`, `VPC Gateway Endpoint (S3)`, `Route 53` private zone
* **Storage** – `Amazon EFS` (multi-AZ), `Amazon S3` buckets (raw, staged, curated, logs), `Amazon ECR` repositories, `EBS` volumes
* **Security & ops** – `AWS Secrets Manager`, `AWS Systems Manager`, `AWS CloudWatch`, `AWS Backup`, `GuardDuty`, `AWS Config`, `CloudTrail`
* **Application runtimes** – `Spark`, `Airflow`, `dbt`, `custom ETL` containers
* **External actors** – business users / data scientists (HTTPS), DevOps & engineers (SSM Session Manager)

---

### Components Connectivity relationships

* **Users → NLB**: `HTTPS/443`
* **Engineers → SSM VPCE**: `TLS` (SSM StartSession)
* **NLB → Workers**: `HTTPS/443`, `SSH/22`
* **NLB → FreeIPA Master**: `LDAP/389`, `LDAPS/636`
* **Workers ↔ FreeIPA**: `LDAP/389`, `Kerberos/88`, `LDAPS/636`
* **FreeIPA Master ↔ Replica**: `LDAP/389` (replication)
* **Workers → EFS**: `NFS/2049`
* **Workers → S3 (GW VPCE)**: `HTTPS/443`
* **Workers / FreeIPA → Secrets Manager VPCE**: `HTTPS/443`
* **Workers → SSM VPCE**: `HTTPS/443`
* **Workers → NAT Gateway → Internet**: `HTTPS/443`
* **CloudWatch Agent → CloudWatch**: `HTTPS/443`
* **CodeBuild → ECR / TF-state S3**: `HTTPS/443`

---

### End-to-end data flows

* **Ingestion**: external uploads or API pulls → `S3 raw`
* **Processing**: Airflow triggers Spark → read `S3 raw`, stage on `EFS`, write `S3 staged` → dbt → `S3 curated`
* **Secrets & auth**: Kerberos tickets from FreeIPA; secrets via Secrets Manager VPCE
* **Archival**: EFS snapshots via AWS Backup; S3 lifecycle to Glacier
* **Consumption**: Athena/EMR on `S3 curated` → results via NLB
* **Observability**: CloudWatch metrics/logs, SNS alarms to PagerDuty

---

### Network segmentation & security zones

* **Public subnets** – NLB, NAT; inbound limited to 443/22/389
* **Private-App subnets** – workers, FreeIPA; east-west limited to LDAP/Kerberos/NFS; outbound via NAT
* **Private-Data subnets** – EFS mount targets; ingress `NFS/2049` only
* **Endpoints subnet** – VPCEs (SSM, Secrets Manager) + S3 Gateway; no direct Internet route
* **Controls** – default-deny SGs, VPC Flow Logs, admin only through SSM

---

### Network & data-flow diagram

```mermaid
graph TD
    Users[👥 Users]
    Engineers[🔧 Engineers]
    NLB[NLB public]
    SSMVPCE[SSM VPC Endpoint]
    NAT[NAT Gateway]
    Internet[🌐 Internet]
    CloudWatchAgent[📈 CW Agent]
    CloudWatch[CloudWatch]
    CodeBuild[🛠️ CodeBuild]
    ECR[ECR]
    TFState[S3 TF State Bucket]

    subgraph "Private-App Subnets"
        Workers[🏗️ Worker ASG]
        FreeIPA1[🔐 FreeIPA Master]
        FreeIPA2[🔐 FreeIPA Replica]
    end

    subgraph "Private-Data Subnets"
        EFS[💾 EFS Mount Targets]
    end

    subgraph "Endpoints Subnet"
        Secrets[🔑 Secrets Mgr VPCE]
        S3GW[S3 Gateway Endpoint]
    end

    Users -->|443| NLB
    Engineers -->|TLS| SSMVPCE
    Users -->|443/22| NLB
    NLB -->|443/22| Workers
    NLB -->|389/636| FreeIPA1
    Workers -->|389/88/636| FreeIPA1
    FreeIPA1 -.->|replication 389| FreeIPA2
    Workers -->|2049| EFS
    Workers -->|443| S3GW
    Workers -->|443| Secrets
    Workers -->|443| SSMVPCE
    Workers -->|443| NAT
    NAT -->|443| Internet
    CloudWatchAgent -->|443| CloudWatch
    CodeBuild -->|push/pull| ECR
    CodeBuild -->|plan/apply| TFState

```

---

</details>

---

## Operational Procedures

<details>
<summary>Operational Procedures: monitoring, backup, and maintenance</summary>

### metrics\_alarms\_dashboards

<details>
<summary>Key metrics, alarm thresholds, and CloudWatch dashboards</summary>

---

* **EC2 Worker Fleet**

  * `CPUUtilization` > 85 % for 5 min → *Scale-up* alarm
  * `mem_used_percent` (CW Agent) > 90 % for 5 min → *Critical* alarm
  * `disk_used_percent` > 80 % (root & /data) → *Warning* alarm
  * `AutoScalingGroupDesiredCapacity` vs. `InService` mismatch > 2 instances for 10 min → *Capacity Drift* alarm

* **EFS**

  * `BurstCreditBalance` < 5 GiB → *Throughput Risk* alarm
  * `PercentIOLimit` > 80 % → review performance mode

* **FreeIPA**

  * Custom metric `ldap_bind_failures` > 50 in 5 min → *Auth Failure Spike* alarm
  * `StatusCheckFailed_System` > 0 → *Instance Impaired*

* **Networking**

  * `NATGatewayBytes` > 80 % of baseline for 30 min → *Cost Spike* alert
  * `RejectedConnectionCount` (VPC Flow Logs) trend up 3 × baseline → *SG Misconfig* warning

* **Security & Compliance**

  * GuardDuty finding severity ≥ 4 → *Critical Security* alarm
  * Config rule non-compliant resources count > 0 → *Compliance Drift*

* **Dashboards**

  * **Platform-Health**: ASG capacity, CPU/Mem, EFS IO, NAT bytes, top GuardDuty findings
  * **Data-Pipeline**: Spark job runtime, Airflow DAG success rate, dbt model failures, S3 throughput

---

</details>

### logging\_flows

<details>
<summary>Log generation, destinations, and retention</summary>

---

* **EC2 System Logs** → `cwlogs:/ec2/system/<env>` — retention 90 days
* **Application Logs**

  * `Spark`, `Airflow`, `dbt` containers → `cwlogs:/apps/<service>/<env>` — retention 90 days
  * Log format: JSON-structured with `level`, `ts`, `job_id` fields
* **FreeIPA Logs** (`/var/log/dirsrv`, `/var/log/krb5kdc`) → `cwlogs:/freeipa/<env>` — retention 180 days
* **SSM Session Logs** → `s3://org-audit-logs/ssm/` + `cwlogs:/ssm/sessions` — S3 object lock 1 year
* **CloudTrail** org trail → `s3://org-audit-logs/cloudtrail/` — Glacier after 365 days
* **VPC Flow Logs** all subnets → `cwlogs:/vpc/flow/<vpc_id>` — retention 30 days; Athena partition for ad-hoc queries
* **GuardDuty & Security Hub Findings** → `cwlogs:/security/findings` + EventBridge → Slack `#sec-alerts`

---

</details>

### alerting\_escalation

<details>
<summary>Alert routing, escalation paths, and run-books</summary>

---

* **Alarm Targets**

  * All CloudWatch alarms → `SNS:platform-alerts`
  * `SNS:platform-alerts` subscriptions: `PagerDuty Events V2`, `Slack Webhook`, email `on-call@company.com`
* **Severity Tags**

  * *CRITICAL* — page on-call immediately (PD severity 2)
  * *WARNING* — Slack notification + PD low-urgency (severity 3)
  * *INFO* — Slack only, no paging
* **Escalation Policy**

  1. On-call DevOps (15 min)
  2. Secondary DevOps (15 min)
  3. Tech Lead + Security Engineer (until acknowledged)
* **Run-book Links**

  * Attached to each alarm via `AWS::CloudWatch::Alarm` `AlarmActions` annotation (S3 markdown URL)
  * Example: `s3://runbooks/platform/efs-burstcredit.md`
* **Drill Schedule**

  * PagerDuty “Failure Friday” — quarterly synthetic alarm injection, verify paging & resolution
  * Slackbot posts summary of drill outcome in `#platform-ops`

---

</details>

---

### backup\_coverage

<details>
<summary>Resources protected & backup mechanism</summary>

---

* **Amazon EFS**

  * Policy-based backups via `AWS Backup` (resource assignment tag `backup:tier=gold`)
  * Daily point-in-time file-system snapshot
* **EBS Volumes**

  * FreeIPA root & data volumes, worker root volumes
  * `AWS Backup` incremental snapshots
* **Amazon S3**

  * Buckets: `raw`, `staged`, `curated`, `logs`
  * Versioning + MFA-Delete enabled
  * Lifecycle transition to `Glacier Flexible Retrieval`
* **AWS Backup Vault**

  * KMS-encrypted (`alias/backup‐vault`)
  * Cross-Region copy to `ap-northeast-1` for DR
* **RDS/Aurora (future)**

  * Placeholder module – automated daily snapshots
* **AWS Backup Audit Manager**

  * Compliance report generation, stored in `s3://org-audit-logs/backup-reports/`

---

</details>

### retention\_schedule

<details>
<summary>Frequency & retention policies</summary>

---

| Resource             | Frequency | Retain  | Transition / Copy                      |
| -------------------- | --------- | ------- | -------------------------------------- |
| **EFS**              | Daily     | 7 days  | Weekly copy → DR region (7 copies)     |
| **EFS**              | Weekly    | 4 weeks | Monthly copy → DR region (12 copies)   |
| **EBS (FreeIPA)**    | Daily     | 7 days  | None                                   |
| **EBS (Workers)**    | Daily     | 3 days  | None                                   |
| **S3 Versions**      | Immediate | 30 days | Glacier after 30 d, delete after 365 d |
| **AWS Backup Vault** | N/A       | 1 year  | Cross-Region vault copy                |

---

</details>

### restore\_and\_validation

<details>
<summary>Restore drills & integrity checks</summary>

---

* **Monthly EFS Restore Test**

  * Automated `AWS Backup` restore to *staging* VPC
  * Run checksum diff against live dataset (`rsync --dry-run --checksum`)
* **Quarterly FreeIPA DR Drill**

  * Restore EBS snapshot to new instance in isolated subnet
  * Verify LDAP bind and Kerberos ticket issuance
* **S3 Object Recovery**

  * Weekly sample restore of randomly selected objects from Glacier
  * Lambda job logs SHA-256 match to `cwlogs:/backup/validation`
* **Documentation & Evidence**

  * Restore steps scripted in SSM Automation document `AWS-RunRestoreValidation`
  * Output artifacts uploaded to `s3://org-audit-logs/restore-evidence/`
* **Fail Criteria**

  * Data mismatch > 0.1 % or restore duration > RTO (2 h) triggers *CRITICAL* PagerDuty alert
* **Compliance Review**

  * Audit Manager backup assessment – reviewed by Security Engineer monthly

---

</details>

---

### patch\_management

<details>
<summary>How & when patches are applied</summary>

---

* **Tools & Channels**

  * `SSM Patch Manager` baselines (`Critical`, `Security`)
  * `Maintenance Windows` tagged `patch:window=weekly`
  * `Ansible AWX` playbook `os_patch.yml` for edge cases
* **Schedule**

  * **Dev**: every **Tuesday 02:00 UTC+7**
  * **Staging**: every **Wednesday 02:00 UTC+7**
  * **Prod**: every **Saturday 03:00 UTC+7** (1-hour change window)
* **Process**

  * Pre-patch snapshot via `AWS Backup` (EBS)
  * SSM installs available patches, reboots if required
  * Post-patch health verification (CloudWatch agent heartbeat)
  * Automated Slack summary to `#ops-patching` with patch counts & reboot list
* **Approval Gates**

  * Prod window requires `TechLead` manual approval in CodePipeline
  * Emergency CVE: run `SSM Automation` doc `AWS-PatchEmergency` (bypass schedule)

---

</details>

### health\_checks\_and\_drift\_detection

<details>
<summary>Continuous validation of system state</summary>

---

* **Health Checks**

  * **ASG instance health** – replace unhealthy > 15 min
  * **ELB/NLB target health** – alarm on failed > 5 of 5 checks
  * **FreeIPA replication** – cron job `ipa-replica-manage list` → CW metric `replication_status`
  * **EFS mount availability** – CW Agent `nfs_avail` metric (1=OK,0=Fail)
* **Drift Detection**

  * **Terraform plan** every PR & nightly in CodeBuild (dry-run)
  * **AWS Config rules** – detect SG 0.0.0.0/0 ports, unencrypted volumes, IAM key age > 90 d
  * **Ansible check-mode** weekly run to flag config drift, results to `cwlogs:/ansible/drift`
  * Deviations auto-tagged `drift=true` for quick filtering
* **Remediation**

  * Auto-remediate minor drift (SG revoke) via `SSM Automation`
  * Major drift opens JIRA ticket via EventBridge → Lambda

---

</details>

### incident\_response

<details>
<summary>Detection ➜ containment ➜ recovery workflow</summary>

---

* **Detection Sources**

  * CloudWatch *CRITICAL* alarms, GuardDuty severity ≥ 4, Config non-compliance
* **Trigger**

  * EventBridge rule → AWS Incident Manager engagement plan `data-platform-ops`
* **Run Books (SSM)**

  * `AWS-StopEC2` (containment), `AWS-RollbackASG` (scale-in bad deploy), `AWS-RestoreEFS` (data recovery)
* **Escalation**

  * PagerDuty tiered: DevOps (T1) → Tech Lead (T2) → Security Eng (T3)
* **Post-Incident**

  * `5-Why` template stored in Confluence
  * Incident data (CloudTrail, CW Logs) preserved to `s3://org-audit-logs/incidents/<id>/` (Write-Once)
  * Change records updated in Ops CMDB (ServiceNow API)

---

</details>

### routine\_automation\_tasks

<details>
<summary>Scheduled tasks & housekeeping</summary>

---

* **Daily**

  * Rotate IAM access keys for CI/CD roles via Lambda (`access_key_rotate.py`)
  * Export top NAT Gateway byte usage to `Athena` cost table
* **Weekly**

  * Security group rule review report (Config) → Slack
  * EFS throughput & burst credit trend analysis → Tech Lead email
  * Terraform `state pull` checksum archived to S3
* **Monthly**

  * Delete aged ECR image tags (> 60 d) except `prod-*` using `ecr-lifecycle.sh`
  * Verify GuardDuty detectors enabled in all regions
  * Run backup restore drill (see Step 3)
* **Quarterly**

  * Chaos game day: simulate AZ failure with ASG `suspend` / `resume`
  * IAM permission boundary audit via Access Analyzer
* **Automation Framework**

  * All jobs scheduled via **EventBridge** cron → **SSM Automation** or **Lambda**
  * Notifications aggregated in Slack `#ops-digest` with CSV attachment

---

</details>

---

</details>

---

## Implementation Plan & Timeline
---

### Team legend
<details>
<summary>Roles & primary skill domains</summary>

---

| Abbr. | Focus | Key Responsibilities |
|-------|-------|----------------------|
| **PM** | Project Mgr / Scrum Master | Ceremonies, backlog, blockers |
| **E1** | Networking & IaC | VPC, SGs, Terraform core |
| **E2** | Compute & Auth | EC2 ASG, FreeIPA |
| **E3** | Storage & DR | EFS, AWS Backup, S3 |
| **E4** | CI/CD & Observability | GitHub Actions, CloudWatch, cost alerts |

---

</details>

--- 

### Team allocation_matrix
<details>
<summary>Engineer tasks per sprint (all work ≈ 85 % capacity)</summary>

---

| Sprint | E1 | E2 | E3 | E4 |
|--------|----|----|----|----|
| **1** (07 Jul – 20 Jul) | VPC, subnets, NAT, endpoints | Launch Template, stub AMI | EFS-Dev create, KMS | Repo scaffold, CI lint |
| **2** (21 Jul – 03 Aug) | NLB + SG harden, **S3 raw/curated buckets** | ASG-Dev & smoke | FreeIPA master + replica | Secrets rotation, **env-tagged secrets structure**, CW dashboards |
| **3** (04 Aug – 17 Aug) | Stage IaC clone, firewall rules | **Autoscale & NLB fail-over validation**, compute tuning | Backup plan, DR docs, **EFS throughput test**, **synthetic workload generator** | tfsec/cfn-nag, cost anomaly alerts |
| **4** (18 Aug – 01 Sep) | Prod blue/green cut-over, **change-freeze comms** | FreeIPA hardening & audit | EFS snapshot verify, **rollback simulation drill** | KT workshops, runbooks |

---

</details>

---

### sprint_backlog
<details>
<summary>Major deliverables by week</summary>

---

#### **Sprint 1 (Weeks 1-2)**
- VPC & networking IaC live (Dev)
- EFS-Dev provisioned, KMS encryption verified
- CI pipeline & linting in place

#### **Sprint 2 (Weeks 3-4)**
- NLB listeners & hardened SGs deployed
- ASG-Dev smoke test passes
- Raw & curated **S3 buckets** with lifecycle + SSE-KMS
- Secrets Manager rotation & **env separation** established

#### **Sprint 3 (Weeks 5-6)**
- Stage environment cloned via IaC
- **Autoscale scale-out/in & NLB cross-AZ fail-over** tested
- **Synthetic workload** runs against Stage; **EFS I/O stress test** logged
- DR docs, tfsec/cfn-nag scans, cost alerts green

#### **Sprint 4 (Weeks 7-8)**
- Production blue/green deployment & smoke
- **Rollback simulation** completed; change-freeze window enforced
- Knowledge-transfer workshops and runbooks signed off
- Formal hand-over & closure

---

</details>

---

### Roadmap and milestones
```mermaid
gantt
    title AWS Data Platform – Sprint-based Milestone Roadmap (2025)
    dateFormat  YYYY-MM-DD
    axisFormat  %b %d

    section Sprint 1 (07-07 → 07-20)
    Dev VPC & CI pipeline green                     :milestone, m_vpc_ci,  2025-07-18, 0d

    section Sprint 2 (07-21 → 08-03)
    S3 buckets provisioned & secured                :milestone, m_s3,      2025-07-30, 0d

    section Sprint 3 (08-04 → 08-17)
    Dev end-to-end smoke test complete              :milestone, m_smoke,   2025-08-06, 0d
    Autoscale & NLB fail-over validation passed     :milestone, m_nlb,     2025-08-11, 0d
    EFS throughput stress test passed               :milestone, m_efs,     2025-08-13, 0d
    DR fail-over drill successful                   :milestone, m_dr,      2025-08-14, 0d

    section Sprint 4 (08-18 → 08-31)
    Rollback simulation & change-freeze approved    :milestone, m_rb,      2025-08-22, 0d
    Cost-optimisation checkpoint                    :milestone, m_cost,    2025-08-25, 0d
    Production go-live & hand-over accepted         :milestone, m_go,      2025-08-29, 0d

```

---

### blocker\_strategy

---

#### mitigation

<details>
<summary>Top risks & mitigations</summary>

---

| Blocker            | Mitigation                                                  |
| ------------------ | ----------------------------------------------------------- |
| AWS quotas         | Raise limits Sprint 1 Week 1; nightly CI quota checks       |
| Security approvals | Evidence staged continuously; Gate #1 & Gate #2 checkpoints |
| DR readiness       | DR docs Sprint 3; fail-over drill milestone                 |
| Cost overrun       | Cost alerts Sprint 3; optimisation review                   |
| Rollback readiness | Simulation in Sprint 4 before cut-over                      |
| Skill gaps         | Pair-programming; daily 30 m tech huddle                    |

---

</details>

---

### buffer\_pto

---

<details>
<summary>Incident & PTO handling</summary>

---

* 20 % sprint buffer covers incidents, re-work, PTO
* Each engineer may take up to **4 PTO days** across project; prior notice needed
* Unused buffer converts to tech-debt resolution

---

</details>

---

## Technical stakeholder communication

---

### Audience Groups

<details>
<summary>Key stakeholder audiences and their information needs</summary>

---

- **Audience summary table**  
  | Audience | Typical Roles | Key Focus Areas | Information They Need from Infra Updates |
  |----------|---------------|-----------------|------------------------------------------|
  | **Executive Leadership** | CEO, CIO, CTO | Strategic alignment, risk exposure, ROI | • Confidence that the platform supports business OKRs <br>• High-impact risks & mitigation plans <br>• Budget adherence vs. forecast <br>• Launch-date confidence levels |
  | **Product & Analytics Owners** | Heads of Data Products, Analytics Managers | Delivery dates, feature readiness | • Milestone completion vs. roadmap <br>• Any blockers that could delay analytics/ML use-cases <br>• Environment availability for testing |
  | **Finance & Procurement** | CFO, Finance Controllers | Cost control, CapEx/OpEx split | • Actual vs. budgeted cloud spend <br>• Forecasted run-rate after go-live <br>• Savings realised from optimisation tasks |
  | **Security & Compliance Stakeholders** | CISO, Audit, Risk Managers | Regulatory compliance, security posture | • Status of encryption, IAM, audit logging <br>• Pen-test / DR drill outcomes <br>• Open compliance actions & due dates |
  | **Operations Leadership** | Head of SRE, IT Ops Manager | Operability, support readiness | • Monitoring/alerting coverage <br>• Run-book & hand-over progress <br>• Mean-time-to-recover targets vs. test results |
  | **Customer-Facing Account Teams (Optional)** | Customer Success, Sales Engineering | External commitments, client confidence | • High-level timelines they can share with clients <br>• Any service-level guarantees or changes |

---

</details>

---

### Communication Artefacts
<details>
<summary>Core mechanisms used to communicate infrastructure status and progress</summary>

---

- **Communication artefact table**  
  | Mechanism / Artefact | Primary Audience(s) | Delivery Channel & Frequency | Typical Contents | Why It Works |
  |----------------------|---------------------|------------------------------|------------------|--------------|
  | **Live KPI Dashboard** | Execs, Ops, Product | Web link (Grafana, Power BI) • `24 × 7` access | - Platform readiness `%`<br>- Uptime & latency SLIs<br>- Cost burn-down vs. budget<br>- Risk heat-map widgets | Puts always-current metrics one click away, reducing ad-hoc status pings. |
  | **Milestone Checklist (Gantt snapshot)** | Product, Execs | Posted in Confluence & sprint demo decks • Updated twice weekly | Colour-coded bars for each work-stream, milestone flags, `%` complete | Visual timeline instantly shows “on track / at risk” without deep tech. |
  | **Weekly Status Email** | Execs, Finance, Security | Plain-text + table • Friday COB | 🚦 `RAG` summary<br>• Completed this week<br>• Planned next week<br>• Top `3` risks/blockers<br>• Spend to date | Low-friction, searchable, can be forwarded by busy VPs. |
  | **Sprint Review Demo (`15 min` video / live)** | Product, Ops, Security | Recorded Teams meeting • end of each sprint | Live walk-through of new infra components, monitoring views, automation scripts | Concrete proof of progress; captures `Q&A` for backlog. |
  | **Monthly Executive Slide Deck** | C-Suite, Board liaison | `PDF` via email & `15 min` verbal readout | • Milestone variance chart<br>• ROI and cost forecast delta<br>• High/critical risks & mitigations<br>• Next-month decisions needed | Executive-level synthesis; keeps infra visible at the strategy table. |
  | **Risk & Decision Log** | Execs, Security, Finance | Shared sheet • real-time | • Date raised / owner / impact<br>• Decision deadline & outcome | Transparent audit trail; links each risk to a mitigation cost. |
  | **Cost Optimisation Tracker** | Finance, Ops | Live spreadsheet + monthly summary | • AWS service-level spend<br>• Savings plans & `RI` coverage<br>• Open optimisation tickets | Aligns engineering tweaks with finance goals; shows realised `$` savings. |

---

</details>

---


### Communication Best Practices
<details>
<summary>Guidelines for expressing infrastructure progress in business-relevant terms</summary>

---
#### Link to Business Objectives
- Start every update with how the infrastructure milestone advances a specific business OKR (`revenue growth`, `customer retention`, `regulatory compliance`)
- Replace service names with the capability they unlock: *“Multi-AZ FreeIPA enables <99.9%> user-auth availability for analytics teams”*
- Highlight time-to-market benefits: *“New AMI pipeline shaved <2 days> from data-model release cycle”*

---
#### Quantify Financial Impact
- Frame costs and savings in familiar finance language: *CapEx avoidance*, *run-rate delta*, *payback period*
- Convert technical optimisations into dollar figures: *“S3 lifecycle tiering saves `$12K` annually at current data growth”*
- Use variance charts (actual vs. budget) rather than raw CloudWatch metrics to keep focus on spend discipline

---
#### Surface Risk & Mitigation Clearly
- Present top risks in a heat-map or simple 🚦`RAG` status, paired with owner and mitigation ETA
- Translate failure modes into business consequences: *“If EFS throughput not achieved, nightly billing run slips by <4 hrs>, delaying invoices”*
- Keep mitigations action-oriented and cost-scoped so executives can weigh trade-offs quickly

---
#### Keep Language Plain & Jargon-Free
- Swap acronyms for plain terms on first mention (*Network Load Balancer → traffic balancer*)
- Limit deep-dive technical data to appendix links; main body should stay at “what / why / impact” level
- Use analogies sparingly but effectively: *“Think of FreeIPA as the office badge system—if it’s down, no one gets in”*

---
#### Focus on Forward Actions
- Conclude each update with **next two actions**, owner, and due date so leaders know decisions required
- Tie blockers to decision gates: *“Awaiting security sign-off—risks pushing go-live by `<5 days>` if not cleared by `<Aug 14>`”*
- Provide escalation path and threshold (e.g., cost over-run >`10%` or schedule slip >`1 week` triggers immediate exec briefing)

---
</details>

---


### Cadence & Escalation Rules
<details>
<summary>Structured timetable for stakeholder communications</summary>

---
#### Weekly & Sprint Rhythm
- **Weekly status email** sent every Friday COB ‒ includes 🚦 `RAG` summary, last/next tasks, top `3` risks, and cloud spend-to-date
- **Sprint review demo** recorded at the close of each `<2-week>` sprint ‒ live walk-through plus Q&A; link shared within `24 h`
- **Standing KPI dashboard check-in** during Monday stand-up ‒ project lead calls out any metric outside threshold

---
#### Monthly & Milestone Reviews
- **Monthly executive readout** on first Monday ‒ `<15 min>` deck + verbal sync covering milestone variance, ROI delta, critical risks
- **Finance cost report** on final working day ‒ spreadsheet of actual vs. budget, Savings Plan/RI coverage, upcoming optimisation actions
- **Milestone gate review** scheduled within `48 h` of milestone readiness ‒ sign-off meeting with Exec sponsor & Product owner

---
#### Trigger-Based Escalations
- **Risk severity ≥ “Red”** or schedule slip `>1 week` ‒ immediate Slack & email to Exec sponsor; 30-minute huddle within same business day
- **Cost variance `>10 %`** vs. plan ‒ escalated to Finance & CTO; require mitigation plan within `3` working days
- **Security/compliance events** (pen-test fail, DR drill issue) ‒ summary to CISO and Execs within `24 h`; remediation tracker opened
- **Change-freeze window** (starts `1 week` pre-go-live) ‒ broadcast via company-wide email and calendar reminder; any urgent change requires CTO approval

---
</details>

---

## Technical risk management

---

<details>
<summary>Infrastructure risk overview and mitigation strategies</summary>

---

* **FreeIPA directory outage halts authentication**

  * Deploy master and replica in separate AZs with `ipa-replica-manage` health probes.
  * ASG lifecycle hook auto-rebuilds replica in `<15 min>`.
  * Hardened AMI contains break-glass local users for emergency access.

* **EFS backup or restore failure causes data loss**

  * AWS Backup daily snapshots + weekly cross-Region copy.
  * Monthly automated restore drill with checksum validation; alerts if mismatch > `0.1 %`.

* **EFS throughput or IOPS saturation delays nightly pipelines**

  * CloudWatch alarm on `PercentIOLimit > 80 %` for `15 min`.
  * IaC switch to elastic throughput or FSx for Lustre when threshold breached.

* **Security breach via overly permissive SG or IAM policy**

  * Terraform-only SG definitions; AWS Config conformance packs block `0.0.0.0/0` ingress.
  * Nightly `tfsec`/`checkov` scans and IAM Access Analyzer auto-remediation.

* **Run-away cost from mis-configured Auto Scaling or NAT egress**

  * ASG `max_capacity` hard-limit `10` instances; NAT byte alarm at `80 %` baseline.
  * AWS Cost Anomaly Detection + weekly Finance report drives optimisation.

* **AWS quota exhaustion blocks scale-out or new environment deployment**

  * Sprint-1 quota uplift requests.
  * CI step queries `ServiceQuotas` API and fails if utilisation > `85 %`.

* **Terraform/Ansible drift introduces unknown state**

  * Nightly Terraform plan (dry-run) + weekly Ansible `check_mode`.
  * Drift resources auto-tagged `drift=true`; minor drift remediated via SSM Automation.

* **Unpatched OS or runtime CVE exposure**

  * Weekly SSM Patch Manager windows per environment with pre-patch snapshot.
  * Critical CVE triggers `AWS-PatchEmergency`; compliance KPI `<95 %` posts alert to `#ops-patching`.

* **Secrets exposure or stale credentials**

  * Secrets Manager `30-day` rotation with Lambda rotators; rotation failure alerts via EventBridge.
  * IAM access keys disabled; SSO-only access enforced.

* **Cross-AZ network failure impacts NLB routing**

  * Multi-AZ NLB listeners with health checks; ASG balanced across AZs.
  * Quarterly chaos test suspends an AZ to verify fail-over ≤ `<1 min>`.

---

</details>

---

## Team technical development

---

### Required skills, tools, and knowledge areas

<details>
<summary>Capabilities DevOps engineers must develop</summary>

---

* **AWS networking foundations** – `VPC`, CIDR design, subnetting, `NAT`, `IGW`, `VPC Endpoints`, `Network Load Balancer`, `Security Groups`, `NACL`.
* **Compute & scaling services** – `EC2` lifecycle, `Launch Template`, `Auto Scaling Group` policies, spot/mixed-instance strategy.
* **Storage services** – `Amazon EFS` performance modes, mount targets, `S3` lifecycle & encryption, `EBS` snapshot management.
* **Identity & access management** – `IAM` roles/policies/SCP, `AWS Identity Center` SSO, customer-managed `KMS`, `FreeIPA` LDAP/Kerberos administration.
* **Infrastructure as Code (IaC)** – Terraform `HCL`, module design, remote state, drift detection, linting with `tflint`, `tfsec`, `checkov`.
* **Configuration management** – Ansible roles/playbooks, dynamic inventory from Terraform, idempotency principles, `ansible-vault` secrets handling.
* **CI/CD tooling** – `GitHub Actions`, `CodeBuild`, `CodePipeline`, automated plan/apply gates, secret scanning.
* **Containerization** – `Docker`/`Podman`, image hardening, compose manifests, registry workflows in `ECR`.
* **Data-pipeline runtimes** – `Apache Spark`, `dbt`, `Apache Airflow` deployment patterns, resource tuning, log collection.
* **Monitoring & observability** – `CloudWatch` metrics/logs/alarms, `CloudWatch Agent`, OpenTelemetry custom metrics, dashboards, PagerDuty integration.
* **Security operations** – `AWS Config` conformance packs, `GuardDuty`, `Security Hub`, vulnerability management, Secrets Manager rotation.
* **Backup & disaster recovery** – `AWS Backup` vault policies, cross-Region copy, restore validation, RTO/RPO calculation.
* **Cost governance** – `AWS Cost Explorer`, budgets, `Cost Anomaly Detection`, Savings Plans/RI strategy.
* **Patch & compliance automation** – `SSM Patch Manager`, maintenance windows, compliance reporting.
* **Scripting & automation** – Python/Bash for tooling wrappers, `SSM Automation` documents, Lambda housekeeping functions.
* **Testing & validation** – `Terratest`, Molecule, load/chaos testing, synthetic alarms.
* **Documentation & knowledge sharing** – Markdown runbooks, Confluence pages, diagramming with `Mermaid`, audit evidence capture.

---

</details>

### Upskilling methods and activities

<details>
<summary>Recommended interventions for capability building</summary>

---

| Method                                   | Purpose & Scope                                                                                                                  | Cadence / Duration                        | Ownership & Resources                                                             |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- | --------------------------------------------------------------------------------- |
| **Structured pair-programming**          | Share Terraform & Ansible patterns; enforce coding standards; real-time knowledge transfer between senior and junior engineers.  | 2 × 2-hour sessions per sprint            | Lead DevOps pairs with rotating mentee; screen-share, joint PR.                   |
| **Hands-on AWS workshops (“lab days”)**  | Guided labs on VPC design, EFS tuning, IAM boundary policies, CloudWatch dashboards. Uses AWS Skill Builder or sandbox accounts. | Monthly ½-day; follow-up quiz             | Tech Lead curates lab scripts; training budget covers workshop credits.           |
| **Code & pipeline reviews**              | Enforce IaC quality (tflint, tfsec, checkov); surface best practices; discuss DR patterns, tagging standards.                    | Every pull-request; 24 h SLA              | All DevOps engineers; review checklist lives in repo `/docs/review.md`.           |
| **Lightning talks / brown-bag sessions** | 15-min demos on niche topics (SSM Automation tips, GuardDuty tuning); encourages peer teaching.                                  | Weekly after stand-up                     | Volunteer presenter; recordings stored in Confluence.                             |
| **Certification tracks**                 | AWS Solutions Architect-Associate, AWS DevOps Professional, HashiCorp Terraform Associate. Builds recognised baseline skills.    | Target ± 1 cert per engineer per 6 months | Individual study plan; exam vouchers in L\&D budget.                              |
| **Documentation sprints**                | Convert tribal knowledge into run-books, diagrams, and ADRs; sanity-check procedures during writing.                             | End of each sprint (½-day)                | Tech Writer + DevOps pair; PR merged to `/docs/runbooks/`.                        |
| **Mentor–mentee rotation**               | Formal 4-week cycles focusing on one skill gap (e.g., FreeIPA ops); ends with mentee delivering a demo.                          | Rolling; tracked by Skills Matrix         | Tech Lead assigns pairs; progress logged in Confluence page “Mentorship Tracker”. |
| **Chaos game days**                      | Simulate AZ outage, IAM mis-config, EFS throttle to practise incident response and DR procedures.                                | Quarterly                                 | SRE coach orchestrates; post-mortem captures lessons.                             |
| **Hackathons (“automation week”)**       | Tackle backlog of automation ideas—e.g., NAT cost analyser, drift auto-fix Lambda. Encourages innovation & cross-team bonding.   | Twice per year; 3 days                    | PM secures timebox; winning project added to roadmap.                             |
| **Community of practice (CoP)**          | Shared Slack channel + fortnightly meeting with data engineers & SREs; discuss patterns, tooling, RFCs.                          | Bi-weekly                                 | Rotating facilitator; agenda crowd-sourced.                                       |

---

</details>

### Tracking progress & ensuring knowledge transfer

<details>
<summary>Metrics, artefacts, and governance for continuous capability growth</summary>

---

| Tracking Mechanism              | What it Measures                                                         | Update Frequency                                    | Visibility                                                                      | Action on Gaps                                                            |
| ------------------------------- | ------------------------------------------------------------------------ | --------------------------------------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Skills matrix (Confluence)**  | Proficiency (0-3) across the 18 capability areas; certification status.  | Sprint-end                                          | Team page; snapshot surfaced in sprint review deck.                             | Tech Lead assigns mentor or workshop where score < 2 for critical skills. |
| **Mentorship tracker**          | Active mentor-mentee pairs, goal statement, progress notes, demo status. | Weekly                                              | Confluence “Mentorship Tracker”.                                                | Escalate to Tech Lead if goal demo not delivered by week 4.               |
| **Certification scoreboard**    | Exam attempts, passes, upcoming bookings.                                | Monthly                                             | Dashboard widget in KPI Grafana.                                                | PM flags funding/code-freeze if cert uptake < 75 % of plan.               |
| **Pull-request quality report** | Lint/security pass rate, review SLA adherence, rework count.             | Automated daily GitHub Action → Slack `#devops-ci`. | PR author addresses repeat findings; pair-programming scheduled if > 3 repeats. |                                                                           |
| **Workshop attendance log**     | RSVP vs. attendance, post-quiz scores.                                   | After each lab day                                  | Confluence table; summary in weekly status email.                               | Non-attendee must complete recording + quiz inside 1 week.                |
| **Lightning-talk archive**      | Indexed recordings & slide decks with tags (e.g., “SSM”, “GuardDuty”).   | Continuous                                          | Confluence—searchable by tag.                                                   | Quarterly audit deletes/outdates ≥ 12 m old; request refresh.             |
| **Run-book coverage KPI**       | % of critical procedures with up-to-date run-book (< 90 days old).       | Monthly                                             | Platform-Health dashboard.                                                      | Documentation sprint backlog item created when KPI < 95 %.                |
| **Chaos day post-mortems**      | Lessons learned, follow-up JIRA actions, knowledge gaps surfaced.        | Within 48 h of event                                | Confluence page linked to incident.                                             | Action items tracked to closure in sprint backlog.                        |

---

#### Governance workflow

1. **Sprint review** – Tech Lead presents skills-matrix diff and PR quality trends.
2. **Monthly L\&D sync** – PM + Tech Lead validate certification pipeline and budget burn.
3. **Quarterly capability review** (aligns with chaos game day retrospective) – update role expectations, refresh skills matrix rubric, adjust training backlog.
4. **Exit criteria for project hand-over** – ≥ 95 % run-book coverage, all engineers ≥ 2 proficiency in core domains (IaC, AWS networking, observability), at least one engineer holding AWS DevOps Pro cert.

---

</details>

