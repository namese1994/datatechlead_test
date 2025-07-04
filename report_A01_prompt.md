## Prompt Catalog

---

### Technical Architecture

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

### Automation & IaC

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

### Access Control & Security

<details>
<summary>Design IAM, authentication flow, and security hardening content</summary>

---

* Create **Access control architecture** by continuing the above conversation, leveraging the context of the previous conversation (Requires using GenAI models with contexts longer than 32K tokens to avoid truncation)
  ```
  Follow up.

  # Role
  You are a senior AWS solutions architect and technical writer.

  # Task
  Please generate the "Access control architecture" documentation section for our AWS Data Platform.

  # Instrution
  Follow these requirements:

  * Focus: IAM roles, IAM policies, security groups, authentication flow design.
  * Depth:

    * Briefly describe each key IAM role (examples: service roles, instance profiles, admin vs. engineer roles, CI/CD pipeline roles).
    * Outline principal IAM policies (least-privilege, access boundaries, rotation).
    * Use RACI table with following roles:  Project Manager ; Tech Lead ; DevOps Engineer ; Data Engineer ; Security Engineer ; Auditor
    * Summarize security group strategy (segmentation, tiering).
    * Bullet the authentication flow from user access through service authentication to backend resources.
    * Reference AWS native tools: Secrets Manager, Session Manager, FreeIPA, etc.

  * Constraints:

    * No prose paragraphs, only bullet points.
    * Avoid vendor marketing language—focus on architectural choices and rationale.
    * Follow the documentation style and bullet structure as shown in the system docs above.
  ```

---

</details>

### System integration documentation
<details>
<summary>Design System integration documentation</summary>

* Create System integration documentation by using this prompt by continuing the above conversation. Interact with GenAI to confirm or refine each step
  
  ```
  Follow up above conversation.

  # Role

  You are a Senior DevOps Engineer and AWS Data Platform Architect with deep expertise in documenting enterprise system integration patterns.

  # Input

  All information as discussed. 

  # Goal

  Generate a comprehensive "System integration documentation" section describing how system components interconnect, data flows, and the overall network architecture, formatted to be included in technical architecture documentation.

  ---

  # Task

  Create the "System integration documentation" section, covering component connectivity, data flows, and network architecture, for inclusion in system docs.

  # Instructions
  Follow these steps, one step at a time. Wait for my confirmation or edit before moving to the next.

  Step 1:  
  From the provided architecture and diagrams, identify all core system components, both application and infrastructure, involved in data flow and system integration (compute nodes, storage, data services, network boundaries, etc.).

  Step 2:  
  Map out, in bullet points, the connectivity relationships between components—specifying which components connect to which, using what protocols, ports, or access patterns. Include both internal (within VPC/subnets) and external (user access, internet, 3rd party) flows.

  Step 3:  
  Describe end-to-end data flows through the system: from data ingestion, processing, storage, to consumption. Summarize network segmentation and security zones (such as private/public subnets, load balancers, endpoint gateways).

  Step 4:  
  Output the final "System integration documentation" section in the following style:
  - Wrap all content in a `<details>` block with a `<summary>` titled "System integration: connectivity, data flows, and network architecture".
  - For diagrams, insert a markdown code block as a placeholder for architecture or network flow mermaid diagrams 
  - Focus on clear, concise technical bullets. No marketing language or unnecessary prose.

  Remember: Use markdown, and strictly follow the documentation ctx_doc_style.md.
  ```

---

</details>

---

### Operational Procedures

<details>
<summary>Design Operational Procedures</summary>

*  Create System integration documentation by using this prompt by continuing the above conversation. Interact with GenAI to confirm or refine each step
  ```
  
  Follow up

  # Role

  You are a Senior DevOps Engineer and AWS Platform Operations Architect specializing in documenting operational procedures for cloud-based data platforms.


  # Goal

  Generate a detailed "Operational procedures" documentation section (for technical architecture docs) covering monitoring setup, backup procedures, and maintenance workflows.

  ---

  # Task

  Create the "Operational procedures" section as a series of details blocks, using bullet points and strictly following the documentation ctx_doc_style.  
  At each step, produce a ready-to-use docs section for that aspect.

  # Instructions

  Follow these steps. Wait for my confirmation or edits before moving to the next step.

  **Step 1:**  
  Document, all system components and AWS services involved in monitoring, backup, and operational maintenance. Use bullet points.

  **Step 2:**  
  Document, the monitoring setup:  
  - Key metrics, alarms, and dashboards  
  - Logging flows (which components generate which logs, where logs are sent/retained)  
  - Alerting and escalation mechanisms

  **Step 3:**  
  Document, the backup procedures:  
  - What resources are backed up, using which tools/services  
  - Retention policies and backup schedules  
  - Restore/test/validation practices

  **Step 4:**  
  Document, standard maintenance workflows:  
  - Patch management (how, when, tools used)  
  - Health checks, drift detection, incident response  
  - Routine operational tasks and automation

  Remember: Use markdown, and strictly follow the documentation ctx_doc_style.md. 

  ```
---

### 