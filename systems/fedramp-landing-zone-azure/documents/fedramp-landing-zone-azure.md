<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# FedRAMP Landing Zone on Azure

**Project Link:** [View Project](https://learn.nextwork.org/projects/b9933a6a-1afe-46ed-bf22-c7119f6fcbed)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_r3n8w5q2)

## Why FedRAMP-Ready Infrastructure Matters

This project establishes a FedRAMP-ready landing zone on Azure using Terraform, Azure Policy, and Prowler.

The goal is to build a secure baseline aligned to NIST 800-53 controls, where compliance is enforced through infrastructure, not documentation.

## Setting Up the Compliance Toolkit

### Installing Prowler and verifying Azure CLI authentication

This phase prepares the environment for compliance validation and deployment.

Prowler is installed to assess security posture, while Azure CLI authentication ensures Terraform and Prowler can operate against the subscription. Terraform is initialized to download the azurerm provider and establish the working environment.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_r4t7xp9a)

### Scaffolding the Terraform project with Cursor Composer 2

The project structure defines how infrastructure and policies are managed.

Composer generates core Terraform files and a policies directory, separating infrastructure logic from compliance rules. This establishes a repeatable pattern for managing landing zone components.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_j6n2qf4v)

### Initializing the azurerm provider

Terraform initialization downloads provider dependencies and creates the working state.

This step ensures the configuration can interact with Azure resources and maintain state consistency during deployment.

## Building the Azure Management Group Hierarchy

### Defining FedRAMP-Lab, Platform, and Workloads groups with Azure Policy assignments

Defining FedRAMP-Lab, Platform, and Workloads groups with Azure Policy assignments

The management group hierarchy defines governance boundaries.

Policies assigned at the root level propagate to all child groups and subscriptions, ensuring consistent enforcement of compliance controls across the environment.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_r3w8n5q1)

### Enforcing CM-7 and SC-7 controls through policy guardrails

Policies are used to restrict configurations and enforce security baselines.

Applying policies at higher scopes ensures that all resources inherit required controls, reducing configuration drift and enforcing least functionality and network boundary protections.

## Enabling Centralized Audit Logging

### Provisioning the audit log Storage Account (NIST AU-4)

A centralized storage account is used to capture audit logs across the environment.

This ensures that all control-plane activity is recorded and retained, supporting compliance and audit requirements.

### Configuring Activity Log diagnostic settings for AU-2 and AU-3

Diagnostic settings define which events are captured and how they are stored.

Administrative, security, policy, and alert events are routed to a centralized location, ensuring visibility into who performed actions and when they occurred.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_yx5c2jaf)

### Deploying the full landing zone with Terraform apply

Deployment provisions the full landing zone across management groups, policies, and logging infrastructure.

This confirms that infrastructure and compliance controls are implemented together as a single system.

## Running the Prowler CIS Azure Benchmark Scan

### Interpreting pass/fail results across 160+ security checks

Prowler evaluates the environment against the CIS Azure Benchmark.

The results highlight gaps in areas such as networking, compute, and security controls, which are expected in a baseline lab environment.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_h2c9f7a3)

## Mapping Findings to NIST 800-53 and Drafting the SSP

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_qr7v2bkf)

### Generating the CIS-to-NIST 800-53 crosswalk with control-family coverage percentages

The crosswalk measures how well implemented controls align with NIST requirements.

Coverage percentages highlight strong areas such as access control and weak areas such as system integrity, guiding remediation priorities.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_bn5p8rge)

### Refining SSP narratives for non-technical reviewers

The SSP translates technical controls into clear explanations.

Activity logging is framed as accountability, showing what actions occurred, when they happened, and who initiated them. This aligns technical implementation with audit expectations.

## Secret Mission: Enforcing Encryption at Rest (SC-28)

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/b9933a6a-1afe-46ed-bf22-c7119f6fcbed_kw7m3xpq)

### Adding a third Azure Policy, re-running Prowler, and measuring raised control coverage

An additional policy enforces encryption at rest for storage resources.

Re-running Prowler shows higher control coverage, confirming that policy enforcement directly impacts compliance posture.

## Wrapping Up: What Was Built and What Comes Next

### Resources created and cost considerations

This project builds a compliant Azure landing zone with governance, logging, and policy enforcement.

The key outcome is a system where compliance is embedded into infrastructure rather than applied after deployment.

This project took about 55 minutes. The main challenge was interpreting Prowler results and mapping them accurately to NIST controls.

### Cleanup and next steps for federal cloud authorization work

I did this project today to learn how to build a secure Azure landing zone using Terraform, Prowler, and Azure Policy, automating compliance checks and drafting security plans. Another skill I want to learn is cloud security architecture and designing compliant cloud environments that survive control failures and audit reviews.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/b9933a6a-1afe-46ed-bf22-c7119f6fcbed)*
