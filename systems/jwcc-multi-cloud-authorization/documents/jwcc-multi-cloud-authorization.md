<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Simulate a JWCC Multi-Cloud Authorization

**Project Link:** [View Project](https://nextwork.ai/projects/790df98b-5bdc-410f-93d6-9663f417999e)

**Author:** Roy Piring Jr  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/790df98b-5bdc-410f-93d6-9663f417999e_tgk6qas3)

## Mission Overview: JWCC Multi-Cloud Authorization Simulation

### What this project set out to prove

This project onboards a DoD logistics data pipeline onto a JWCC-equivalent multi-cloud architecture. The goal was to show that the workload could be deployed across AWS, Azure, Google Cloud, and OCI while keeping a clear RMF/FISMA authorization boundary.

The system tested more than cloud portability. It had to prove that provider failover, control inheritance, policy checks, encryption, networking, monitoring, and evidence generation could stay aligned across different cloud environments.

This mattered because a workload can run in more than one cloud and still fail authorization if the boundary, inherited controls, and evidence package are not rebuilt correctly. The build made that distinction explicit.

## Defining the Authorization Boundary and Agent Swarm

### Setting up the 44-agent organization

In this step, I established the governance and operating structure for the 44-agent swarm. I created a .cursorrules file and a project roster so each agent had a clear role, boundary, and delivery path.

The swarm was built around a boundary-first method. That meant every agent had to work from the authorization boundary before producing cloud modules, evidence files, diagrams, or briefing artifacts.

This structure mattered because a JWCC authorization simulation needs consistency across providers. The roster and rules kept the agent work aligned so the output stayed audit-ready, portable, and tied to the RMF/FISMA boundary.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/790df98b-5bdc-410f-93d6-9663f417999e_j6o3k9r0)

### RPO/RTO targets and failover implications

The mission set an RPO of 15 minutes and an RTO of 30 minutes. That meant the pipeline could lose at most 15 minutes of logistics data and had to restore the full pipeline, including boundary inspection, within 30 minutes.

Those targets shaped the failover design. Provider failover had to complete inside the RTO window, so replication lag, DNS or routing cutover, and CAP-equivalent inspection paths all had to support recovery in under 30 minutes.

The targets also set the minimum replication and backup cadence. Any CSP or fallback path that could not meet both targets had to be documented as a gap instead of treated as a silent failover option.

### Initial capability gap discovery

The main capability gap was data_classification, meaning managed data discovery and classification for CUI.

GCP was limited because it only provided DLP API coverage, which required API-based orchestration instead of a fully managed service like Macie or Purview. OCI had no equivalent managed service for this capability.

All other capabilities in the matrix were available across the four JWCC providers. Identity, networking, state storage, cryptography, and monitoring had provider paths on AWS, Azure, GCP, and OCI.

## Building the Cloud-Agnostic Baseline Across Four Providers

### Engineering the provider modules in parallel

In this step, I used the 44-agent swarm to build the multi-cloud baseline in parallel. The work centered on creating a cloud-agnostic Terraform interface and provider-specific modules for AWS, Azure, GCP, and OCI.

The architecture used a CAP-equivalent network boundary with centralized inspection egress and FIPS-validated cryptographic controls. That kept the authorization boundary visible instead of letting each provider use its own unmanaged network pattern.

I also implemented policy-as-code validation with Conftest and Checkov. Those gates checked whether the provider plans met the same security posture before they could be treated as valid deployment paths.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/790df98b-5bdc-410f-93d6-9663f417999e_mkqbog19)

### Mapping Rego policies to NIST 800-53 controls

The four Rego policies mapped to NIST 800-53 Rev 5 controls. sc7_boundary.rego and sc7_egress.rego mapped to SC-7, sc28_encryption.rego mapped to SC-28, and sc8_transmission.rego mapped to SC-8.

The CAP-equivalent boundary enforced SC-7 by forcing all egress through a central inspection point instead of allowing direct internet gateway breakout. The inspection path used AWS Network Firewall, Azure Firewall Premium, GCP Cloud IDS with packet mirroring, and OCI DRG with Network Firewall.

Conftest validated that no workload route sent 0.0.0.0/0 to an IGW without passing through the inspected transit path. That made boundary control enforcement testable through policy instead of relying on diagram intent.

## Surfacing the Capability Gap and Proving Portability

### Why the baseline breaks and how to route around it

In this step, I identified where the cloud-agnostic baseline failed to meet the mission requirement. The break came from managed data classification, which was not equally supported across all JWCC providers.

I built a capability-mapping layer to route around provider-specific gaps. Instead of pretending every cloud had the same services, the system checked capability support and routed the missing function to a provider that could support it.

I then proved portability by deploying and verifying the logistics pipeline workload on a secondary cloud provider. That showed which parts of the workload were portable and which parts required explicit routing.

### OCI gap and capability-router decision

Managed data classification and discovery, used as a Macie or Purview equivalent for SC-28 and AU-12 support, was missing from OCI. The parity matrix marked it as status: gap and service: none.

When data_classification_required=true, the router followed the fallback order of AWS and Azure. It placed classification on the first supported provider, which was AWS by default, and never routed that function to OCI.

If the primary provider was OCI, the system recorded a degradation. The core workload stayed on OCI, while classification split to AWS or Azure, and only Macie or Purview was enabled on the routed provider.

### Multi-provider portability proof

Portability was proven by running terraform plan on two provider modules. AWS acted as the primary provider, and Azure acted as the alternate.

Both plans exposed the same interface outputs: instance_id, private_ip, network_id, and encryption_status. The core workload, including compute, network, and cryptography, planned successfully on both providers.

AWS planned 5 resources to add, and Azure planned 7 resources to add. Data classification was not portable, so the capability router sent it to AWS or Azure based on parity-matrix.yaml, with OCI excluded and the result documented in capability-map/portability-proof.md.

## Tailoring the IL5-Equivalent Control Set and Structuring OSCAL Evidence

### Tailoring 800-53 Rev 5 for the mission baseline

In this step, I tailored the IL5-equivalent control set for the mission baseline. The work documented control inheritance across the load-bearing families and separated provider-owned, mission-owned, and shared responsibilities.

I structured the compliance evidence with OSCAL and compliance-trestle so the authorization package could be read and validated as structured security data. This made the evidence more useful than static notes because controls, components, gaps, and findings could be traced.

I also built an interactive dashboard to show the requirements and validation status. The final check was making sure each system-owned control had verified policy-as-code coverage where the build claimed enforcement.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/790df98b-5bdc-410f-93d6-9663f417999e_da8kx75v)

### Inherited vs. system-owned control distinction

Inherited controls were satisfied by the CSP’s FedRAMP High PA. The mission cited AWS, Azure, GCP, and OCI attestation and did not add its own separate implementation for those controls.

System-owned controls were implemented in the mission repository. Those controls lived in Terraform under boundary/cap-equivalent/ and modules/, Conftest Rego policies, Checkov checks, and process documents for areas such as personnel security and incident response.

Shared controls split responsibility between the provider and the mission. The provider supplied the platform capability, while the mission configured and monitored it, such as SC-28 encryption and data classification.

### Four named validation bars and their evidence links

The boundary validation bar linked to evidence/oscal/control-tailoring.yaml, jwcc-simulation-ssp.json, docs/diagrams/system-security-boundary.svg, and boundary/cap-equivalent/.

The control coverage bar linked to coverage-validation-report.md, component-definition.json, poam.json, and trestle validate -a over evidence/oscal/.

The crypto validation bar linked to policies/rego/sc8_transmission.rego, policies/rego/sc28_encryption.rego, boundary/cap-equivalent/aws.tf with the FIPS CMK, and evidence/dashboard/conftest-summary.md. The portability validation bar linked to capability-map/portability-proof.md and capability-map/capability-router.tf.

## Delivering the ATO-Style Authorization Package

### Assembling presentation-ready artifacts for the mock AO

In this step, I consolidated the technical output into an ATO-style authorization package. The package brought together the Terraform modules, OSCAL evidence, and policy-as-code gates into presentation-ready artifacts.

I rendered the boundary diagram, built the Marp slides, and created the interactive compliance dashboard. These artifacts gave the mock Authorizing Official a risk-focused view while still giving engineering teams enough technical detail to understand the implementation.

The final readout connected stakeholder communication with engineering evidence. It documented the lessons learned and showed whether the IL5-equivalent mission requirements were met, limited, or dependent on provider-specific capability.

### Five structural caveats in the authorization package

The package had five load-bearing caveats. It was a simulation only, not a real IL5 onboarding. The AO owned the authorization decision, while agents only drafted evidence. Chaos scenarios were illustrative, provider capabilities could change, the parity matrix was dated and had to be revalidated before decisions, and any provider move required control re-inheritance, boundary redraw, and AO review.

Those caveats were required because an authorization package must not imply that an ATO has been granted. It also had to preserve human decision authority under RMF and flag time-bound or provider-dependent evidence clearly.

Slide 11 also listed a sixth meta-caveat: dependency class. The package depended on software, human judgment, and provider capability state.

### Where the cloud-agnostic pattern held and where it broke

The cloud-agnostic pattern held for the core workload. Shared interface outputs, encryption under SC-28, the CAP boundary under SC-7, SC-8, and SC-13, and identity, networking, and monitoring worked across the four providers.

Conftest passed on every plan, which showed that the core security posture could be checked across providers. That supported the portability claim for the parts of the workload that had equivalent capabilities.

The pattern broke at managed data classification. AWS and Azure had native Macie and Purview paths, GCP required custom DLP orchestration, and OCI had no equivalent. The lesson was clear: plans on four clouds do not mean authorized on four clouds. Capability mapping and explicit routing were required, with AWS to Azure fallback and OCI excluded from classification workloads.

## Secret Mission: Provider Failover Against RPO and RTO

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/790df98b-5bdc-410f-93d6-9663f417999e_s9p9ljb2)

### Measured RTO, RPO documentation, and re-authorization implication

The measured RTO was 4 minutes and 40 seconds, or 4.67 minutes, from parity-matrix gap declaration to a successful Azure failover plan. That result was well under the 30-minute mission target and was documented in capability-map/failover-proof.md.

The RPO was documented as a 15-minute design target, not a live lag measurement. The evidence cited AWS S3 RTC at 99.99% within 15 minutes and Azure Storage RA-GRS with a typical RPO under 15 minutes, plus mission assumptions for state and secrets.

The re-authorization finding mattered most. A provider move was not a free swap. It re-inherited 107 controls from the fallback CSP’s FedRAMP PA, redrew the authorization boundary, and required the updated SSP and diagram to return to the Authorizing Official before continued operation.

## Reflections and Takeaways

### Key tools and concepts from the build

The key tools I used included Cursor Composer for managing the 44-agent swarm, Terraform for multi-cloud infrastructure deployment, Conftest and Checkov for policy-as-code enforcement, and compliance-trestle for OSCAL security documentation.

I also used Marp to render authorization briefings and maintained a capability parity matrix to track provider-specific gaps. Those tools helped turn the authorization simulation into a structured system with deployable modules, measurable policy gates, and traceable evidence.

The main concepts I learned included boundary-first architecture for RMF compliance, agentic workflows for cross-provider engineering, and maintaining security posture during provider failover. The most important lesson was the difference between engineering failover and authorization failover. Engineering failover was handled by the capability-mapping layer, while authorization failover required formal reassessment of inherited controls when crossing provider boundaries.

### Time and challenges

This build took me approximately 2 hours. That time covered the authorization boundary, Terraform provider modules, policy-as-code gates, capability mapping, OSCAL evidence, dashboard work, failover proof, and ATO-style briefing materials.

The hardest part was managing the capability-mapping layer so the failover configuration kept the same security posture across providers while Rego and Checkov gates passed consistently. The build had to route around provider gaps without hiding them.

Every provider move changed inherited controls, which meant the boundary documentation had to stay audit-ready and return to the AO before continued operation.

I completed this build to understand how to architect a multi-cloud onboarding simulation with AI-driven agents that could generate Terraform modules, OSCAL compliance evidence, and authorization briefing materials. Moving forward, I want to learn how to integrate automated security policy gates into CI/CD pipelines so continuous compliance can hold across different cloud environments.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/790df98b-5bdc-410f-93d6-9663f417999e)*
