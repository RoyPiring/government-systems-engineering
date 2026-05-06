<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Multi-Cloud GovRAMP Modernization Lab

**Project Link:** [View Project](https://learn.nextwork.org/projects/3c745f73-b8de-4458-bbf6-6e667f1b9e49)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_r7tn2xfg)

## The Mission: Modernizing State Government Infrastructure

### Project scope and objectives

## Building the Lab Environment

### Installing WSL2, Cursor IDE, and CLI tools

The environment setup establishes the execution layer for all subsequent steps.

WSL2 provides a Linux-compatible runtime for consistent tooling behavior. Cursor IDE acts as the development interface for authoring infrastructure and documentation, while CLI tools enable direct interaction with cloud providers and Kubernetes clusters. Authentication is configured across AWS, Azure, and GCP, and billing alerts are set to prevent uncontrolled cost growth during experimentation. This step ensures that all tools operate within a unified and repeatable environment.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_p7w2n4q8)

### Installing OpenTofu, k3s, and Velero

The platform uses OpenTofu for infrastructure provisioning, k3s for Kubernetes orchestration, and Velero for backup and recovery.

k3s is selected instead of managed Kubernetes services to reduce cost and increase control over the environment. It maintains compatibility with standard Kubernetes APIs while running on lightweight infrastructure, making it suitable for lab-scale replication of on-prem conditions. Velero introduces backup and restore capabilities that will later support disaster recovery validation across clusters.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_m5j8c2y6)

### Configuring cloud accounts and cost guards

Cloud accounts are configured with governance in mind.

Each provider is authenticated, and cost controls are enforced through billing alerts and scoped resource usage. The system is documented using a structured README that includes system boundaries, tool versions, and topology diagrams. This ensures that every component of the lab is traceable and reproducible, which is required for compliance-style environments.

## Provisioning the Multi-Cloud System Boundary with OpenTofu

### Writing the multi-provider IaC configuration

The infrastructure layer defines the system boundary across three cloud providers and one local cluster.

Separate modules are created for AWS, Azure, and GCP to reflect differences in resource types, networking models, and provisioning logic. This avoids overloading a single abstraction with provider-specific conditionals and ensures that each environment can be managed independently. The result is a set of loosely coupled modules that together form a unified system boundary.

### Planning and applying infrastructure across three clouds

Infrastructure is provisioned through coordinated OpenTofu execution.

Each provider module defines its own resources, including compute instances, networking rules, and startup configurations. The apply process creates three cloud-based k3s clusters that complement the local on-prem cluster. This produces a four-cluster system that can be accessed through a single kubeconfig, enabling centralized control and validation.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_xn5v9kqd)

### Verifying all four clusters are healthy

Cluster validation confirms that the system boundary is operational.

Each cluster exposes a functioning Kubernetes API, and connectivity is verified through kubectl commands. The environment represents a controlled perimeter where workloads can be deployed, monitored, and recovered. This aligns with GovRAMP expectations for defining scope, controls, and evidence boundaries in a system authorization context.

## Deploying GitOps Control with Argo CD

### Installing Argo CD on the hub cluster

This step establishes centralized deployment control across all clusters.

Argo CD is installed on the on-prem cluster, which acts as the hub. This cluster becomes the control plane for application delivery, allowing all workloads to be managed from a single location. The user interface is exposed, and initial validation confirms that Argo CD can interact with the local Kubernetes API.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_r7nw4p2x)

### Registering all three cloud clusters

The system expands control from the hub to all remote clusters.

Cloud clusters are registered as external targets using their kubeconfigs. The on-prem cluster does not require registration because it is already the host environment. This distinction ensures that only external endpoints are explicitly added, while the hub cluster remains implicitly available.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_t5bq9w3y)

### Deploying the benefits portal with App of Apps pattern

Application deployment is standardized through a GitOps pattern.

A parent application defines the full set of child applications that should exist across clusters. This pattern ensures that the desired state is declared in Git and enforced by Argo CD. The benefits portal is deployed consistently across all clusters, with configuration controlled through versioned manifests.

This approach satisfies configuration baseline requirements by ensuring that every deployment is traceable, repeatable, and auditable through Git history and Argo CD logs.

## Implementing COOP Disaster Recovery with Velero

### Installing Velero and configuring shared backup storage

This step introduces disaster recovery capabilities across clusters.

Velero is installed on selected clusters, and a shared object storage bucket is configured as the backup target. This creates a single source of truth for backups, allowing data to be restored across environments without requiring manual transfer.

The configuration ensures that both metadata and application state are captured, forming the basis for recovery operations.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_v4n7x2q9)

### Running the cross-cluster DR drill

The system validates recovery through a simulated failure.

A primary cluster is assumed lost, and recovery is executed on an alternate cluster using backups stored in the shared bucket. Because both clusters reference the same storage, the restore process can access identical backup artifacts without replication steps.

This confirms that the system can reconstitute workloads in a different environment, which is a core requirement for continuity of operations.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_t5j9k3n8)

### Calculating RTO and RPO metrics

Recovery performance is measured using defined metrics.

Recovery Time Objective is calculated as the time between failure declaration and service restoration, while Recovery Point Objective measures the data gap between the last backup and the failure event. The recorded values fall well within the defined thresholds, confirming that the system meets lab-level recovery expectations.

### Validating against government benchmarks

The system is evaluated against predefined targets.

Measured recovery times and data loss windows are significantly lower than the acceptable limits for moderate-impact systems. This demonstrates that the recovery process is not only functional but also efficient relative to expected standards.

## Authoring the GovRAMP Crosswalk and Migration Wave Plan

### Mapping 60 GovRAMP Moderate controls to artifacts

This step translates technical implementation into compliance evidence.

Each control is mapped to a corresponding artifact within the system, including configuration files, deployment manifests, and recovery procedures. This creates a traceable relationship between requirements and implementation, which is necessary for audit readiness.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_r3t8w2fn)

### Drafting the three-wave migration plan

The migration plan defines how workloads transition into the platform.

The plan is structured into phases that include data classification, staged deployment, and rollback strategies. Each phase is documented with clear criteria, ensuring that transitions are controlled and reversible.

Configuration manifests serve as the baseline for deployment, enabling consistent rollout across environments.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_h8c4p1mz)

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_b9g3k7xe)

### Generating the portability scorecard and system boundary diagram

The system is evaluated for cross-environment consistency.

The portability scorecard measures how easily workloads can move between clusters. Lower scores highlight gaps in enforcement, such as incomplete GitOps synchronization. Improvements focus on ensuring that all clusters maintain alignment with the declared state.

The system boundary diagram provides a visual representation of components, connections, and control layers, supporting both technical understanding and compliance documentation.

## Quality Review and ATO Readiness Assessment

### Running the ATO readiness review across all deliverables

This step evaluates the system as if it were preparing for an Authority to Operate.

All artifacts are reviewed for accuracy, traceability, and alignment with compliance requirements. This includes validating configuration files, control mappings, and documentation against expected standards. The goal is to confirm that the system is not only functional but also defensible under audit conditions.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_wr4n8t6j)

### Validating NIST control IDs and GovRAMP requirements

The control mapping is assessed for completeness and accuracy.

Gaps are identified where controls lack direct artifact references, and these gaps are documented in a remediation register. This ensures that every control either has supporting evidence or a defined path to implementation. Some mappings remain incomplete and require alignment with official GovRAMP control definitions, indicating that the system is partially but not fully compliant.

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_qz8f4w2d)

### Benchmarking against production government systems

The system is compared against real-world environments.

Differences are identified between the lab and production systems, particularly in operational maturity. Missing contexts, such as incorrect cluster targeting during execution, highlight the importance of environment validation and configuration consistency. These gaps demonstrate where additional controls and verification steps are required before production use.

## Secret Mission: Crossplane vs. OpenTofu Portability Comparison

![Image](https://learn.nextwork.org/refreshed_maroon_timid_jujube/uploads/3c745f73-b8de-4458-bbf6-6e667f1b9e49_k7m2x9pq)

### Comparing infrastructure paradigms and recommending an approach

The recommendation separates responsibilities between tools.

Crossplane is suited for application-scoped resources that need to align with Kubernetes workflows and namespace-level control. OpenTofu is better suited for foundational infrastructure such as networks, compute resources, and shared services where state management and provider maturity are critical.

Using both tools requires strict ownership boundaries. Each resource must be managed by a single system to avoid conflicts and inconsistent state.

## Wrapping Up: Teardown and Cost Verification

### Running tofu destroy and confirming teardown

The system is decommissioned through infrastructure teardown.

All resources created during the lab are removed using OpenTofu, ensuring that no active infrastructure remains. This step confirms that the environment can be fully reset and that resource lifecycle management is controlled.

### Verifying billing alerts and orphaned resources

Cost validation ensures that no unintended resources persist.

Billing alerts are reviewed, and resource inventories are checked across all providers. This confirms that teardown was complete and that no orphaned resources remain, preventing unexpected charges.

### Cost leak prevention checklist

The system enforces cost discipline through validation.

Verification steps ensure that all infrastructure has been removed and that monitoring mechanisms are in place to detect anomalies. This reinforces operational hygiene in multi-cloud environments.

---

*Built with [NextWork](https://learn.nextwork.org) - [View this project](https://learn.nextwork.org/projects/3c745f73-b8de-4458-bbf6-6e667f1b9e49)*
