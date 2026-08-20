<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Zero-Cost Federal Platform Rehearsal Rig

**Project Link:** [View Project](https://nextwork.ai/projects/34c557ca-b856-453b-a1e7-2176c6590695)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/34c557ca-b856-453b-a1e7-2176c6590695_rgq11weh)

## The Mission: A Zero-Cost Federal Rehearsal Rig

### Defining the rig and its limits

I built a zero-cost federal platform rehearsal rig to test whether the planned delivery path could work before funding. The rig runs a three-zone promotion flow, object storage, database migrations, and pipeline stages in a local Kubernetes environment. It provides evidence that the components can connect and that the delivery sequence can be rehearsed.

The rig must never be treated as proof of production readiness. It does not reproduce federal cloud accounts, managed services, security boundaries, operational staffing, or provisioning delays. Its purpose is narrower: expose design problems early, verify the pipeline shape, and create a working reference for future funding decisions. The fidelity map records where the local build matches the intended platform and where it does not.

## Designing Before Building: Decision Records and Delivery Scaffolding

### Setting up the design phase

I established the architecture records and version-control structure before deploying the platform components. The documentation captured the intended design, known trade-offs, and conditions that would require a later decision to be reversed. The repository structure gave each change a traceable place in the build history.

I also used a branch-based workflow so design, implementation, testing, and documentation changes could be reviewed as separate units. This reduced the chance that an architectural decision would exist only inside configuration files or personal memory. The scaffolding created a record of what was chosen, why it was chosen, and what the rehearsal could prove. It also gave later work a controlled path from an initial decision through deployment evidence and final release documentation.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/34c557ca-b856-453b-a1e7-2176c6590695_79svnjie)

### ADR-001: Kubernetes distribution tradeoff

ADR-001 selects kind v0.32.0 and places dev, int, and prod in three namespaces within one cluster. This design provides the required promotion shape without adding the cost or setup time of three separate clusters. It allows the same local machine to rehearse movement through the three zones.

The trade-off is weaker isolation. The namespaces do not provide cluster-scoped separation, and the rig cannot apply or test admission policy independently for each zone. A shared cluster can show whether namespaced resources promote correctly, but it cannot prove that separate zone-level control planes or policies behave as intended.

The decision should be reversed when admission policy must be tested per zone. At that point, separate clusters or another structure with the required control boundary would be necessary.

## Standing Up the Three-Namespace Cluster and Object Store

### Cluster foundation and SeaweedFS deployment

I created a multi-namespace Kubernetes cluster with kind and deployed SeaweedFS as the object store. The dev, int, and prod namespaces represent the three promotion zones used throughout the rehearsal. This gave the delivery pipeline separate targets while keeping the full build on one local machine.

SeaweedFS provided the file-arrival path needed to test event visibility. The design goal was to show that an uploaded object could enter the store and produce an observable notification that later processing could use. The deployment also tested whether the same object-storage components could run across all three namespaces.

This foundation established the cluster shape used by later database, GitLab, and promotion work. It proved local component behavior while leaving cloud service equivalence as a documented fidelity gap.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/34c557ca-b856-453b-a1e7-2176c6590695_nqprl0h0)

### Filer notification event observed end to end

A Create event appeared on the SeaweedFS filer metadata stream after the object arrived. The event path was /buckets/test-bucket/event-proof.txt. oldEntry was null, and newEntry.name was event-proof.txt. Those fields confirmed that the filer observed a new object rather than an update to an existing entry.

This result proved that object arrival was visible through the filer metadata path. It did not prove that the S3 notification API worked because that API was not available for the required behavior. The distinction matters for later integration decisions. The rig can demonstrate an event-driven path through filer notifications, but it cannot claim direct compatibility with an S3 notification design.

The observed limitation was recorded in the fidelity map so the successful event would not hide the missing API behavior.

## Proving the Database Migration and Pipeline Path

### PostgreSQL 18 and GitLab CE deployment goals

I deployed PostgreSQL 18 and GitLab CE into the cluster to test two separate delivery concerns. PostgreSQL provided a version-fixed database target for migration testing. GitLab CE provided the pipeline path needed to run containerized jobs and verify network access from the runner.

The database work checked whether schema changes could execute against the selected PostgreSQL version instead of relying on an unpinned local database. The pipeline work checked whether a job container could reach the kind cluster and apply the required deployment stages.

Together, these components connected database change control with the promotion workflow. The goal was not to reproduce a managed federal service. It was to prove that the defined migrations, runner network path, and staged deployment flow could execute inside the rehearsal environment.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/34c557ca-b856-453b-a1e7-2176c6590695_65ue4snu)

### Least-privilege credential design

I separated database responsibilities between migration_user and app_user. migration_user owns schema changes, while app_user handles normal runtime reads and writes. This prevents the application credential from carrying permissions that are only needed during controlled migrations.

Using one superuser for both jobs would increase the effect of a leaked application password. An attacker or faulty application process could drop tables, rewrite the schema, or take ownership of database objects. The runtime role does not need those capabilities to perform its intended work.

The split limits what a compromised app_user credential can change. It does not remove every database risk, but it keeps migration authority out of the normal traffic path and makes the purpose of each credential clear.

## Governance That Travels With the Rig: Fidelity Map and Documentation

### Writing the governance artifacts

I created a Fidelity Map that documents six critical gaps between the local rehearsal rig and the intended federal platform. The list includes the discovered SeaweedFS notification limitation. Each gap has an assigned owner and review date so it remains a tracked condition rather than a warning that can be forgotten after the demonstration.

The map separates working local behavior from claims about production equivalence. It records what the rig can test, what it cannot reproduce, and where a later platform decision still needs evidence. This protects reviewers from interpreting a successful local deployment as proof that cloud controls, managed services, or program processes are ready.

Linking the map from the README makes those limits travel with the rig. Anyone receiving the repository can find the boundaries beside the setup and release instructions.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/34c557ca-b856-453b-a1e7-2176c6590695_mnndl7zo)

### A gap the rig cannot resolve

Gap 5 covers real provisioning lead times. A local kind cluster can start within minutes on one machine because its resources are created directly from local configuration. That speed does not represent the time required to establish a federal cloud environment.

Cloud account setup, network tickets, approval paths, and wait queues exist within the program office. They are organizational dependencies, not Kubernetes objects that can be modeled in YAML. The rig can rehearse application deployment after those prerequisites are complete, but it cannot shorten or reproduce the process that delivers them.

This boundary affects planning. A fast clean-clone result proves local setup speed only. It cannot be used as an estimate for cloud provisioning or as evidence that external dependencies will be available on the same timeline.

## Shipping the Release: Tagged, Documented, and Ready to Hand Off

### Timed clean-clone proof and release

I performed a timed clean-clone run to measure how long the rig took to start from a fresh repository copy. This replaced an estimated setup duration with an observed result. The test also checked whether the documented instructions were complete enough to recreate the environment without relying on an existing local state.

A clean clone is stronger evidence than rerunning a working directory because it removes cached assumptions, untracked files, and manual changes that may have accumulated during development. Timing the process added an operational measure to the release record.

The result became part of the handoff documentation alongside the tagged release and fidelity map. It proved repeatable local bring-up under the tested conditions. It did not measure cloud provisioning, production deployment, or another machine with different resources.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/34c557ca-b856-453b-a1e7-2176c6590695_futifnoj)

### What completion actually means

A running cluster was only one condition for completion. The release also required a tagged v1.0.0 version, closed stories, and documentation with reviewed dates. These controls marked which configuration and instructions belonged to the finished handoff rather than leaving the result as an unversioned working directory.

The stakeholder readout needed to include the measured clean-clone time so setup claims came from an observed run. The README also needed to link directly to the fidelity map. That connection keeps the rig's limits visible beside its setup and usage instructions.

Completion therefore meant the build could be identified, recreated, reviewed, and handed to another person with its boundaries intact. It did not mean the rehearsal had become a production federal platform.

## Secret Mission: End-to-End Promotion Pipeline Through GitLab Stages

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/34c557ca-b856-453b-a1e7-2176c6590695_t33w43v0)

### Deployments confirmed across all three namespaces

SeaweedFS was running in all three promotion zones: dev, int, and prod. Each namespace contained filer, master, S3, and volume deployments showing 1/1 Ready. The cluster also contained the supporting system pods, including CoreDNS and local-path.

This result confirmed the intended promotion shape on the local cluster. The same component set existed in each namespace, which gave the GitLab stages distinct deployment targets and showed that the configuration could move across the three zones.

Readiness at 1/1 proved that the Kubernetes deployments reached their expected local state. It did not prove workload performance, zone isolation, federal security controls, or production capacity. Those claims remain outside the rehearsal scope and are recorded through the fidelity documentation.

## Reflections: Tools, Concepts, and Lessons Learned

### Key tools and concepts

I used Kubernetes with kind to run the local cluster, Kustomize to manage the dev, int, and prod configurations, GitLab CE to coordinate the pipeline stages, SeaweedFS for object storage, and PostgreSQL 18 for version-fixed migration testing.

The build showed how a three-zone promotion path can run without specialized GitOps controllers. Kustomize overlays carried environment-specific configuration, while GitLab CI/CD applied the stages in order. I also learned why architectural fidelity gaps must be documented beside successful results. A local rehearsal can prove component behavior without proving that the same environment exists in federal production.

Decision records, pinned versions, owners, and review dates gave the rig a governance structure that remained part of the handoff.

### Time and challenges

This build took approximately 90 minutes to complete. The hardest part was debugging connectivity between the GitLab runner and the Kubernetes API. The job container needed the correct network path into the kind cluster before any promotion stage could apply its deployment configuration.

I traced the failure to the runner's network access and checked the network_mode configuration. After confirming that the runner could reach the cluster API, the dev, int, and prod stages executed as expected.

The issue showed that a correct pipeline definition is not enough when the execution container cannot reach its target. Runner networking is part of the delivery architecture. Verifying that path turned the pipeline from a static configuration into a tested promotion flow.

### What this project meant

I completed this build today to gain direct experience creating a zero-cost, multi-zone Kubernetes rehearsal rig for a federal data platform delivery path. I implemented Kustomize overlays, automated the deployment stages through GitLab CI/CD, and used decision records and a fidelity map to document the design and its limits.

The result showed that the local promotion flow, object arrival path, database migrations, and runner connection could operate together under the tested conditions. It also made clear which production claims the rig could not support.

My next step is to add automated drift detection and reconciliation. That would extend the current pipeline-based deployment model toward a GitOps workflow where declared state is checked against the running cluster and differences are corrected through a controlled process.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/34c557ca-b856-453b-a1e7-2176c6590695)*
