<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Contract-First Go API Delivery

**Project Link:** [View Project](https://nextwork.ai/projects/f7da5bbd-e8ad-4537-81b4-728bbc132039)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/pineapple_king_overclocked_the_ai/uploads/f7da5bbd-e8ad-4537-81b4-728bbc132039_wifchgmz)

## Establishing the Contract-First Delivery Rule

### Why evidence must precede implementation

I built a contract-first delivery system for a Go records service. The sequence began with written architecture decisions, continued through a validated OpenAPI promise and fail-closed implementation, and ended with delivery controls designed to identify contract drift before deployment. Each stage produced evidence that could be inspected without relying on a demonstration or an undocumented explanation.

The evidence boundary extended from the first repository checkpoint to the planned immutable ECR image digest. This order mattered because later code could not redefine what the service had originally promised. Missing routes, undocumented routes, invalid responses, absent configuration, and publishing defects became named failures with retained outputs. The result reduced handover uncertainty by showing what was decided, what was implemented, what passed, and what had not yet run.

## Creating a Traceable Delivery Foundation

### Setting up the delivery rail and Linux tooling

I prepared the Linux development environment, initialized the repository, and created the structure for documentation, evidence, contract tests, service code, and delivery automation. The first stage focused on the delivery rail rather than endpoint implementation. It established the Go module, ignore rules, evidence index, and seven design documents needed to guide later work.

I committed this foundation before adding the OpenAPI file, handlers, or executable service. That ordering made the repository history part of the evidence. Reviewers could see the architecture and acceptance rules before implementation began, then compare later commits against those decisions. The workspace also separated plans, source code, contract fixtures, pipeline files, and captured results so that each claim could point to a specific artifact.

![Image](https://nextwork.ai/pineapple_king_overclocked_the_ai/uploads/f7da5bbd-e8ad-4537-81b4-728bbc132039_9kjcsims)

### Proving implementation had not started

The repository contained one commit at the pre-build checkpoint: 1c731c2. That commit included .gitignore, go.mod, seven design documents, and evidence/index.md. It did not contain openapi.yaml, an internal/handlers directory, or a cmd/recordsvc executable. The recorded tree therefore showed planning and module setup without service implementation.

This checkpoint established a clear boundary between decisions and code. The Git history proved that the architecture, evidence plan, and delivery rules were committed before a handler or API contract implementation existed. Later files could be traced to later commits rather than appearing as though they had always been present. That sequence made it possible to verify that implementation followed the recorded plan instead of allowing the plan to be rewritten around completed code.

## Defining the API Promise Before Implementation

### Writing the OpenAPI contract first

I wrote the OpenAPI document for the service’s two API paths before creating any handler code. The contract defined the approved routes, operations, response expectations, and schemas that the Go router would later have to satisfy. This made the external interface the starting promise rather than a description produced after the service already worked.

The contract became the basis for both implementation and testing. A documented route without a router registration would represent an unfulfilled promise, while a registered route missing from OpenAPI would represent unapproved behavior. Response validation also ensured that matching route names were not enough when returned data violated the schema. By fixing these expectations first, later code changes had a stable interface boundary against which drift could be detected.

![Image](https://nextwork.ai/pineapple_king_overclocked_the_ai/uploads/f7da5bbd-e8ad-4537-81b4-728bbc132039_w3xgaj7y)

### Validating the contract before handlers exist

I validated tests/contract/openapi.yaml with kin-openapi v0.149.0 before creating any Go handlers. The validator completed successfully while the repository still had no internal/handlers directory and no cmd/recordsvc program. This proved that the OpenAPI document could stand on its own as a valid contract rather than depending on implementation code.

The timing of the validation was part of the delivery evidence. Because no handler package or executable existed, service behavior could not have influenced the validator’s result. The passing output showed that the promise was structurally valid before implementation began. Later handlers and router registrations therefore had to follow an already accepted interface, and any mismatch belonged to the implementation or required an explicit contract revision rather than a quiet rewrite.

## Building a Fail-Closed Records Service

### Implementing contracted synthetic-record endpoints

I created deterministic synthetic-record models that matched the OpenAPI schemas, then implemented the contracted service behavior around those models. The fixed dataset made endpoint results repeatable and allowed route, response, and schema checks to compare known values rather than changing external data. The implementation remained focused on the two approved API paths and their declared response shapes.

I then connected the models to Go handlers and the router without adding behavior outside the published contract. The deterministic records supported repeatable tests because the same request produced the same response content. This made failures easier to attribute to routing, serialization, schema, or configuration rather than an unstable data source. The service implementation became one fulfillment of the OpenAPI promise, not the authority that defined the promise after delivery.

![Image](https://nextwork.ai/pineapple_king_overclocked_the_ai/uploads/f7da5bbd-e8ad-4537-81b4-728bbc132039_1xbwp7rw)

### Demonstrating required configuration and exit 78

The service read LISTEN_ADDR before constructing the server. When that variable was missing, startup stopped with exit code 78 instead of selecting a default address. I captured the run in evidence/config-exit.txt, which named LISTEN_ADDR and recorded exit_status=78. The service did not continue far enough to create or start a listener.

This behavior proved that configuration failed closed at the startup boundary. An operator had to provide the listen address explicitly, so the service could not invent a network binding that differed from the intended environment. The retained output connected the missing input to a specific exit status and named error. It also demonstrated that configuration validation occurred before server construction, preventing an incomplete setup from becoming a running service.

## Making Contract Drift a Build Failure

### Testing routes and responses in both directions

I built a bidirectional contract gate that compared the OpenAPI paths with the Go router. The first direction checked that every documented operation had a matching registration. The second checked that every registered route appeared in the contract. This prevented both dropped promises and undocumented behavior from passing simply because the service compiled.

The gate also validated response bodies against the declared schemas. Matching route names alone could not pass if an endpoint returned the wrong data type or response structure. These checks turned contract drift into a build failure before deployment. The delivery process could now distinguish missing implementation, extra implementation, and schema disagreement as separate defects, giving reviewers clear evidence of which boundary had been violated.

![Image](https://nextwork.ai/pineapple_king_overclocked_the_ai/uploads/f7da5bbd-e8ad-4537-81b4-728bbc132039_oy6mg6i4)

### What documented-only, router-only, and schema failures revealed

I retained three deliberate red cases. The documented-only case kept GET /v1/records/{} in OpenAPI while removing its router registration, proving that the gate detected a promise the service had dropped. The router-only case temporarily registered GET /v1/debug without documenting it, proving that unapproved behavior could not enter the service unnoticed.

The schema case changed the /status response to a numeric value even though the contract required a string. The route still existed on both sides, but the gate rejected the response because route parity alone did not satisfy the interface promise. Together, these failures demonstrated three distinct protections: all documented routes must exist, all registered routes must be approved, and each returned body must match its declared schema.

## Publishing an Immutable Image with Federated Access

### Rehearsing keyless publishing, evidence release, and teardown

I prepared the GitLab CI/CD path to use OIDC federation for temporary AWS access instead of stored access keys. The planned job built the Go service, ran the contract and configuration checks, created a shell-free container image, scanned the release evidence, and published the accepted image to Amazon ECR. The release identity was intended to be the immutable image digest rather than a mutable tag alone.

I also defined the teardown checks needed to remove the test repository and read it back as not found. At this recorded stage, the publishing job had not run, so no image digest, publish output, GitLab identity evidence, scan result, deletion proof, or three not-found read-backs could be claimed. The rehearsal established the delivery path and expected evidence while keeping planned controls separate from completed results.

![Image](https://nextwork.ai/pineapple_king_overclocked_the_ai/uploads/f7da5bbd-e8ad-4537-81b4-728bbc132039_et8pmm0q)

### Proving temporary AWS credentials replaced stored keys

The release evidence available at this stage covered the contract, configuration failure, deliberate red runs, lint checks, and AWS bootstrap files. Those artifacts matched the prepared release state. The image, GitLab identity, publish, scan, GitLab deletion, and digest evidence did not yet exist because the publishing job had not executed.

The teardown file contained only an earlier missing-repository error and did not contain the required three not-found read-backs. I therefore did not treat it as proof of completed cleanup. The OIDC configuration described how GitLab would obtain temporary AWS credentials, but the absent publish evidence limited the claim. This distinction prevented planned keyless access and teardown behavior from being reported as measured delivery results before the job ran.

## Hardening Route Normalization Against False Drift

![Image](https://nextwork.ai/pineapple_king_overclocked_the_ai/uploads/f7da5bbd-e8ad-4537-81b4-728bbc132039_qusi8rc6)

### Normalization edge case that protects the gate

I added table-driven tests for route parameters whose names differed while their path position remained the same. The gate normalized {id}, {recordId}, and {record_id} to GET /v1/records/{}. It also reduced the regex form {id:[0-9]+} to the same route key, treating the parameter pattern as part of matching behavior rather than a separate API path.

This normalization prevented false drift when only a placeholder name or inline regex changed. Without it, the gate could report one missing route and one extra route even though both descriptions referred to the same endpoint shape. The test preserved meaningful comparison by focusing on method and path structure. It still allowed actual route additions or removals to fail while ignoring spelling differences that did not create a new public route.

## Reflecting on Contract-First Delivery

### Tools and delivery concepts applied

I used Go 1.27.1, Docker Engine with WSL 2, GitHub for version-controlled delivery, GitLab CI/CD with OIDC for planned keyless AWS access, and Amazon ECR for digest-addressed image storage. Kin-openapi v0.149.0 validated the specification, while Go tests exercised routes, schemas, configuration behavior, and normalization cases.

The build applied contract-first sequencing, bidirectional route comparison, response-schema checks, fail-closed configuration, table-driven normalization tests, and a planned shell-free image release. I also practiced separating prepared delivery controls from completed evidence. The contract, service, red runs, and normalization checks were measured, while the unpublished image digest and incomplete teardown remained outside the final proof.

### Project duration and key challenge

I completed the recorded build in approximately 65 minutes. The most difficult part was writing normalization logic that treated renamed path parameters, regex parameter forms, and trailing slash differences consistently. The gate needed to reject real route drift without producing failures for equivalent endpoint shapes expressed with different placeholder syntax.

I addressed that challenge with table-driven cases that mapped the supported forms to one normalized route key. This made the comparison rule executable and repeatable rather than dependent on manual judgment. The tests demonstrated that {id}, {recordId}, {record_id}, and {id:[0-9]+} represented the same path position while leaving genuine documented-only or router-only differences visible to the contract gate.

### Next steps in the learning journey

I completed this work to practice building a contract-first Go API whose OpenAPI promise preceded its implementation. The build covered pre-code evidence, specification validation, deterministic records, required configuration, bidirectional route checks, schema failures, and route normalization. It also prepared an OIDC-based publishing path while preserving the fact that the final publish had not run.

My next goal is to add automated container security scanning before an image can be accepted for registry publication. The scan should produce retained evidence, fail the delivery job when defined findings exceed the accepted boundary, and remain tied to the same immutable digest that passed the contract tests. I also need to complete the keyless publish and teardown runs so their planned controls become measured release evidence.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/f7da5bbd-e8ad-4537-81b4-728bbc132039)*
