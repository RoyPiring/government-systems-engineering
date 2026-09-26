<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Survive the Cutover, Then Redesign

**Project Link:** [View Project](https://nextwork.ai/projects/04aee313-3c49-437a-a058-d848fe32f5e0)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/04aee313-3c49-437a-a058-d848fe32f5e0_f5fc7khi)

## Choosing a Wrapper-First Cutover Strategy

### Preserving behavior before redesign

I built this delivery rehearsal around one boundary: preserve the existing software behavior during the CI/CD cutover before redesigning the product. The migration moved execution from a Jenkins stand-in to GitLab CI while keeping the source, build script, archive rules, and expected artifact bytes unchanged. This made runner replacement the only intended variable.

I used a wrapper-first strategy so both runners called the same accepted three-stage process instead of implementing separate build logic. SHA-256 equality served as the migration contract because the cutover was expected to reproduce the same packaged program. I treated redesign as a later change with its own acceptance test because intentional product behavior would invalidate byte equality. Separating these decisions let me distinguish migration defects from approved product changes and prevented a successful redesign from hiding a broken cutover.

## Establishing a Defensible Delivery Baseline

### Setting up the delivery record

I prepared the Linux host with Docker Engine, created the private repository, and opened the Linear project before starting the timed rehearsal. The repository held the Architecture Decision Records, system topology, build plan, source, wrappers, pipeline definitions, oracle, and evidence files. Linear tracked the implementation work and the release decision outside the code history.

This setup gave the cutover a visible starting point and separated planning from execution. I documented why the migration used a shared wrapper, why artifact bytes were the acceptance contract, and why redesign remained out of scope until parity passed. Git preserved the technical sequence, while Linear recorded work status and decision closure. Neither system made the evidence immutable by itself, but together they made later changes inspectable. The baseline ensured that the migration did not invent its acceptance criteria after Jenkins and GitLab outputs were already available.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/04aee313-3c49-437a-a058-d848fe32f5e0_d22bipbk)

### Proving decisions predated the build

The file evidence/design-baseline-commit.txt recorded commit c20956d1c9f7e02a7982836084bcb1adb16dad53. That commit was titled Establish pre-build design baseline and already contained the three Architecture Decision Records, topology, and 60-minute implementation plan. None of the build or comparison evidence existed in that committed state.

I merged the baseline to main before running either delivery path, so the recorded commit preceded the artifacts and oracle results. This ordering mattered because the decisions defined what would count as parity and which differences were acceptable before the outputs were known. Git history supported the chronological claim, although repository history could still be rewritten by an authorized owner. Within this rehearsal, the saved commit identifier, file contents, and later evidence commits created the required separation between pre-build intent and post-build findings.

## Launching a Reproducible Day-Zero Lab

### Bringing up the local delivery environment

I validated the local configuration and started a four-service Docker Compose stack containing GitLab, GitLab Runner, Jenkins, and diffoscope. GitLab and Jenkins represented the new and legacy delivery paths, while diffoscope provided the independent artifact comparison. The runner connected GitLab jobs to the shared build environment used during the rehearsal.

Both paths crossed the same toolchain boundary and invoked the same repository-controlled build logic. I kept the pre-build design baseline unchanged while bringing the services online, then verified container health and tool availability before generating assessment artifacts. This created a controlled Day Zero lab where the runner could change without silently changing the compiler, archive tooling, or source inputs. The environment supported repeatable local evidence, but it did not represent production scale, availability, or security. Its purpose was to isolate cutover behavior and make differences attributable.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/04aee313-3c49-437a-a058-d848fe32f5e0_wm57u7gv)

### Fixing the toolchain and runner evidence

I captured the running toolchain in evidence/day-zero-versions.txt before the timed assessment. The record contained Go 1.27.1, GNU tar 1.35, diffoscope 240, GitLab Runner 17.11.0, and the resolved Jenkins plugin list from plugins.resolved.txt. These values tied the later results to the exact local execution environment.

The versions came from the cutover-lab-jenkins, cutover-lab-diffoscope, and gitlab/gitlab-runner:v17.11.0 images. Freezing them reduced ambiguity when the Jenkins and GitLab outputs differed because the comparison did not span unknown compiler or archive versions. The record did not prove that the container images were permanently immutable or that another host would reproduce identical behavior. It established which tools produced this rehearsal’s artifacts and gave a future investigator a concrete environment to recreate before attributing a changed digest to pipeline logic.

## Proving the Oracle Can Detect a Known Difference

### Running the intentional negative control

I created a controlled mismatch before attempting the passing parity run. The Jenkins path used the accepted build flags, while the GitLab path deliberately omitted -trimpath. Both runners still processed the same source and archive structure, allowing the oracle to evaluate whether it could detect a known byte-level difference caused by one identified configuration change.

I built both artifacts, calculated their SHA-256 digests, and ran diffoscope against the resulting archives. The comparison returned a red result and exposed 274 hunks. I preserved the hashes, detailed diff, and declared cause as negative-control evidence. This step prevented a later green result from being accepted without proving that the oracle could fail when it should. The negative control did not demonstrate migration parity and was never reported as a pass. It established sensitivity: a meaningful build difference changed the bytes, and the comparison system detected and explained that change.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/04aee313-3c49-437a-a058-d848fe32f5e0_rjoxppse)

### What the expected-red result demonstrated

The negative-control archives produced different SHA-256 digests because the GitLab build omitted -trimpath. Diffoscope reported 274 hunks, and the investigation attributed every hunk to that one deliberate omission. The oracle therefore detected the byte difference and connected it to the controlled change introduced for the test.

I classified the result as EXPECTED_RED. It was not a passing migration because the artifacts differed, and it was not an unexplained failure because the full difference had an identified cause. Recording that distinction protected the later decision from two mistakes: treating any red output as evidence of a broken system or treating an expected mismatch as acceptable parity. The negative control proved that the comparison could reject a known defect before it was used to accept equal artifacts. UNEXPLAINED_COUNT=0 meant the observed differences were accounted for, not that the migration had passed.

## Restoring Artifact Parity Across Runners

### Restoring the accepted three-stage wrapper

I restored the GitLab path to the accepted three-stage wrapper used by the Jenkins stand-in. Both runners then called the same preparation, build, and packaging stages from the repository instead of maintaining separate command sequences. The corrected build reinstated -trimpath and applied the same four deterministic tar rules used by the legacy path.

I reran both pipelines with the same source, toolchain, build script, and archive policy. The runner remained the intended variable: Jenkins executed the legacy route, while GitLab Runner executed the cutover route. Keeping shared logic in the wrapper reduced the chance that two pipeline files would drift while appearing equivalent. This restoration did not modernize the application or change its greeting. It returned the experiment to the migration contract, where success meant the new delivery system could reproduce the legacy artifact exactly before any redesign work received approval.

### Why equal digests establish migration parity

The Jenkins archive and GitLab archive produced the same SHA-256 digest. Both paths used the same source revision, shared build wrapper, fixed toolchain, -trimpath setting, and four deterministic tar rules. Only the runner changed, while the packaged bytes remained identical. The oracle recorded VERDICT=PASS with UNEXPLAINED_COUNT=0.

For this cutover, equal bytes were the acceptance contract because the stated goal was to preserve the existing program while replacing its delivery path. Matching digests proved that the compared archive contents were identical, not merely similar in behavior or size. The result did not prove that Jenkins and GitLab were equivalent in every operational characteristic, such as permissions, scheduling, logs, or recovery. It established the narrower migration claim required for release: the new runner produced the exact accepted artifact under the frozen inputs and toolchain.

## Publishing the Evidence-First Release Decision

### Closing the decision pack and delivery record

I completed the decision pack after the negative control and passing parity run were recorded. The release evidence included the pre-build design commit, Day Zero versions, Jenkins and GitLab artifact hashes, diffoscope output, expected-red explanation, passing oracle result, and the separate redesign acceptance plan. This gave reviewers one traceable record from intent through release decision.

I kept measured migration evidence separate from redesign estimates. The cutover decision depended on equal artifact bytes, while redesign remained an unmeasured future change until its branch and replacement test existed. I published the repository evidence, updated the GitHub delivery record, and closed the associated Linear work only after parity passed. This sequence prevented planning assumptions from being reported as results. The release documented what had been proven, what had intentionally failed, and what still required a different contract before implementation could be accepted.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/04aee313-3c49-437a-a058-d848fe32f5e0_56x7bhpz)

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/04aee313-3c49-437a-a058-d848fe32f5e0_wj9ju3t9)

## Demonstrating Why Redesign Requires a New Acceptance Test

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/04aee313-3c49-437a-a058-d848fe32f5e0_2f7moyz6)

### Replacing byte equality for an intentional product change

Byte equality was appropriate only while Jenkins and GitLab built the same program. The redesign intentionally changed the greeting, so its archive could not share the original SHA-256 digest. Keeping the migration gate would reject the intended product change even when the redesigned behavior was correct.

I replaced that contract with a behavioral test for the redesign branch. The test built the changed source, executed ./out/bin/hello, and required the exact output hello from the redesigned synthetic artifact. Artifact inequality became expected, while the executable result became the acceptance condition. This did not weaken the earlier parity proof because the migration had already passed under its original contract. It established a new test for a new decision. The separation ensured that cutover evidence remained about preservation, while redesign evidence measured the intentional behavior introduced afterward.

## Reflecting on the Cutover Rehearsal

### Tools and delivery-system concepts learned

I used Docker Engine to run the local services, Jenkins and GitLab as the two delivery paths, GitLab Runner to execute the cutover workflow, diffoscope to explain artifact differences, Git and GitHub to preserve the evidence record, and Linear to track the delivery decision. SHA-256 digests provided the final byte-equality check.

The main lesson was to isolate migration from redesign. A shared wrapper reduced pipeline drift, a frozen toolchain constrained the comparison, and the intentional negative control proved the oracle could detect a real difference. Equal digests then established artifact parity after the accepted wrapper was restored. I also learned that an acceptance test belongs to a specific decision. Byte equality served preservation, but it could not judge an intentional behavior change. The redesign required a separate behavioral contract rather than an exception added to the migration gate after results were visible.

### Time invested and key challenge

I completed the rehearsal in approximately 120 minutes. The most difficult part was distinguishing intentional artifact inequality from a configuration defect. During the cutover phase, unequal digests meant the migration had failed. During redesign, the same inequality was expected because the product output had changed. The surrounding acceptance contract determined how the evidence should be interpreted.

I handled that distinction by completing the byte-parity decision before introducing the redesign branch. The negative control explained all 274 hunks through the missing -trimpath flag, while the restored wrapper produced identical digests. Only then did I replace byte equality with the executable greeting test. This sequence prevented redesign work from masking an unresolved cutover problem. It also showed why labels such as EXPECTED_RED, PASS, and intentional inequality needed declared causes and retained evidence rather than informal explanations.

### Next learning goal

I completed this build to learn how to preserve behavior during a CI/CD cutover and evaluate redesign under a separate acceptance test. The evidence chain connected pre-build decisions, frozen versions, an intentional negative control, restored artifact parity, a published release decision, and a later behavioral test for the changed product.

My next goal is to build automated regression suites that test artifact consistency across operating systems and runner types. Those suites should freeze toolchain inputs, separate expected platform differences from unexplained changes, and retain comparison evidence for review. I also want to extend the wrapper contract with repeatable provenance and dependency checks without coupling them to product redesign. The governing principle will remain the same: prove that migration preserved the accepted system first, then judge intentional change against a test written for the new behavior.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/04aee313-3c49-437a-a058-d848fe32f5e0)*
