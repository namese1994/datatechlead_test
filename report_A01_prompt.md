## Prompt Catalog

---

### Technical Architecture Prompts

<details>
<summary>Generate comprehensive AWS Data Platform architecture content</summary>

---

* Draft a **high‑level architecture description** covering: EC2 user systems, EFS/NFS shared storage, IAM roles & policies, FreeIPA authentication, VPC & subnet layout, security groups, and network flow.
* Produce a **Mermaid diagram** (system context) that illustrates component interactions and data flow from user workstations to AWS services.
* Generate a **Terraform skeleton** for provisioning EC2 instances, EFS file system, IAM role structure, and FreeIPA deployment in AWS.
* List **performance considerations** (scaling, cost, latency) for each component with bullet justification.
* Summarize **integration points** between platform services and external corporate systems.

---

</details>

### Implementation Plan Prompts

<details>
<summary>Create detailed deployment chronology and resource allocation</summary>

---

* Outline an **8‑week implementation timeline** with weekly milestones, deliverables, and responsibility assignments for a four‑engineer team.
* Provide **step‑by‑step installation** commands (Terraform init/apply, Ansible playbook runs) for each milestone.
* Generate a **risk register** table (risk, impact, likelihood, mitigation) for the deployment plan.
* Produce **communication snippets** for weekly stakeholder updates (engineer‑friendly and executive‑friendly versions).
* Suggest **training activities** (workshops, pair‑programming, knowledge‑sharing sessions) aligned with each milestone.

---

</details>

### Automation & IaC Prompts

<details>
<summary>Produce Infrastructure as Code and configuration management snippets</summary>

---

* Generate **Terraform module templates** for reusable EC2 + EFS + IAM stacks.
* Create **Ansible playbook snippets** for FreeIPA installation, NFS mounting, and user provisioning.
* Draft **CI/CD pipeline YAML** (GitHub Actions) that validates Terraform, runs security scans, and deploys to AWS.
* Provide **parameterization guidance** (variables.tf, inventory.ini) for environment toggling (dev / staging / prod).
* Produce **README bullets** that explain how to run and troubleshoot each automation component.

---

</details>

### Access Control & Security Prompts

<details>
<summary>Design IAM, authentication flow, and security hardening content</summary>

---

* Generate **IAM role and policy bullet lists** mapping platform roles to least‑privilege permissions.
* Craft **authentication flow description** integrating FreeIPA with AWS via IAM Identity Provider.
* Produce **Mermaid sequence diagram** illustrating user login, token exchange, and resource access.
* List **security group rules** (inbound/outbound) for EC2, FreeIPA, and NFS nodes with rationale.
* Suggest **compliance checks** (CIS benchmarks, AWS Config rules) and automated remediation actions.

---

</details>

### Monitoring & Operations Prompts

<details>
<summary>Create operational procedures and observability guidance</summary>

---

* Draft **CloudWatch dashboard bullet points** (CPU, memory, disk I/O, FreeIPA auth failures).
* Generate **alert threshold suggestions** with ticket escalation workflow.
* Provide **backup & restore procedure bullets** for EFS data and FreeIPA directory.
* Produce **daily/weekly maintenance checklist** for platform reliability tasks.
* Suggest **post‑incident review template bullets** (root cause, corrective actions, lessons learned).

---

</details>

---

## Prompt Workflow Guide

---

### Step‑by‑Step Prompt Usage

<details>
<summary>End‑to‑end GenAI workflow for rapid report assembly</summary>

---

* **Stage 1 – Outline:** Use architecture prompts to generate raw content, copy into corresponding report sections.
* **Stage 2 – Refine:** Iterate on content prompts to add depth or clarify stakeholder language as needed.
* **Stage 3 – Automate:** Run IaC prompts to obtain code blocks; paste into report with correct indentation.
* **Stage 4 – Validate:** Employ security and operations prompts to cross‑check completeness and risk coverage.
* **Stage 5 – Finalize:** Generate executive summary prompts to craft concise stakeholder communication bullets.

---

</details>

---

## Quality Checklist

---

### Prompt File Review

<details>
<summary>Ensure compliance with ctx_doc_style.md and content completeness</summary>

---

* [ ] YAML front matter present with snake\_case title.
* [ ] Each subsection contains exactly one details block.
* [ ] No numbered lists; all bullets follow one‑concept rule.
* [ ] All placeholders written in clear, descriptive bullets for easy replacement.
* [ ] Technical symbols (e.g., `<5%`, `$10K`) wrapped in backticks if present.
* [ ] Block elements indented 2 spaces under parent bullet where required.
* [ ] Prompt coverage spans architecture, implementation, automation, security, operations, and leadership communication.

---

</details>
