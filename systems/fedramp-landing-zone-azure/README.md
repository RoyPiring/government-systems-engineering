# FedRAMP Landing Zone on Azure

> Inside the [Government Systems Engineering](../../README.md) portfolio · *Cloud systems engineered for federal-grade security and compliance.*

## Overview

This project establishes a FedRAMP-ready landing zone on Azure using Terraform, Azure Policy, and Prowler.

The goal is to build a secure baseline aligned to NIST 800-53 controls, where compliance is enforced through infrastructure, not documentation.

The architecture is built across **7 phases**, anchored by **Setting Up the Compliance Toolkit** on the input side and **Wrapping Up: What Was Built and What Comes Next** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: FedRAMP Landing Zone on Azure
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart LR
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic







    Dev[/Operator with Azure CLI/]
    Cursor(Cursor Composer 2)
    TF(Terraform azurerm provider)

    subgraph MGH[Mgmt Group Hierarchy]
        Root(FedRAMP-Lab Root)
        Plat(Platform MG)
        Work(Workloads MG)
    end

    subgraph Policy[Azure Policy Guardrails]
        P1{{CM-7 Least Functionality}}
        P2{{SC-7 Boundary Protection}}
        P3{{SC-28 Encryption at Rest}}
    end

    subgraph Audit[Audit Layer NIST AU-2/3/4]
        Diag(Activity Log Diagnostic Settings)
        Logs[(Audit Log Storage Account)]
    end

    subgraph Scan[Compliance Scanning and Reporting]
        Prowler(Prowler CIS Azure Benchmark)
        Cross[/CIS to NIST 800-53 Crosswalk/]
        SSP[/System Security Plan/]
    end

    Dev -->|scaffolds| Cursor
    Cursor -->|generates HCL| TF
    Dev -->|terraform apply| TF
    TF -->|provisions| Root
    Root -->|propagates to| Plat
    Root -->|propagates to| Work
    Root -->|assigns| P1
    Root -->|assigns| P2
    Root -->|assigns| P3
    Plat -->|emits events| Diag
    Work -->|emits events| Diag
    Diag -->|writes audit| Logs
    Prowler -->|scans 160+ checks| Root
    Prowler -->|findings feed| Cross
    Cross -->|maps to controls| SSP
    P3 -->|improves coverage in| Prowler
class Logs datastore
class P1,P2,P3 event

    class Logs datastore
    class Cursor,TF,Root,Plat,Work,Diag,Prowler service
    class P1,P2,P3 event
    class Dev,Cross,SSP io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/fedramp-landing-zone-azure.md`](./documents/fedramp-landing-zone-azure.md).

## Implementation

This system is built across **7 phases**:

1. **Setting Up the Compliance Toolkit**
2. **Building the Azure Management Group Hierarchy**
3. **Enabling Centralized Audit Logging**
4. **Running the Prowler CIS Azure Benchmark Scan**
5. **Mapping Findings to NIST 800-53 and Drafting the SSP**
6. **Enforcing Encryption at Rest (SC-28)**
7. **Wrapping Up: What Was Built and What Comes Next**

For the full walkthrough with screenshots and step-by-step content, see [`documents/fedramp-landing-zone-azure.md`](./documents/fedramp-landing-zone-azure.md).

## Validation

Build outcomes verified end-to-end. Each phase below is captured with screenshots, configuration, and observable behavior in [`documents/fedramp-landing-zone-azure.md`](./documents/fedramp-landing-zone-azure.md):

- ✅ Setting Up the Compliance Toolkit
- ✅ Building the Azure Management Group Hierarchy
- ✅ Enabling Centralized Audit Logging
- ✅ Running the Prowler CIS Azure Benchmark Scan
- ✅ Mapping Findings to NIST 800-53 and Drafting the SSP
- ✅ Enforcing Encryption at Rest (SC-28)
- ✅ Wrapping Up: What Was Built and What Comes Next
