# Survive the Cutover, Then Redesign

> Inside the [Government Systems Engineering](../../README.md) portfolio · *Cloud systems engineered for federal-grade security and compliance.*

## Overview

I built this delivery rehearsal around one boundary: preserve the existing software behavior during the CI/CD cutover before redesigning the product. The migration moved execution from a Jenkins stand-in to GitLab CI while keeping the source, build script, archive rules, and expected artifact bytes unchanged. This made runner replacement the only intended variable.

I used a wrapper-first strategy so both runners called the same accepted three-stage process instead of implementing separate build logic. SHA-256 equality served as the migration contract because the cutover was expected to reproduce the same packaged program. I treated redesign as a later change with its own acceptance test because intentional product behavior would invalidate byte equality. Separating these decisions let me distinguish migration defects from approved product changes and prevented a successful redesign from hiding a broken cutover.

The architecture is built across **7 phases**, anchored by **Choosing a Wrapper-First Cutover Strategy** on the input side and **Demonstrating Why Redesign Requires a New Acceptance Test** at the end. Each phase is listed in the Implementation section below.

## Architecture

```mermaid
---
title: Survive the Cutover, Then Redesign
---
%%{init: {"theme":"base","themeVariables": {"primaryColor":"#1B4332","primaryTextColor":"#F4D03F","primaryBorderColor":"#F4D03F","secondaryColor":"#264653","tertiaryColor":"#2F5233","lineColor":"#F4D03F","fontFamily":"ui-monospace, SFMono-Regular, Menlo, Consolas, monospace","fontSize":"13px"}}}%%
flowchart TD
    classDef datastore fill:#264653,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef service fill:#1B4332,stroke:#F4D03F,stroke-width:2px,color:#F4D03F
    classDef event fill:#7B42BC,stroke:#F4D03F,stroke-width:2px,color:#FFFFFF
    classDef io fill:#0d1117,stroke:#F4D03F,stroke-width:1.5px,color:#F4D03F,font-style:italic

    Approver[/Release approver, who needs the cutover proven before redesign is discussed/]
    Investigator[/A later investigator recreating the environment before blaming pipeline logic/]

    subgraph Contract["The migration contract, chosen before any build"]
        Boundary{{Preserve existing software behavior during the cutover; redesign is a later change with its own acceptance test}}
        OneVariable{{Runner replacement is the only intended variable: source, build script, archive rules and expected bytes stay fixed}}
        Sha{{SHA-256 equality is the acceptance contract, because the cutover is expected to reproduce the same packaged program}}
        Separated{{Separating the two decisions stops a successful redesign from hiding a broken cutover}}
    end

    subgraph Baseline["A defensible delivery baseline"]
        Host(Linux host with Docker Engine, a private repository, and a Linear project opened before the timed rehearsal)
        RepoFiles[(Repository holds the decision records, topology, build plan, source, wrappers, pipeline definitions, oracle and evidence)]
        Linear[(Linear tracks implementation work and the release decision outside the code history)]
        BaselineFile[("evidence/design-baseline-commit.txt records commit c20956d, titled Establish pre-build design baseline")]
        Predates[(That committed state held three decision records, the topology and the 60-minute plan, and none of the build or comparison evidence)]
        Merged{{Merged to main before either delivery path ran, so the decisions defined parity before the outputs were known}}
        Rewritable{{Git supports the ordering claim; an authorized owner could still rewrite history}}
    end

    subgraph Lab["Day Zero lab, brought up before assessment"]
        Compose(Four-service Docker Compose stack)
        Gitlab(GitLab and GitLab Runner: the cutover delivery path)
        Jenkins(Jenkins stand-in: the legacy delivery path)
        Diffoscope(diffoscope: the independent artifact comparison)
        Versions[(evidence/day-zero-versions.txt: Go 1.27.1, GNU tar 1.35, diffoscope 240, GitLab Runner 17.11.0, resolved Jenkins plugin list)]
        Frozen{{Freezing the toolchain keeps the comparison from spanning unknown compiler or archive versions}}
        NotProd{{A controlled lab, not production scale, availability or security, and not a promise another host reproduces it}}
    end

    subgraph Negative["Negative control, run before any passing claim"]
        Omit(GitLab path deliberately omits the trimpath flag while Jenkins keeps the accepted build flags)
        Digests[(Different SHA-256 digests from the same source and archive structure)]
        Hunks[(diffoscope reports 274 hunks, every one attributed to that single omission)]
        ExpectedRed[(Classified EXPECTED_RED with UNEXPLAINED_COUNT=0: differences accounted for, not a passing migration)]
        Sensitivity{{Proves the oracle can reject a known defect before it is trusted to accept equal artifacts}}
        TwoMistakes{{Guards the later decision from reading any red as a broken system, or an expected mismatch as parity}}
    end

    subgraph Parity["Parity restored across runners"]
        Wrapper(Accepted three-stage wrapper restored: preparation, build and packaging called from the repository by both runners)
        Reinstated[(The trimpath flag reinstated and the same four deterministic tar rules applied)]
        NoDrift{{Shared logic in the wrapper stops two pipeline files from drifting while appearing equivalent}}
        Equal[(Both archives produce the same SHA-256 digest with only the runner changed)]
        Pass[(Oracle records VERDICT=PASS with UNEXPLAINED_COUNT=0)]
        NarrowClaim{{Proves identical archive contents, not that the two systems match on permissions, scheduling, logs or recovery}}
    end

    subgraph Decision["The published release decision"]
        Pack[(Decision pack: design commit, Day Zero versions, both artifact hashes, diffoscope output, the expected-red explanation, the passing oracle result, and the separate redesign plan)]
        Order(Repository evidence published, delivery record updated, and the Linear work closed only after parity passed)
        NoEstimates{{Measured migration evidence kept apart from redesign estimates, so planning assumptions are never reported as results}}
    end

    subgraph Redesign["A new decision needs a new test"]
        Changed(Redesign branch intentionally changes the greeting, so its archive cannot share the original digest)
        WouldReject{{Keeping the migration gate would reject the intended change even when the new behavior is correct}}
        Behavioral[(Behavioral test: build the changed source, execute ./out/bin/hello, require the exact expected output)]
        Inequality{{Artifact inequality becomes expected and the executable result becomes the acceptance condition}}
        NotWeaker{{The earlier parity proof stands, because the migration already passed under its own contract}}
    end

    Approver -- "requires" --> Boundary
    Boundary -- "held by" --> OneVariable
    Boundary -- "measured by" --> Sha
    Boundary -- "kept apart by" --> Separated
    Boundary -- "recorded on" --> Host
    Host -- "carries" --> RepoFiles
    Host -- "tracked beside" --> Linear
    RepoFiles -- "sealed by" --> BaselineFile
    BaselineFile -- "contained" --> Predates
    Predates -- "held under" --> Merged
    Merged -- "bounded by" --> Rewritable
    Host -- "runs" --> Compose
    Compose -- "provides" --> Gitlab
    Compose -- "provides" --> Jenkins
    Compose -- "provides" --> Diffoscope
    Compose -- "captured in" --> Versions
    Versions -- "gives" --> Frozen
    Compose -- "bounded by" --> NotProd
    Versions -- "is the concrete environment handed to" --> Investigator
    Merged -- "precedes" --> Omit
    Gitlab -- "first run as" --> Omit
    Jenkins -- "compared against" --> Omit
    Omit -- "produced" --> Digests
    Digests -- "explained by" --> Hunks
    Hunks -- "classified as" --> ExpectedRed
    ExpectedRed -- "establishes" --> Sensitivity
    ExpectedRed -- "prevents" --> TwoMistakes
    Sensitivity -- "earns the right to run" --> Wrapper
    Wrapper -- "restores" --> Reinstated
    Wrapper -- "gives" --> NoDrift
    Reinstated -- "rerun on both runners, giving" --> Equal
    Diffoscope -- "and the digests confirm" --> Equal
    Equal -- "recorded as" --> Pass
    Pass -- "bounded by" --> NarrowClaim
    Sha -- "is the contract satisfied by" --> Pass
    Pass -- "assembled into" --> Pack
    ExpectedRed -- "assembled into" --> Pack
    Versions -- "assembled into" --> Pack
    BaselineFile -- "assembled into" --> Pack
    Pack -- "released through" --> Order
    Order -- "held under" --> NoEstimates
    Order -- "closes" --> Linear
    Pack -- "handed to" --> Approver
    Pack -- "inspectable by" --> Investigator
    Separated -- "is why" --> Changed
    Changed -- "would be blocked, since" --> WouldReject
    WouldReject -- "replaced by" --> Behavioral
    Behavioral -- "inverts the rule, so" --> Inequality
    Inequality -- "bounded by" --> NotWeaker

    class RepoFiles,Linear,BaselineFile,Predates,Versions,Digests,Hunks,ExpectedRed,Reinstated,Equal,Pass,Pack,Behavioral datastore
    class Host,Compose,Gitlab,Jenkins,Diffoscope,Omit,Wrapper,Order,Changed service
    class Boundary,OneVariable,Sha,Separated,Merged,Rewritable,Frozen,NotProd,Sensitivity,TwoMistakes,NoDrift,NarrowClaim,NoEstimates,WouldReject,Inequality,NotWeaker event
    class Approver,Investigator io
```

The diagram shows the topology and data flow of the system as built. The full architectural narrative, with screenshots and prose, lives in [`documents/byte-parity-cutover-rehearsal.md`](./documents/byte-parity-cutover-rehearsal.md).

## Implementation

This system is built across **7 phases**:

1. **Choosing a Wrapper-First Cutover Strategy**
2. **Establishing a Defensible Delivery Baseline**
3. **Launching a Reproducible Day-Zero Lab**
4. **Proving the Oracle Can Detect a Known Difference**
5. **Restoring Artifact Parity Across Runners**
6. **Publishing the Evidence-First Release Decision**
7. **Demonstrating Why Redesign Requires a New Acceptance Test**

For the full walkthrough with screenshots and step-by-step content, see [`documents/byte-parity-cutover-rehearsal.md`](./documents/byte-parity-cutover-rehearsal.md).

## Validation

Each build phase below is documented in [`documents/byte-parity-cutover-rehearsal.md`](./documents/byte-parity-cutover-rehearsal.md), with screenshots, configuration, and notes as captured during the build:

- ✅ Choosing a Wrapper-First Cutover Strategy
- ✅ Establishing a Defensible Delivery Baseline
- ✅ Launching a Reproducible Day-Zero Lab
- ✅ Proving the Oracle Can Detect a Known Difference
- ✅ Restoring Artifact Parity Across Runners
- ✅ Publishing the Evidence-First Release Decision
- ✅ Demonstrating Why Redesign Requires a New Acceptance Test
