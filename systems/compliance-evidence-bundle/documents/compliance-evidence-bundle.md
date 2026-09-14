<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Compliance Evidence Bundle Pipeline

**Project Link:** [View Project](https://nextwork.ai/projects/688850c1-2d04-4180-8410-879423a8a566)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/688850c1-2d04-4180-8410-879423a8a566_3m90rr5l)

## Building a Contract-Enforced Compliance Evidence Bundle

### Project goals and reviewer independence

I built this system to package security evidence into a contract-enforced bundle that a reviewer could inspect without depending on my explanation. The bundle joined six declared evidence slots with their status, source, digest, and limitation. A reviewer could validate its structure, recompute hashes, inspect the included artifacts, and identify which evidence remained incomplete.

Reviewer independence shaped the acceptance criteria. The pipeline could not treat a generated file, successful command, or builder statement as proof by itself. Each artifact needed a declared place in the schema and a verifiable relationship with the published bundle. I also separated technical integrity from assessor judgment. The implementation could prove that files matched their recorded digests and that required slots were present. It could not decide whether an external assessor would accept those artifacts as sufficient compliance evidence.

### Installing the security toolchain

I installed Syft, Grype, cosign, SeaweedFS, and the required Python dependencies before generating evidence. Each tool had a defined responsibility. Syft created the software bill of materials, Grype scanned the resulting package inventory for known vulnerabilities, cosign produced signing and provenance material, and SeaweedFS provided the local S3-compatible object store used for publication and retrieval.

I verified that each command was available before the assembly run so a missing executable would fail during setup instead of producing an incomplete bundle later. Python handled schema validation, manifest construction, digest calculation, publication checks, and retrieval verification. This toolchain established the measured execution environment for the build. It did not prove that later tool versions would produce identical findings, so the captured outputs and version context remained part of the evidence required to reproduce or investigate the result.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/688850c1-2d04-4180-8410-879423a8a566_u0eq9283)

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/688850c1-2d04-4180-8410-879423a8a566_0pyrw08k)

## Writing the Contract Before the Artifacts

### Defining the six-slot schema and NIST control questions

I defined the six-slot JSON schema before generating the bundle. The contract named sbom, vuln-scan, build-env, provenance, hardening-check, and control-map as required evidence categories. Each slot needed a declared status and the metadata required by the publisher. This prevented the assembler from silently omitting evidence that was unavailable or inconvenient to produce.

I also wrote the control questions before examining the finished artifacts. The questions came from the system’s selected NIST SP 800-53 Rev. 5 control context and established what a reviewer should be able to determine from the bundle. The schema enforced structural completeness, while the questions tested whether the evidence was useful to a reader. Neither mechanism declared the system compliant. The contract proved whether the bundle followed its predefined shape, and the questions exposed where evidence supported an answer, remained stubbed, or required external assessment.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/688850c1-2d04-4180-8410-879423a8a566_n5zb5wp7)

### Why commit order matters for control integrity

I committed the contract and control questions before producing the artifacts. This ordering reduced the opportunity to rewrite the acceptance criteria around whatever the tools happened to generate. If I had written the questions afterward, I could have selected only those that the existing evidence answered and presented that fit as intentional design.

The control source also mattered. NIST SP 800-53 Rev. 5 was an external catalog that I did not control, so it provided a stable reference for framing the questions. Git history showed that the contract preceded the assembled bundle and supported the claim that the standard was set first. That history was evidence of sequence, not absolute proof of integrity, because a repository owner can rewrite commits and dates. Stronger enforcement would require protected branches, signed commits, remote audit logs, or an independent timestamp. I recorded that limit instead of treating ordinary Git history as immutable.

## Assembling and Publishing the Bundle

### Generating four real artifacts from one build run

I automated one build run that generated four evidence artifacts from the same execution context. Syft produced the SBOM, Grype generated the vulnerability scan, the pipeline captured the build environment, and cosign created the provenance material. The assembler recorded each artifact in its declared slot and calculated the digest needed to bind the files to the bundle manifest.

Generating the four artifacts together reduced ambiguity about which build each file described. The pipeline did not combine unrelated scans from different execution periods and present them as one evidence package. Before publication, the contract validator checked all six slots, including those without completed evidence. The assembler then created the bundle and calculated its SHA-256 digest. This process established structural completeness and content identity for the packaged files. It did not prove that every scanner finding was correct or that the build satisfied an assessor’s control interpretation.

### READY and STUBBED slot breakdown

The final manifest contained four READY slots and two STUBBED slots. The ready evidence consisted of sbom, vuln-scan, build-env, and provenance. Each represented an artifact generated during the build and included in the validated package. The two stubbed entries were hardening-check, identified as linux-ci-pipeline-run-001, and control-map, identified as control-map-population-001.

I retained the stubbed entries instead of deleting them or marking them ready without evidence. Their presence kept the six-slot contract complete while making the missing work visible to the reviewer. STUBBED did not mean failed, passed, or waived. It meant the slot existed, but the supporting artifact was not produced within this run. This distinction prevented a structurally valid bundle from being mistaken for a fully populated compliance submission. The manifest showed exactly which evidence was available and which work still required completion.

### How the hash verification gate protects integrity

The publication gate calculated the bundle’s SHA-256 digest and used that value as its content identity. During retrieval, the script downloaded the object, recalculated the digest, and compared it with the requested key. A changed byte would produce a different hash and cause verification to fail. This connected the retrieved package to the exact content accepted by the publisher.

SeaweedFS availability required separate evidence. Four earlier start attempts failed: one panicked while using -dir=.\seaweedfs-data, and the other bindings never exposed a healthy S3 endpoint. The later weed mini process remained active, and http://localhost:8333 returned HTTP 200. The published bundle remained on that running store. The hash gate proved content integrity after retrieval, while the health check showed that the local service was reachable. Neither result proved long-term durability, external replication, or protection against deletion by an administrator.

## Proving the Publish Gate Refuses

### Designing the negative test cases

I created negative cases that removed or damaged required evidence before publication. Each test began from the same contract and changed one condition so the resulting refusal could be attributed to that defect. The publisher had to stop before upload, identify the failed requirement, and avoid producing a misleading content key for an invalid package.

The missing-artifact test proved that naming a slot in the manifest was insufficient when its required file did not exist. Additional checks covered schema and digest expectations so the gate evaluated both structure and content. I recorded the predicted outcome before each run, then compared it with the returned result. A refusal only counted when it matched the intended defect; an unrelated crash would not prove the contract worked. After each negative test, I restored the original files and reran publication. This recovery check distinguished a specific contract refusal from a publisher that was simply broken.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/688850c1-2d04-4180-8410-879423a8a566_abws7jnj)

### What three publish runs together prove about the contract

The three publication runs showed that the gate could reject invalid inputs and accept the restored contract. I predicted that the final run would succeed and print [PUBLISHED]. After the original files were restored, every validation passed and the publisher returned [PUBLISHED] Bundle sha256:0215a0fe7efdf0057c020e860d3eb4c9dcf3b20a5d1eb0bb1f1b33735059db04.

The successful third run mattered because the earlier refusals alone could have come from a permanently broken publisher. Restoration demonstrated that the failures were tied to the introduced defects rather than general execution failure. Together, the runs proved conditional behavior for the tested cases: invalid bundles were stopped, while the original six-slot bundle was accepted. They did not prove that every possible malformed input would be rejected. Broader assurance would require additional schema mutations, corrupted digest cases, duplicate-slot tests, and authorization failures at the storage boundary.

## Content-Addressed Storage and Digest Verification

### Publishing two bundles and writing the retriever

I published a second bundle with different content so the object store contained two distinct SHA-256 keys. This tested whether publication preserved immutable content identities rather than replacing one fixed object named “latest.” The second assembly overwrote the local artifacts/ directory, but it produced a new digest instead of reusing the first bundle’s key.

I then wrote a retrieval script that accepted a digest, downloaded the corresponding object from SeaweedFS, recalculated its SHA-256 value, and compared the result with the requested digest. Retrieval succeeded only when the content matched that key. This created a verifiable path from a recorded digest to a specific stored package. The test established that both published objects could be addressed independently within the running local store. It did not prove geographic replication, retention after storage loss, access control strength, or immutability against an administrator who could delete objects.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/688850c1-2d04-4180-8410-879423a8a566_sq0c0a1r)

### Why the first bundle's digest is the stronger claim

The first bundle’s digest became the stronger test after the second assembly replaced the contents of the local artifacts/ directory. At that point, retrieving the newest digest would only show that the store could return content matching the current workspace. Retrieving the older digest tested whether an earlier object remained independently addressable after a later publication.

The retriever requested the first SHA-256 key, downloaded its object, and confirmed that the recalculated digest matched. That result showed that SeaweedFS retained the earlier package by content identity rather than redirecting every request to the latest upload. It also separated local workspace state from stored evidence: the first bundle no longer depended on files still present under artifacts/. The claim remained bounded to the tested store and session. It did not establish permanent retention, legal immutability, backup recovery, or protection from privileged deletion.

## Acceptance Test: A Reader Answers the Control Questions Cold

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/688850c1-2d04-4180-8410-879423a8a566_efgs4yuw)

### Reader proximity assessment and honest limits

I assessed whether the reader could answer the control questions independently from the bundle. The result showed that the citer was too close to the implementation because the builder opened the same files produced by the assembler. That proximity created a risk that prior knowledge, rather than the evidence package alone, shaped the answers. I therefore did not present the exercise as a clean independent review.

I recorded two additional limits. First, the bundle was an input to a compliance submission, not the submission itself. The pipeline could package and verify evidence, but an assessor’s acceptance remained an external judgment. Second, hardening-check stayed STUBBED because OpenSCAP required Linux while the workstation ran Windows. These limits did not invalidate the completed artifacts, but they constrained the conclusion. A stronger acceptance test would use a reader who had not built the system and a Linux CI run that populated the missing hardening evidence.

## Reflections and Key Takeaways

### Tools and concepts from the evidence bundle pipeline

I used Syft to generate the SBOM, Grype to scan for known vulnerabilities, cosign to create signing and provenance material, and SeaweedFS to store bundles through an S3-compatible interface. Python enforced the JSON Schema, assembled the package, calculated SHA-256 digests, controlled publication, and verified retrieved content. Git preserved the order between the contract and resulting evidence.

The main lesson was that compliance evidence needed both structure and integrity. The six-slot contract prevented missing categories from disappearing, while READY and STUBBED states exposed what existed. Content addressing tied each published object to its bytes, and negative tests proved that the gate refused specific defects. I also learned that reviewer proximity affects the strength of an acceptance test. The pipeline made evidence inspectable and traceable, but it did not convert tool output into an assessor’s compliance determination.

### Time and challenges

I completed the build in approximately 60 minutes. The hardest part was documenting reader proximity honestly while keeping the bundle’s proven value clear. The reader could answer the questions, but the same person had inspected the assembler’s source files. That made the exercise useful as an internal check but too close to count as independent review.

I also had to preserve the meaning of the two STUBBED slots. Omitting them would make the package appear complete by shrinking the contract, while marking them READY would claim evidence that did not exist. Keeping hardening-check and control-map visible protected the reviewer from that ambiguity. The finished implementation taught me how to package technical outputs into a digest-keyed evidence bundle without overstating what the package proved. My next step is to run this pipeline in CI/CD, add Linux-based OpenSCAP evidence, and test the bundle with an independent reader.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/688850c1-2d04-4180-8410-879423a8a566)*
