## Prompt Catalog

---

### Technical Architecture Prompts

<details>
<summary>Generate comprehensive AWS Data Platform architecture content</summary>

---

* Draft a **high‑level architecture description** covering: EC2 user systems, EFS/NFS shared storage, FreeIPA authentication, VPC & subnet layout, security groups, and network flow. 

  * GenAI proposes infrastructure options to support brainstorming. Using this prompt:
    ```
    # ROLE
    You are a senior cloud architect and data platform lead.  
    
    # CONTEXT
    I need to design a robust AWS Data Platform Foundation for an enterprise usecasethat requires:
    - User Linux systems - EC2 instance architecture, sizing, configurationspecifications
    - NFS storage architecture - shared file system design, capacity planningperformance considerations
    - FreeIPA integration design - authentication system architecture, directoryservice integration
    
    Other non-functional requirement:
    - Ensure high availability, scalability
    - Ensure disaster recovery capability
    
    # Your tasks:
    
    1. Propose three infrastructure options that satisfy the above requirements.
    2. For each architecture, provide:
        - A high-level diagram in Mermaid markdown format (for easy visualization)
        - Key AWS services and components used
        - Access control and authentication flow
        - Pros and cons (including security, scalability, operational complexity, cost)
        - A brief summary of when each option is preferable
    3. Present the answer in markdown, clearly separating each option, usingbulletpoints and detail blocks.
    4. Keep the output concise but comprehensive, suitable for a technical report.
    
    # Extra requirements:
    - Start each draft with a bold headline and a short summary.
    - All code and diagrams must be in markdown-compatible format.
    ```
  * After the options are proposed, the final decision will be made manually. Then, ask GenAI to add enhancements and create a Core Infrastructure Diagram by continuing the conversation above with GenAI. Using this prompt:
    ```
    Follow up above.

    This is my infrastructure choice for above task:
    
    - Network
      - VPC
      - Private Subnet for all workloads
      - Load Balancers: 
        - Network LB for FreeIPA and workers
    -  Workload
      - NFS with EFS
      - workers using EC2 and Auto Scaling Groups, connect EFS and FreeIPA. Assumethere is no web app, only data processors.
      - FreeIPA with master/slave on difference AZ, for authentication.
    - Security
      - Session Manager
      - FreeIPA
      - Secrets Manager
    - Monitoring
      - Use managed monitoring system with Cloud Watch 
      
    # Your tasks:
    Create Architecture diagrams using mermaid code- detailed system integration,component relationships, infrastructure topology
    
    # Instruction
    
    ## Follow below steps
    1. Enumerate any additional components not previously listed, and explain theirrelationships within the complete data platform.
    
    2. Review the architecture for areas that could be improved, and then implementthose improvements.
    
    3. Design architecture diagrams, focus Core infrastructure, using Mermaid,placing the code in code blocks. Make sure the diagrams are easy to understandfor both technical teams and business stakeholders.

    ```
  * Continue the conversation, develop the Core Infrastructure Diagram into a Technical Detailed version that is detailed and user-friendly for the technical team. Using this prompt:
    ```
    Follow up.
    Using the Core Infrastructure Diagram above, develop a Technical Detailed version covering detailed system integration, component relationships, and infrastructure.
    This version is optimized for the technical team (especially DevOps), and should be clear and easy to follow for infrastructure deployment.
    Present the solution in a single diagram.
    ```


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
