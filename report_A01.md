## Technical Architecture

---

### System Overview

<details>
<summary>Comprehensive description of the AWS Data Platform foundation</summary>

---


* **Purpose**: <em>Describe the high‑level objectives of the data platform (analytics, ML, BI, etc.).</em>
* **High‑level Architecture Diagram**: Diagram summarizing core services.
  
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
## NFS Alternative Proposal
---

### When EFS Caps Are Hit
<details>
<summary>Higher throughput, bigger dataset, or global access needs</summary>

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

## Implementation Plan & Timeline

---

### Deployment Chronology

<details>
<summary>Step‑by‑step technical timeline for 8‑week rollout</summary>

---

* **Week 1**: Requirements confirmation, VPC scaffolding, Terraform repo bootstrap.
* **Week 2**: IAM baseline roles/policies, EC2 base AMIs, FreeIPA proof‑of‑concept.
* **Week 3**: EFS creation & performance testing, security groups hardened.
* **Week 4**: Terraform modules finalized, code review & CI/CD integration.
* **Week 5**: Ansible playbooks for OS hardening, application baseline setup.
* **Week 6**: Monitoring stack deployment, CloudWatch alarms, logging sinks.
* **Week 7**: Load testing, failover drills, security audit, cost optimization review.
* **Week 8**: Final production cut‑over, documentation handoff, stakeholder sign‑off.

---

</details>

### Milestone & Deliverable Table

<details>
<summary>Key checkpoints, owners, and success criteria</summary>

---

* `TODO` insert Markdown table listing milestone, date, owner, acceptance criteria.
* Placeholder bullet for gating conditions and exit criteria per milestone.
* Placeholder bullet for dependency tracking across tasks.

---

</details>

---

## Infrastructure as Code (Terraform)

---

### Repository Structure

<details>
<summary>Folder layout, module decomposition, and coding conventions</summary>

---

* `/live` vs `/modules` directory pattern.
* Naming conventions, backend state configuration, remote state locking.
* CI pipeline bullets: `terraform fmt`, `terraform validate`, `tflint`, OPA policies.

---

</details>

### Core Modules

<details>
<summary>Reusable Terraform modules for platform components</summary>

---

#### vpc\_module

* Inputs: CIDR, subnets, tags.
* Outputs: VPC ID, subnet IDs.

---

#### ec2\_module

* Inputs: AMI ID, instance type, user‑data template.
* Outputs: Instance ID, private IP.

---

#### efs\_module

* Inputs: performance mode, throughput mode, lifecycle policy.
* Outputs: File system ID, mount targets.

---

#### iam\_module

* Inputs: role name, policy JSON, path.
* Outputs: Role ARN.

---

</details>

### Sample Snippets

<details>
<summary>Placeholder code blocks demonstrating key module usage</summary>

---

* `TODO` add fenced `hcl` code block for EC2 & EFS resources.
* `TODO` add example of IAM role with trust policy.

---

</details>

---

## Configuration Management (Ansible)

---

### Playbook Strategy

<details>
<summary>Role‑based approach for provisioning and configuration</summary>

---

* Bullet outline of site.yml, role dependencies, inventory design.
* Placeholder bullet for idempotency checks and rerun safety.
* Placeholder bullet for Ansible‑pull vs Ansible‑tower discussion.

---

</details>

### Key Roles

<details>
<summary>Core Ansible roles mapped to platform components</summary>

---

#### role\_freeipa\_server

* Tasks: install packages, configure replication, open firewall ports.

---

#### role\_freeipa\_client

* Tasks: enroll EC2 instances, configure SSSD, test authentication.

---

#### role\_nfs\_client

* Tasks: install nfs‑utils, mount EFS via EFS mount helper, set fstab.

---

</details>

---

## Access Control & Security

---

### IAM & Authentication Flow

<details>
<summary>End‑to‑end access control architecture</summary>

---

* Bullet placeholders for SSO integration, identity federation, user onboarding.
* Diagram placeholder for trust relationships and auth sequence.
* Placeholder for secrets management via AWS Secrets Manager / SSM Parameter Store.

---

</details>

### Network Security

<details>
<summary>Defense‑in‑depth approach and traffic segmentation</summary>

---

* Bullet placeholders for security groups, NACLs, flow logs review.
* Placeholder for encryption in transit (TLS) and at rest (KMS keys).

---

</details>

---

## Operational Procedures

---

### Monitoring & Alerting

<details>
<summary>System health visibility and proactive incident response</summary>

---

* Bullet placeholders for CloudWatch dashboards, alarm thresholds.
* Placeholder for incident escalation runbook reference.
* Placeholder for log retention policies.

---

</details>

### Backup & Recovery

<details>
<summary>Data protection strategy and disaster recovery preparedness</summary>

---

* Bullet placeholders for AWS Backup plans, point‑in‑time restores.
* Placeholder for recovery time (RTO) and recovery point (RPO) objectives.

---

</details>

---

## Leadership & Team Coordination

---

### Team Structure & Roles

<details>
<summary>Guidance for coordinating a 3‑4 engineer implementation team</summary>

---

* Bullet placeholders for role matrix: Tech Lead, DevOps Engineer, Security Engineer, QA.
* Placeholder for daily stand‑up agenda and sprint cadence.

---

</details>

### Upskilling & Mentorship

<details>
<summary>Plan for developing engineer capabilities during project</summary>

---

* Bullet placeholders for pair programming sessions, lunch‑and‑learns.
* Placeholder for certification goals (AWS Solutions Architect – Associate).

---

</details>

---

## Stakeholder Communication

---

### Business Alignment

<details>
<summary>Tech‑to‑business translation and executive updates</summary>

---

* Bullet placeholders for KPI dashboard summary, cost visibility.
* Placeholder for monthly executive readout slide deck.

---

</details>

### Progress Reporting

<details>
<summary>Regular update mechanisms and communication cadence</summary>

---

* Bullet placeholders for weekly status email template.
* Placeholder for Jira/Roadmap snapshot links.

---

</details>

---

## Risk Management & Mitigation

---

### Risk Register

<details>
<summary>Identification, impact analysis, and mitigation actions</summary>

---

* Bullet placeholders for security risks, cost overrun, skills gap.
* Placeholder for residual risk acceptance criteria.

---

</details>

### Contingency Strategies

<details>
<summary>Fallback procedures for critical failure scenarios</summary>

---

* Bullet placeholders for manual failover steps.
* Placeholder for alternative provider strategy.

---

</details>

---

## Training & Knowledge Transfer

---

### Documentation Handoff

<details>
<summary>Process for transferring system knowledge to support teams</summary>

---

* Bullet placeholders for runbook repository, FAQ wiki pages.
* Placeholder for recorded walkthrough sessions.

---

</details>

### Ongoing Education

<details>
<summary>Post‑deployment learning path for engineers</summary>

---

* Bullet placeholders for future AWS service deep dives.
* Placeholder for cross‑training with data analytics team.

---

</details>

---

## Terminology & Standards

---

### Glossary

<details>
<summary>Canonical definitions for technical terms used in this document</summary>

---

* `EC2`: <em>Elastic Compute Cloud</em> – Placeholder definition.
* `EFS`: <em>Elastic File System</em> – Placeholder definition.
* Add more acronym bullets as needed.

---

</details>

### Documentation Style Compliance

<details>
<summary>Reminder of mandatory ctx_doc_style rules</summary>

---

* All content must remain bullet‑only, no numbered lists.
* All block elements two‑space indented.
* No `---` separators between ### sections.

---

</details>

---

## Quality Checklist (Pre‑Submission)

---

### Structure Review

<details>
<summary>Template checkboxes for structural compliance</summary>

---

* [ ] YAML front matter present with snake\_case title.
* [ ] Every ### subsection contains exactly one details block.
* [ ] Main ## sections separated by `---`.
* [ ] Details blocks start and end with `---` separators.
* [ ] Subsubsections separated by `---` inside details blocks.

---

</details>

### Content Review

<details>
<summary>Template checkboxes for content completeness</summary>

---

* [ ] All required technical deliverables addressed.
* [ ] Leadership coordination narrative included.
* [ ] Diagrams and code snippets inserted where marked `TODO`.
* [ ] Terminology section updated with project‑specific terms.

---

</details>
