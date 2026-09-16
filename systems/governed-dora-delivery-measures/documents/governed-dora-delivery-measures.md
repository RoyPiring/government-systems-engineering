<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Govern DORA Metrics with DevLake

**Project Link:** [View Project](https://nextwork.ai/projects/7220c8e7-318e-4d8d-a824-b6b0b4475ff1)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/7220c8e7-318e-4d8d-a824-b6b0b4475ff1_w63rz35q)

## Framing the Federal Delivery Decision

### Defining the service, decision-maker, and governance decision

I built this measurement system to give a federal delivery leader defensible evidence about software delivery performance across a GitLab deployment pipeline. The decision was not simply whether a metric looked good or bad. It was whether the measured results were trustworthy enough to influence funding, operational priorities, and future delivery controls.

I defined five governed measures: Deployment Frequency, Change Lead Time, Change Failure Rate, Failed Deployment Recovery Time, and Rework Rate. Each measure needed a named source, declared calculation, known limitation, and reversal condition. This prevented the dashboard from presenting numbers without explaining how they were produced. I treated the dashboard as a decision surface backed by a governance contract, not as an independent source of truth. Any funding or performance conclusion still depended on complete GitLab deployment evidence and separately collected incident records.

## Building the Measurement Workbench

### Preparing the local DevLake measurement environment

I installed the Apache DevLake stack on my local Windows workstation and verified that its required services started successfully. The workbench included DevLake for collection and transformation, Grafana for visualization, and the supporting database services defined by the Docker Compose configuration. This created one controlled environment for configuring sources, collecting data, and testing the governed measures.

I checked each service before building the dashboard so a missing container or unreachable endpoint would not be confused with absent delivery evidence. I also preserved the Compose configuration and workspace structure needed to repeat the setup. The local environment supported the measured build, but it did not establish production availability, resilience, or scale. Its purpose was reproducibility: the same source configuration, governance files, and collection sequence could be rerun while keeping experimental changes separate from live delivery systems.

### Verifying GitLab, GitHub, and Linear access

I verified access to all three systems before collecting evidence. GitHub CLI was authenticated as RoyPiring with repository access, allowing me to publish the governed evidence package and create the Evidence 1.0 release. Linear was authenticated to the Roy Piring workspace with administrator access, which supported task tracking and the independent review request.

GitLab Community Edition 19.3.1 was running locally at http://localhost:8929. The root account could create private projects, seed commits and merge requests, and record production deployments. I treated these access checks as readiness evidence, not proof that each source contained complete data. Authentication only showed that the pipeline could reach the systems. The later collection results determined which delivery, incident, and governance records were present and usable for the five governed measures.

### Protecting the DevLake encryption secret

I generated the DevLake ENCRYPTION_SECRET with the Windows cryptographic random generator rather than choosing a human-readable password. The resulting value contained 128 uppercase characters. Docker Compose read it from .env, while a second copy was stored in secrets/devlake-encryption-secret.txt to reduce the risk of losing the only usable value.

Both paths were covered by Git ignore rules, so the secret was excluded from repository commits. I also kept the value out of chat responses and screenshots because DevLake used it to protect stored connection credentials, including tokens and passwords. Losing or changing the secret could make those encrypted values unreadable and require the connections to be recreated. The duplicate local copy protected recoverability on this workstation, but it was not an enterprise secret-management design. A production deployment would require controlled backup, access logging, rotation procedures, and an approved secrets service.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/7220c8e7-318e-4d8d-a824-b6b0b4475ff1_reei3ej2)

## Defining Defensible Measurement Governance

### Creating the five-measure contract and decision records

I created a governance pack that defined the five delivery measures before building their Grafana panels. Each definition named its source systems, calculation boundaries, interpretation limits, validation method, and decision use. This prevented metric definitions from shifting after the results were visible or after a stakeholder challenged an unfavorable number.

I also recorded four Architecture Decision Records covering source selection, incident ingestion, rework classification, and governance handling. Each record included a reversal trigger so the current choice could be replaced when specified evidence or platform capability appeared. The contract separated a measurement definition from its displayed value: a panel could change as new data arrived without silently changing what the metric meant. This structure made the system auditable because a reviewer could trace each number from its panel to its data source, governing decision, and required validation evidence.

### Tracing how missing incidents create a false gain

Change Failure Rate depended on incident records ingested through the Incoming Webhook and linked to production deployments. When that path was disconnected, DevLake still retained the GitLab deployment activity but rebuilt the project relationships without incident input. Failed changes disappeared from the calculation, causing the rate to fall even though delivery performance had not changed.

Assertion 3 made this weakness observable. I recorded the original value, detached the webhook, ran Collect Data, and captured the lower result. I then restored the webhook, collected again, and confirmed that the earlier, less favorable rate returned. The governance pack required timestamped before, broken, and restored measurements. A narrative statement without those values failed the check. This negative control proved that missing incident evidence could make the dashboard look healthier and that a better number did not necessarily represent better delivery.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/7220c8e7-318e-4d8d-a824-b6b0b4475ff1_os0chbnz)

## Establishing Native GitLab Delivery Evidence

### Ingesting commits, merge requests, and production deployments

I created a private GitLab repository as the controlled source for delivery evidence. I seeded commits, merge requests, and production deployment events so DevLake could collect a known activity history. This gave Deployment Frequency and Change Lead Time native records tied to the same repository and deployment process rather than combining unrelated examples from different systems.

I configured the GitLab connection, associated it with the DevLake project, and ran collection before reviewing the transformed data and dashboard results. The seeded history established when changes were committed, reviewed, merged, and deployed to production. It did not create incident evidence because successful deployments and source-control activity could not prove whether a release caused service impairment. The repository therefore supplied the delivery side of the measurement model, while the independent webhook supplied the operational failure and recovery records needed for the stability measures.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/7220c8e7-318e-4d8d-a824-b6b0b4475ff1_b7c1y7lv)

### Recognizing incomplete stability evidence

The GitLab collection populated delivery activity but left the incident-backed measures empty. Change Failure Rate required evidence that a production deployment caused an incident, while Failed Deployment Recovery Time required timestamps showing impairment and restoration. GitLab commits, merge requests, and successful deployment records did not contain those facts in this implementation.

I treated the empty panels as a data-boundary finding rather than a zero failure rate. A zero would have implied that every production change succeeded, while the evidence only showed that the selected source had not supplied incidents. The missing values identified the next required source: Incoming Webhook INCIDENT records linked to the relevant deployments. This distinction protected the dashboard from false certainty. GitLab could prove that delivery occurred and support frequency and lead-time calculations, but it could not independently prove failure, recovery, or the absence of either condition.

## Connecting Incident Evidence to Deployment Outcomes

### Mapping independent incident records to deployments

I connected an Incoming Webhook to the DevLake project and used it to ingest incident records independently of GitLab. Each incident carried the timestamps and deployment relationship needed to connect an operational failure with a production change. This completed the evidence path required for Change Failure Rate and Failed Deployment Recovery Time.

After ingestion, I ran Collect Data and inspected the project-scoped relationships that DevLake created between incidents and deployments. The stability measures populated only when those links existed. This design kept deployment activity and incident reporting as separate sources, which reduced the risk that a successful deployment record would be mistaken for proof of operational success. It also introduced a dependency that required monitoring: if the webhook stopped sending data, the dashboard could read better than reality. The negative control later tested that exact failure mode and documented the resulting measurement change.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/7220c8e7-318e-4d8d-a824-b6b0b4475ff1_smgdb3oy)

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/7220c8e7-318e-4d8d-a824-b6b0b4475ff1_oviza339)

## Publishing a Governed Five-Measure Delivery Dashboard

### Publishing source-labelled delivery measures

I published a Grafana dashboard containing the five governed delivery measures. Every panel identified its measure, governing definition, and source path so a viewer could distinguish GitLab-derived delivery activity from webhook-derived incident evidence. The dashboard presented results, while the governance pack documented how each result should be interpreted and challenged.

I released the validated evidence package through GitHub as Evidence 1.0. The package included the decision records, validation requirements, source labels, and negative-control evidence supporting the dashboard. Publication froze a reviewable baseline without claiming that future values would remain unchanged. New collection runs could update the measurements, but changes to definitions or source responsibilities required governance review. This approach kept operational reporting connected to a versioned measurement contract and made missing evidence visible instead of allowing an attractive panel to stand without qualification.

### Proving the incident-ingestion negative control

I detached the Incoming Webhook and ran Collect Data while leaving the four production deployments unchanged. DevLake rebuilt the project-scoped relationships without incident input, causing the incident-to-deployment relationship table to become empty. Grafana then reported Change Failure Rate as 0%, making the delivery system appear healthier even though no deployment behavior had changed.

I reattached the webhook, collected again, and confirmed that the incident relationships and original failure rate returned. The before, broken, and restored readings demonstrated causality between the missing source and the false gain. This was a negative control for measurement integrity, not a test of application reliability. It showed that the dashboard could produce a favorable value when a required evidence path disappeared. The governance response was to monitor source completeness and reject unqualified interpretations of a zero rate when incident ingestion had not been verified.

## Red-Teaming a Measurement Decision

### Reviewing the challenged decision and its evidence

I created Linear item PS-160 to request an independent challenge of one decision from ADR-001 through ADR-004. The reviewer received the Evidence 1.0 release, governance/decisions.md, governance/validation.md, and a live view of Federal Delivery Measures. This gave them the released evidence and documented reversal criteria needed to test a decision without rewriting the original record.

No reviewer result was recorded in Linear at the time of the readout. The issue still asked the reviewer to choose a decision, and no comment identified which ADR they challenged or what conclusion they reached. I therefore left the independent-review outcome open rather than reporting completion from the existence of the task alone. The evidence package was ready for review, but assignment and access did not prove that a neutral reviewer had performed the challenge. A final result required a named ADR, tested evidence, conclusion, and recorded rationale.

### Testing the documented reversal trigger

ADR-003 selected Incoming Webhook as the incident source and defined a specific reversal trigger: DevLake would need to provide a native incident connector with equivalent deployment mapping and timestamp fidelity. The review checked the current configuration and available evidence against that condition rather than reconsidering the decision from preference.

Incoming Webhook remained active, and no equivalent native connector was present in the tested environment. The reversal trigger therefore did not fire, so the recorded verdict remained Accept and the released decision stayed unchanged. This result did not mean Incoming Webhook was permanently preferred or that no alternative could work. It meant the predeclared condition for replacing it had not been satisfied. Keeping the trigger explicit prevented the architecture from changing without comparable evidence and allowed a future review to reverse the decision without rewriting its original rationale.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/7220c8e7-318e-4d8d-a824-b6b0b4475ff1_ambgflws)

## Reflecting on the Delivery Measurement Build

### Tools and measurement concepts applied

I used Apache DevLake to collect and transform delivery evidence, Grafana to publish source-labelled measures, and GitLab Self-Managed 19.3.1 to provide commits, merge requests, and production deployments. Incoming Webhook supplied independent incident records. GitHub Releases preserved Evidence 1.0, while Linear tracked the delivery work and independent review request.

The main lesson was that DORA metrics required governance as much as calculation. I defined five measures before publishing them, tied each panel to its sources, and documented reversal conditions for major design choices. The incident-path negative control showed that missing evidence could produce a better result without better performance. I also learned to keep review history additive: a challenge could accept or reverse a decision, but it should not erase the original record. Together, these controls made the dashboard useful for decisions without presenting it as unquestionable truth.

### Implementation timeline and challenges

I completed the build in approximately 90 minutes. The most difficult part was coordinating the independent review without contaminating it. The reviewer needed enough context to understand the released evidence and reversal triggers, but I could not choose the challenged ADR for them or convert an unrecorded review into a completed result.

The implementation also required separating missing incident data from a genuine zero failure rate. GitLab supplied valid delivery evidence, yet the stability panels remained empty until the Incoming Webhook provided incident records. The negative-control sequence then had to preserve the four deployments while changing only the incident input. Capturing the before, disconnected, and restored values made that relationship defensible. These challenges reinforced that collection health, decision history, and reviewer independence were part of the measurement system rather than administrative work outside it.

### Next steps in delivery governance

I completed this build to understand how a governed DORA dashboard could combine GitLab delivery activity with independent incident evidence in Apache DevLake. The result provided five source-labelled measures, four documented decisions, a negative control for missing incidents, and a versioned evidence release. It also kept the independent-review outcome open because no reviewer conclusion had been recorded.

My next step is to automate source-health checks and governance evidence inside CI/CD. The system should flag missing webhook data, stale collection timestamps, and broken deployment-to-incident mappings before a favorable metric reaches decision-makers. I also want to explore Linear agents for coordinating reviews, provided they preserve reviewer identity, evidence links, and additive decision history. Any automation must retain the same boundary: agents can gather and test evidence, but they cannot silently change metric definitions or declare a governance decision approved.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/7220c8e7-318e-4d8d-a824-b6b0b4475ff1)*
