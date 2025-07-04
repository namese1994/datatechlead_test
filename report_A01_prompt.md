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

  * Create a detailed description of the system architecture after manual selection and adjustment of the architecture diagram. Use the following prompt with new GenAI conversation to avoid information overload caused by excessive or redundant details during the design selection process, attach the **ctx_doc_style.md** file so that GenAI complies with the format, and include the mermaid code for the revised architecture in the prompt.
    ```
    # ROLE
    You are an infrastructure architecture expert (Cloud/Data/Platform) and DevOps Tech Lead. 

    # Input

    [Main requirement]: Design the complete technical architecture and deployment plan for  an AWS Data Platform that serves as the foundation for a data engineering team

    Below are two fully revised system architecture diagrams (`mermaid`):
    \<paste 2 mermaid architecture diagram here>

    # Your Task


    **Describe the system architecture in detailed written form** to complement the two     provided architecture diagrams, aimed at two audiences:

    * **Business Stakeholders:** need to understand the big picture, value, and risks
    * **Technical (especially DevOps):** need clarity on each component for deployment and  operations

    # Output Requirements

    Write documentation to complete the following sections, as part of the [Main    requirement].

    1. **Architecture Overview:**

       * Describe the system in logical layers (presentation / data / computation /     storage / etc.), the role and interactions of each component.
    2. **Technical Analysis:**

       * Reason for choosing each component
       * Pros/cons
       * AWS services used (e.g., `EFS`, `EC2`, `IAM`, `Step Functions`, etc.)
    3. **Cost Estimation:**

       * Monthly cost estimate per component (use tiers: Free Tier, On-demand, Reserved as  applicable)
    4. **NFS Alternative Proposal:**

       * Suggest 1–2 alternatives if NFS is no longer suitable (scale, throughput, or   availability concerns)

    5. **Connection to Business Goals:**

       * Explain how the architecture supports organizational growth, stability, or scaling

    # Style & Format

    * Use markdown, follow `ctx_doc_style.md`
    * Clear sections (`##`, `###`, `####`)
    * Use bullet points instead of long paragraphs

    ```

---

</details>

### Deployment chronology

<details>
<summary>Create detailed deployment chronology and resource allocation</summary>

---

* Outline an **8‑week Deployment chronology** with weekly milestones, deliverables, and responsibility assignments for a four‑engineer team. Use this prompt for additional importance information before starting create Deployment chronology.
  ```
  # Role

  You are an experienced Project Manager with over 20 years of delivering Data Platform   projects in IT, especially on AWS.

  # Input

  I will provide a Infrastructure and Technial Detailed Diagram for an AWS Data Platform  deployment project:
  """
  <Copy Infrastructure and Technial Detailed Diagram>

  # Goal

  Design a complete 8-week Deployment chronology that meets technical requirements and  ensures effective coordination among all related teams.

  ---

  # Task

  Step 1 – Request for Additional Information:
  Before planning, list all the additional information you need me to provide to ensure  your handover plan is accurate and practical. Wait for answsering before go to step 2.

  Step 2 – Design Deployment chronology :
  Based on all input information, Create Deployment chronology, including:

  * 8-week schedule, broken down by week and clear sprints
  * Key technical milestones
  * Include weekly milestones, deliverables, and responsibility assignments for a four‑engineer team
  * 20–30% buffer time for incident handling or requirement changes
  * Apply Agile Scrum
  * Strategy to minimize task dependencies and prioritize handling blockers first
  * A mermaid chart as code to show timeline as roadmap, summary

  # Style & Format

  * Use markdown, follow `ctx_doc_style.md`
  * Clear sections (`##`, `###`, `####`)
  * Use bullet points instead of long paragraphs
  ```
  After that, provide the information that GenAI requests. Like below:

  ```
  1. Project Context

  * Current phase: initiation
  * Type of handover: between internal teams
  * Reason for handover: completion of data platform setup
  * Deadline: within 8 weeks

  2. Team & Resources

  * Current operating team: 4 DevOps engineers, average 1 year of experience
  * Receiving team: Data engineering team
  * Training time: arranged reasonably as required
  * Personnel availability: 5%-8% time off; no other commitments

  3. Environment & Configuration

  * Number of environments: Dev, Staging, Production
  * Existing data: none
  * Active workloads: none
  * Change management: platform team manages internally

  4. Documentation

  * Requirements: architecture documentation, runbook, operation manual, security documentation
  * Source code: IaC source code
  * Monitoring: dashboard, alert

  5. Operational Requirements

  * Backup & DR: storage backup only
  * Incident response & escalation process
  * Vendor dependency: none
  * Compliance: SOC2, GDPR, industry regulations

  ```

---

</details>

### Automation & IaC Prompts

<details>
<summary>Produce Infrastructure as Code and configuration management snippets</summary>

---

* Design Infrastructure as Code Strategy. Using this prompt and interact with GenAI to confirm or edit each step:
  ```
  
  # Role

  You are an Devops Engineer Expert with over 10 years of delivering Data Platform projects on AWS.

  # Input

  I will provide a Overal Architect of this project:
  """
  <Paste  Overal Architect, include nfrastructure and Technial Detailed Diagram and descriptions>
  """

  # Goal
  Infrastructure as Code Strategy for Fully Automated Deployments with Terraform and Ansible.

  ---

  # Task

  Create detail Infrastructure as Code Strategy for Fully Automated Deployments with Terraform and Ansible.

  # Instructions
  Follow the steps below. Only do one step per response. Wait for me to confirm or edit before moving on to the next step.

  Step 1:
  Based on the provided infrastructure diagram and technical detailed diagram, identify all components related to deploying a data platform on the AWS  platform. This includes but is not limited to: infrastructures, applications, container images for deploying applications, and all necessary components  for the data platform operation that are not mentioned in the diagrams.

  Step 2:
  Identify the complete list of components that will be deployed using Terraform, and those that will be deployed using Ansible. Present them as bullet   points
 
  Step 3:
  Define the deployment sequence by groups, carefully considering dependencies and access controls. 
  My recommendation is to separate the Terraform configurations into three parts: one dedicated to creating user-related accounts and IAM policies; another for deploying stable infrastructure components that rarely change but serve as dependencies for applications, such as networking; and a third for deploying components that frequently change during the development process.
  However, please adjust if there is a better solution.

  Step 4:
  Create a repository structure for Terraform and Ansible source code. This repository is exclusively for DevOps use. 

  Remmember: Use markdown, follow `ctx_doc_style.md`
  ```
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
