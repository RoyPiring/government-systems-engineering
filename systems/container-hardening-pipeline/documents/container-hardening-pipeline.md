<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Container Hardening Compliance Pipeline

**Project Link:** [View Project](https://nextwork.ai/projects/31f6ad02-8f4e-4982-afe5-72e58925c2a2)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/31f6ad02-8f4e-4982-afe5-72e58925c2a2_m9dndkyi)

## Building a Falsifiable Compliance Pipeline

### The core problem this system solves

I built a dual-gate CI pipeline that hardened container images and tested them with OpenSCAP and Trivy. The system replaced the broad question “Is this secure?” with machine-checked assertions that could pass or fail against recorded criteria.

OpenSCAP measured configuration rules, while Trivy checked the image for known vulnerabilities. The pipeline treated both results as required evidence instead of allowing one successful scanner to represent the full security state.

This approach made the outcome falsifiable because a failed rule, changed count, or detected vulnerability could contradict the claim that the image met the gate. A green run still had limits. It showed that the tested image passed the selected community-maintained checks at that moment. It did not prove complete security or replace an authoritative compliance assessment.

### Delivery structure and environment setup

I created a private GitHub repository, organized the work in Linear with epic-driven stories, and installed OpenSCAP, Ansible, and Trivy in WSL2. These components established the source, tracking, hardening, and scanning paths used by the pipeline.

I committed the initial repository structure and workflow skeleton to main before adding the full scan and remediation logic. This registered the GitHub Actions workflow and its weekly cron schedule early, making the intended delivery path visible in version control.

The foundation separated planning, implementation, and evidence. Linear tracked the required work, GitHub stored the pipeline, and WSL2 supplied the local security tools. Committing the skeleton first also gave later hardening changes a known starting point. The pipeline could then add scans and gates without changing its delivery structure midway through the build.

## Establishing the Delivery Infrastructure

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/31f6ad02-8f4e-4982-afe5-72e58925c2a2_mlwpazeg)

### Scheduled rebuild logic

The workflow used cron: '0 3 * * 1'. The five fields represented minute 0, hour 3, every day of the month, every month, and weekday 1.

GitHub Actions interprets scheduled workflows in UTC. Under that clock, the pipeline requested one weekly rebuild every Monday at 03:00 UTC. The schedule supplied recurring execution even when no developer opened a pull request or pushed a new commit.

That recurring run mattered because container packages, vulnerability data, and scan results could change while the repository stayed unchanged. A weekly rebuild gave the pipeline another chance to detect drift or newly visible findings against the same declared controls. The cron expression established when GitHub Actions should request the run. It did not prove that every scheduled job completed successfully, so workflow history remained the evidence for actual execution and gate results.

## Documenting Architectural Decisions

### Why decisions are written before the build

I wrote four Architecture Decision Records before completing the pipeline. They documented the selected content source, dual-gate behavior, weekly schedule, and vulnerability-scanning approach.

Recording these choices first gave the implementation a declared design to follow. The workflow could be checked against written decisions instead of treating its final YAML and scripts as the only explanation for why the system behaved a certain way.

The ADRs also captured trade-offs and limits. The content source supported repeatable configuration checks but was not authoritative assessment material. Both gates were required because configuration and vulnerability scans answered different questions. The schedule controlled recurring drift checks, while the scanning decision defined what evidence entered the result. Writing these boundaries early reduced the chance that accidental settings would later be presented as deliberate security policy.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/31f6ad02-8f4e-4982-afe5-72e58925c2a2_mx0f4mn7)

### ADR-001 fidelity limit and its significance

ADR-001 recorded that the pipeline used community-maintained ComplianceAsCode profiles. The resulting scans were not equivalent to an authoritative Security Content Automation Protocol assessment performed with DISA-published content or the SCC scanner used for an assessment of record.

This limit mattered because a green workflow could look like a formal compliance verdict when it only represented the selected community rules. The scan still provided useful evidence about configuration intent, repeated checks, and drift within the tested container.

I therefore treated a green result as proof that the image passed this pipeline’s declared controls, not as proof of official compliance. The distinction protected the report from claiming authority the content source did not carry. The pipeline could support engineering hardening and preparation, while the formal assessment remained a separate process with its own approved scanner, content, scope, and evidence requirements.

## Establishing the BEFORE Baseline

### Why a numbered starting point is required

I built an unhardened Ubuntu 22.04 container image with OpenSSH server and scanned it with OpenSCAP. This created a numbered BEFORE state that later remediation could be compared against.

Without a baseline, an AFTER scan could show findings but could not prove whether the hardening reduced them. The starting counts established how many rules passed, failed, or did not apply before Ansible changed the image.

The baseline also fixed the effective comparison set. OpenSCAP evaluated 228 rules, but 198 were not applicable to the container. The remaining 30 rules formed the denominator for measured change. I used the 7 initial failures as the main repair target while preserving the full count breakdown. This allowed the later result to show exactly what moved rather than reporting that the image had become “more secure” without a defined starting point or measurable difference.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/31f6ad02-8f4e-4982-afe5-72e58925c2a2_do27jw00)

### Baseline scan counts and effective denominator

The baseline OpenSCAP scan reported 23 passing rules, 7 failing rules, 198 not-applicable rules, and 228 total evaluated rules. These values described the unhardened Ubuntu 22.04 container image.

I calculated the effective denominator by subtracting the not-applicable count from the total evaluated count: 228 − 198 = 30. Those 30 rules were the checks the container scan could score under the selected profile.

The 7 failures became the main hardening target, while the 23 passes showed which applicable rules already met the expected state. Keeping the denominator fixed was necessary for the BEFORE and AFTER comparison. A lower failure count would not prove progress if the later scan had evaluated a different applicable set. By retaining the same 30-rule denominator, the pipeline could connect changes in pass and fail counts to remediation rather than a changed scope.

## Proving the Reduction with a Dual-Gate Pipeline

### Generating and applying Ansible remediation

I generated an Ansible remediation playbook from the OpenSCAP baseline failures. The playbook targeted the configuration findings identified in the unhardened image instead of applying an unrelated hardening checklist.

I applied the remediation during the container build and then scanned the resulting image again. This created an AFTER state that could be compared with the original counts under the same effective denominator.

The workflow also ran Trivy as the second gate. OpenSCAP measured configuration rules, while Trivy measured known vulnerability findings. Keeping both gates prevented a lower configuration-failure count from standing in for the full pipeline result. The Ansible step demonstrated that the baseline evidence could drive a repeatable repair path, and the follow-up scans tested whether those changes produced measurable results. The pipeline judged the built image through recorded scanner output rather than a manual review of the playbook.

### BEFORE and AFTER fail counts against the denominator

The BEFORE scan reported 7 failing rules out of an effective denominator of 30. The AFTER scan reported 1 failure against that same 30-rule set. The failure count therefore decreased by 6 without changing the applicable scope.

Passing rules increased from 23 to 29. The two movements reconciled against the fixed denominator: 23 pass plus 7 fail equaled 30 before remediation, while 29 pass plus 1 fail equaled 30 afterward.

This comparison demonstrated measurable configuration change rather than a percentage produced from shifting inputs. The remaining failure also stayed visible, preventing the result from being described as complete hardening. The evidence supported the narrower claim that the Ansible remediation moved six applicable rules from fail to pass under the selected OpenSCAP profile. The Trivy result remained a separate gate because configuration counts did not prove vulnerability status.

## Secret Mission: Regression Drill and Handover Test

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/31f6ad02-8f4e-4982-afe5-72e58925c2a2_2lgez23z)

### Runbook defects surfaced by a second person

No second person completed the handover test, so the evidence did not support a claim that another operator had successfully followed the runbook. I instead recorded the questions that blocked a cold read.

The first unanswered question was how to trigger a rebuild in GitHub Actions. The runbook did not name Actions or explain how to use Re-run jobs. The next blocker was what to do when the workflow stayed green and no failing report appeared.

The regression drill also encountered stale scan behavior because oscap-docker returned command not found. The runbook did not explain how to identify or recover from that condition. These gaps showed that implementation knowledge remained in the builder’s context rather than in the handover document. The runbook needed explicit trigger, failure-location, stale-scan, and recovery instructions before another person could repeat the drill without assistance.

## Reflections and Lessons Learned

### Key tools and concepts

I used GitHub Actions for the CI/CD workflow, OpenSCAP and the SCAP Security Guide for configuration scanning, Trivy for vulnerability detection, Ansible for remediation, and Linear for delivery tracking.

The central concept was falsifiable security evidence. The pipeline used explicit scanner results and gate conditions rather than treating a review or green icon as proof of complete security. The fixed 30-rule denominator also made the BEFORE and AFTER comparison measurable.

I learned how scheduled rebuilds can expose configuration drift or new vulnerability findings even when repository code does not change. The handover drill added another lesson: a working pipeline is not ready for another operator unless the runbook explains how to trigger it, find failures, recognize stale evidence, and recover. Technical controls and operational instructions both had to hold for the system to remain useful.

### Time and challenges

This build took approximately 90 minutes. The hardest part was parsing the ARF XML files inside the delta-report script. The logic had to compare rule states correctly and render the resulting differences as readable Markdown in the GitHub Actions summary.

A parsing error could misclassify a pass, failure, or not-applicable result and corrupt the 30-rule comparison. The report therefore needed to preserve both the raw counts and the state changes used to explain the 7-to-1 reduction.

I completed this build to learn how dual-gate scanning can make container-hardening claims testable. My next goal is to apply automated security regression testing to infrastructure-as-code configurations. That work should preserve fixed baselines, explicit denominators, separate configuration and vulnerability gates, scheduled execution, and runbooks that another operator can follow without relying on the original builder.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/31f6ad02-8f4e-4982-afe5-72e58925c2a2)*
