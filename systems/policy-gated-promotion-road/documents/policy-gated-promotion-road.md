<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build Policy-as-Code Promotion Gates

**Project Link:** [View Project](https://nextwork.ai/projects/d0bf5d09-4205-4194-a10b-cd8e17f15b1c)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/builtwithroy/uploads/d0bf5d09-4205-4194-a10b-cd8e17f15b1c_bcc0n7ry)

## Establishing Command-Backed Delivery Governance

### Building a trustworthy promotion system

I built a five-rung promotion road so delivery engineers and the Information System Security Officer could evaluate every move from development to production using the same written policy. The road defined dev, integration, test, staging, and production as ordered environments rather than informal labels. I owned the map, promotion command, validation logic, and evidence while preserving all thirteen supplied starter files exactly as provided.

The result was a governance process backed by executable decisions instead of verbal checkpoints. A single table showed the requirements attached to every rung, while command output recorded which requests passed and why unsafe requests were refused. This gave the change board an inspectable trail connecting the declared policy, submitted promotion request, validator result, and released source without suggesting that unobserved pipeline activity had occurred.

## Making the Missing Enforcement Boundary Visible

### Observing the starter bundle’s missing policy engine

I inspected the starter bundle before writing any enforcement code so I could separate missing implementation from environmental problems. I confirmed that Git, Go 1.27, and Docker were available, pulled the required Go container image, and identified the thirteen supplied files that had to remain unchanged. I then ran the baseline build and preserved its output as evidence of the repository’s original state.

The build failed because road.Lint and road.Check were undefined. That was an expected and useful failure, not a sign that the starter files or toolchain were defective. The existing command structure already called the two missing functions, making the unfinished boundary visible. By recording that state first, I could later show precisely what my validator files added and avoid taking credit for code that already existed in the supplied bundle.

![Image](https://nextwork.ai/builtwithroy/uploads/d0bf5d09-4205-4194-a10b-cd8e17f15b1c_my3hiawo)

### Why the planned failure proves the enforcement boundary

The compiler errors explicitly named road.Lint and road.Check as undefined, while the surrounding starter code was recognized and compiled far enough to reach those missing symbols. That narrowed the implementation gap to the policy validators. It also established that the expected command path already existed, so I did not need to redesign the starter application or alter the supplied renderer simply to produce a successful build.

This planned failure became the before-state for the project. Once I added the missing validators, a clean build would demonstrate that the identified gap had been closed. More importantly, later PASS and refusal outputs would show that the new functions did more than satisfy the compiler. They would prove that the boundary could inspect the road, validate a promotion request, name policy violations, and return an outcome that another engineer could reproduce.

## Building a Five-Rung Promotion Road

### Creating one map and one promotion command

I implemented the five-rung road and the two missing validators while keeping the thirteen supplied files unchanged. The map defined dev, integration, test, staging, and production in one ordered structure, giving each environment a consistent place for required evidence, allowed data, approval rules, and separation-of-duty checks. Keeping those requirements in one map prevented the command and the table from developing different interpretations of the road.

After the implementation, the build completed without output, lint printed PASS lint: 5 rungs, and the supplied renderer printed one table row for each rung. The same command surface could now lint the map, check a promotion request, or display the road for a reviewer. That made the policy both readable and executable: people could inspect the table, while automation could evaluate the same definitions and refuse a request before an unsafe promotion advanced.

![Image](https://nextwork.ai/builtwithroy/uploads/d0bf5d09-4205-4194-a10b-cd8e17f15b1c_qb5vxbr4)

## Proving Promotion Paths and Governance Boundaries

### Testing every passing promotion request

I tested every permitted one-rung promotion rather than demonstrating only a single successful example. The four supplied requests covered dev to integration, integration to test, test to staging, and staging to production. Each request passed through the same command and policy map, showing that the road supported the complete intended route without requiring a special command or separate set of rules for each environment.

I also ran the required refusal set before merging the policy branch. Only after the valid requests passed and the invalid requests produced the expected explanations did I merge the change and close the planning ticket. This order connected implementation, validation, review, and planning status in one history. It showed that the ticket was closed because the command-backed acceptance evidence existed, not merely because the validator code had been written.

### Demonstrating mandatory policy refusals

The refusal run covered five policy failures: skipping a rung, using real data on the test rung, submitting a manual gate with no approver, allowing the production initiator to approve the same move, and omitting the change-board chair. Each output named the relevant environment, evidence item, or person, giving the operator enough information to correct the request instead of returning an unexplained rejection.

The production request was especially important because one approval problem did not hide the other. The validator reported both the separation-of-duty conflict and the missing change-board role in the same run. Together, these cases showed that the command checked order, data classification, evidence completeness, and human approval boundaries. The test did not prove every possible governance rule, but it proved the specific controls declared by this five-rung road.

![Image](https://nextwork.ai/builtwithroy/uploads/d0bf5d09-4205-4194-a10b-cd8e17f15b1c_u2yxxmql)

## Shipping an Evidence-Ready Handover

### Packaging the release, readouts, and operational documentation

I converted the command results into an operational handover containing two readouts, a fidelity note, and eight supporting documents. The package explained the five-rung road, showed the accepted and refused requests, and identified the three policy files that had passed the checks. I kept the implementation evidence separate from the supplied source so another engineer could see what changed without losing the original baseline.

The local pipeline had not been run during the observed session, so both readouts and the private v1.0.0 release stated NOT REHEARSED. That wording preserved the boundary between evidence I had personally seen and activity that might exist elsewhere. I also confirmed that the release commit matched the handover state. The release therefore documented a checked policy implementation while making no unsupported claim that the full delivery pipeline had executed successfully.

## Extending Evidence Checks Without Changing the Release

![Image](https://nextwork.ai/builtwithroy/uploads/d0bf5d09-4205-4194-a10b-cd8e17f15b1c_77ux7jvh)

### Using a temporary request to preserve released source

I tested a narrower evidence failure without editing the request that had already shipped. I copied the integration-to-test request into a temporary file and removed only dast-report. The promotion command refused the copy because that required item was missing, giving me a focused demonstration that the validator checked individual evidence requirements rather than accepting a request merely because its source and destination were valid.

After capturing the refusal, I deleted the temporary request and ran the original integration-to-test request again. The original still passed, confirming that the experiment had not altered the released source or its accepted outcome. This approach protected release fidelity while extending the evidence set. A reviewer could compare the temporary failure with the unchanged passing request and identify the exact missing item that caused the different result.

## Reflecting on Policy-as-Code Delivery Practice

### Tools and governance concepts applied

I used Go to implement the policy checks, Git to preserve the design and implementation history, Docker to provide the required build environment, and a private GitHub repository for the reviewed release. Linear held the planning ticket that moved to completion after the checks passed. Each tool served a distinct purpose, keeping source, execution evidence, release state, and planning status traceable without treating them as interchangeable proof.

The central concepts were one ordered promotion map, one command surface for linting, checking, and rendering, and refusals that named the failing rung, evidence item, or approval role. I also practiced evidence fidelity by writing NOT REHEARSED when pipeline logs were unavailable. That restraint mattered because a trustworthy handover must describe what was observed, identify what remains untested, and avoid turning an assumption into release evidence.

### Project timing and challenges

The project took about 59 minutes, matching the estimate on the project page. That time covered inspecting the starter bundle, recording the planned build failure, implementing both validators, exercising the four permitted moves, running the refusal cases, and preparing the release documentation. The work stayed focused because the existing command structure and renderer already defined where the missing policy behavior belonged.

The hardest part was preserving all thirteen supplied files while making the two new validators compile cleanly against their existing types and command paths. I also had to keep the evidence precise when the local pipeline remained unobserved. If I repeated the project, I would reserve a separate window for rehearsing that pipeline and recording its commands, timestamps, and exit results rather than allowing the release deadline to leave that area marked NOT REHEARSED.

### Next steps in delivery governance

I completed this project to learn how a delivery road can reject an unsafe promotion through a repeatable command rather than relying on someone to remember a rule during a meeting. The exercise showed that the map should remain the only place where gate requirements are defined. The command, table, tests, and documentation can then read from that source instead of maintaining separate versions of the policy.

My next learning goal is to connect the same checks to a real change-management system that records authenticated approval events rather than trusting names written in request files. I also want to rehearse the local pipeline and extend the evidence model without weakening the existing separation-of-duty rules. Any extension should preserve the same principle: policy changes are reviewed in the map, while promotion requests are evaluated against it.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/d0bf5d09-4205-4194-a10b-cd8e17f15b1c)*
