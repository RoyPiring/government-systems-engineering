<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# The Enclave That Could Not Connect

**Project Link:** [View Project](https://nextwork.ai/projects/6a7bc358-75be-47de-8c16-c67e610f6cb7)

**Author:** Roy Piring Jr: Sr. Cloud Engineer | Architect  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/6a7bc358-75be-47de-8c16-c67e610f6cb7_fv8etv3v)

## Committing to the Build

### The enclave, the connection, and the finding

I built the simulated enclave to model a SECRET-level mission environment while using OSCAL to structure its compliance and governance evidence. The goal was to show that a classified network is defined not only by deployed resources, but by whether its boundaries and controls can be demonstrated to an authorizing official.

The connection was therefore part of the system itself. Until the enclave could prove which traffic was allowed, which paths were closed, and which controls applied at each boundary, it was not ready to connect to mission services.

I also injected a cross-domain finding on purpose. The finding tested whether the architecture and policy gates could detect an unsafe route, stop the approval path, and return the enclave to zero scanner violations after remediation.

## Categorizing the Enclave and Drawing the Boundary

### Why categorization precedes every resource

In this step, I categorized the enclave before creating infrastructure. I imported the NIST SP 800-53 Rev 5 OSCAL catalog and tailored the baseline through the CNSSI 1253 Rev 5 profile.

I also documented the major architecture decisions before construction. That created an auditable record of the security assumptions, control baseline, and boundary model the Terraform implementation would later have to satisfy.

This sequence mattered because the wrong categorization would produce the wrong control set from the start. Infrastructure built against the wrong baseline could be technically sound and still fail authorization.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/6a7bc358-75be-47de-8c16-c67e610f6cb7_ars96uhg)

### OSCAL catalog import and FIPS cryptography profile

I imported NIST SP 800-53 Rev 5, Release 5.2.0, as nist-800-53-r5.

The OSCAL profile used its modify block to set parameter sc-13_prm_1.

The enforced value was FIPS 140-3 validated cryptography, which made the cryptographic expectation part of the machine-readable control profile instead of leaving it as an informal design note.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/6a7bc358-75be-47de-8c16-c67e610f6cb7_nysax47q)

### Three boundaries and why they stay separate

I modeled three separate boundaries: the Mission Authorization Boundary, Network Connection Boundary, and Provider Inheritance Boundary.

They remained separate because each boundary had a different authorization authority and different evidence requirements. The system security plan addressed the mission enclave, the connection package addressed approved network reach, and the provider boundary documented controls inherited from AWS IaaS.

Keeping them separate made each evidence path easier to audit. It also prevented inherited cloud controls from being confused with controls that remained the enclave team's responsibility.

## Building a Closed Enclave Baseline with Zero Violations

### Closed by construction, not by runtime discovery

In this step, I translated the boundary design into Terraform. The enclave VPC had no internet gateway or NAT gateway, Transit Gateway route tables enforced isolation, and an inspection VPC used AWS Network Firewall for approved traffic paths.

The baseline also included FIPS 140-3 KMS encryption, hardened AMIs, deny-all security group policies, and SPIFFE/SPIRE for workload identity.

I then evaluated the infrastructure through four policy-as-code scanners. The acceptance bar was zero violations before the enclave could move forward.

### Why no internet gateway or NAT gateway exists

The enclave was closed by design because SECRET-equivalent and IL6 traffic was not allowed to reach the public internet.

Removing both the internet gateway and NAT gateway eliminated the direct public egress path. The architecture did not rely on runtime detection to notice unauthorized internet traffic after it happened.

Approved outbound reach could only move through the Transit Gateway and the inspection or cloud-access-point path. That made the permitted route explicit while leaving the open internet unavailable by construction.

## Assembling the Connection-Approval-Equivalent Package

### The connection is the system

In this step, I assembled the connection-approval-equivalent package that would allow the enclave to move from an isolated network into an authorized mission connection.

The package included the decision document, ports and protocols registration, and consent-to-monitor statements. These artifacts defined what the enclave was allowed to connect to and under which conditions.

This mattered because a secure enclave with no approved mission path is still only an isolated environment. The connection package tied the technical boundary to an authorization decision.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/6a7bc358-75be-47de-8c16-c67e610f6cb7_cuqf4df8)

### Allowed protocols and how all other traffic is denied

The approved protocol set included TLS 1.3 on port 443 for SPIRE server communication and gRPC over TLS on port 8081 for the Workload API.

All other traffic was denied by construction. The enclave-isolated Transit Gateway route table used blackhole routes, enclave security groups denied egress, and remaining approved flows passed through Network Firewall inspection.

The result was an allow-by-exception model. Traffic existed because a path was explicitly approved, not because the network was open and expected to filter unsafe traffic later.

## Injecting the Finding, Stalling the Connection, and Re-Designing the Boundary

### A boundary that has never met a reviewer is a diagram

In this step, I deliberately introduced an unauthorized cross-domain route between the SECRET mission enclave and the UNCLASSIFIED shared-services VPC.

The purpose was to test whether the four-scanner gate could detect a real boundary failure and stop the connection approval process. A secure design needed evidence that a bad route would be caught, not only evidence that the original configuration passed.

After detection, I removed the unsafe route and replaced it with an explicit blackhole path so the shared-services CIDR remained unreachable from the enclave.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/6a7bc358-75be-47de-8c16-c67e610f6cb7_jqm4xwqb)

### What the finding proved and how the re-design cleared it

The injected non-blackhole Transit Gateway route proved that the enclave boundary was only as strong as its route table. One static path to the shared-services CIDR was enough to reopen unaccredited cross-domain reachability.

I removed that route, added an explicit blackhole for the shared-services CIDR on the enclave-isolated route table, and kept propagation disabled.

The re-scan returned zero scanner violations, and POAM-001 closed with 0 open items. That proved the finding had moved through detection, remediation, and evidence-backed closure.

## Briefing the Mock Authorizing Official

### Three transferable patterns from the enclave lifecycle

The first pattern was to categorize before building. If the baseline is wrong, the controls, evidence, and implementation begin from the wrong assumptions.

The second pattern was to treat the finding as part of the design review. A boundary that never faces a failure test is only a diagram. The injected route forced the system to prove that its controls could detect and close a real cross-domain path.

The third pattern was that the connection is the system. Without connection approval, the enclave does not exist as an authorized participant on the mission network.

## Reflections and Lessons Learned

### Key tools and concepts

The key tools I used included Terraform for infrastructure provisioning, compliance-trestle for OSCAL artifacts, Conftest for policy-as-code enforcement, SPIRE for workload identity, and AWS services including Transit Gateway, Network Firewall, and KMS.

The main concepts I learned included the categorize-tailor-build-package-brief workflow, documenting boundaries through ADRs before construction, representing inherited controls in OSCAL, and using policy gates to detect and remediate cross-domain failures.

The larger lesson was that enclave security depends on evidence as much as configuration. A closed network, authorization package, inherited control model, and tested failure path all had to agree before the connection could be defended.

### Time and challenge

This build took me approximately 90 minutes. The hardest part was keeping the OSCAL control mappings aligned with the infrastructure changes made during remediation of the cross-domain finding.

The route-table change affected more than Terraform. The policy scans, POA&M status, and authorization evidence also had to reflect the redesigned boundary so the technical state and documented state did not diverge.

I completed this build to learn how to create a zero-trust mission enclave, validate its compliance through OSCAL, and move through a connection-approval lifecycle that included a real failure and remediation path. Next, I want to learn how to automate continuous compliance monitoring across multi-cloud environments.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/6a7bc358-75be-47de-8c16-c67e610f6cb7)*
