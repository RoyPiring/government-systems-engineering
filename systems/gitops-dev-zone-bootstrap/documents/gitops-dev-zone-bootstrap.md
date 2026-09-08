<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# GitOps Dev Zone Bootstrap with Argo CD

**Project Link:** [View Project](https://nextwork.ai/projects/91712a8d-f5d2-4173-8a11-4448d6ecc6ea)

**Author:** Roy Piring: Cloud Platform Engineer | Build Master  
**Email:** rpiringhawaii@gmail.com

---

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/91712a8d-f5d2-4173-8a11-4448d6ecc6ea_os61cwge)

## The Mission: GitOps for a Federal Data Platform

### Why this shape, why now

I built a GitOps delivery pipeline on a local Kubernetes cluster with Argo CD and Kustomize. Git stored the desired state, while Argo CD watched that source and reconciled the cluster without direct workload changes through kubectl apply.

The architecture separated application source from deployment manifests and placed an AppProject fence around the development workload. Dev used automated synchronization and self-healing, while Integration required manual approval before synchronization.

This design created a controlled path from commit to cluster. It supported versioned changes, visible drift, automatic correction in development, and slower promotion in a higher zone. I tested those claims through deployment, an attempted repository-boundary violation, manual scaling, self-healing, and a separate Integration application with an empty syncPolicy.

## Standing Up the Cluster and Reconciliation Engine

### Goals for this step

I created the local Kubernetes cluster with Kind and installed Argo CD as the reconciliation engine. This established the runtime needed to compare the cluster’s live resources with the version-controlled manifests stored in Git.

The cluster hosted the Argo CD control plane and later received the development and Integration workloads. Argo CD supplied the watch and synchronization behavior, while Kubernetes ran the resulting Namespace, Service, Deployment, and pods.

I also secured administrative access after installation. The bootstrap password was used only to enter the system and rotate the credential. After changing it, I deleted the initial admin Secret. This left the active password hash in argocd-secret and removed the clear-text bootstrap value. The control plane was then ready to connect to the deployment repository and enforce the application boundary.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/91712a8d-f5d2-4173-8a11-4448d6ecc6ea_yt7h4hlv)

### Credential rotation and secret deletion

After I deleted argocd-initial-admin-secret, argocd admin initial-password failed with secrets "argocd-initial-admin-secret" not found. The command only reads that bootstrap Secret, so no initial password remained for it to print.

Login still worked because the active password hash remained in argocd-secret. Argo CD stored the first administrator password in clear text inside argocd-initial-admin-secret. Once I changed that password, the bootstrap Secret no longer served the live login path.

Deleting it reduced the risk that someone with permission to read Secrets in the argocd namespace could recover the original password. The Getting Started guidance also stated that Argo CD could recreate the initial Secret later if an administrator needed password regeneration. The failed command confirmed removal of the bootstrap credential, not loss of current access.

## Building the Deployment Repository

### Goals for this step

I created a dedicated GitOps repository for Kubernetes deployment manifests and environment overlays. This repository represented the desired platform state that Argo CD monitored and synchronized into the cluster.

The repository remained separate from the application source. It stored the Kustomize base, development overlay, Integration overlay, AppProject definition, and Application resources required by the delivery path.

This boundary kept deployment history distinct from code history. An application source change could not update the cluster until the desired-state repository recorded the corresponding deployment change. Argo CD then used that commit as the source for reconciliation. The structure also prepared the platform for a second zone and a promotion path, allowing environment differences to remain explicit without copying the full base configuration.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/91712a8d-f5d2-4173-8a11-4448d6ecc6ea_fhn4gyh9)

### Why deployment manifests live apart from application source

The deployment repository contained desired state only. Application source remained elsewhere, so a build could not push a code change that immediately retriggered itself through the same repository. Code history and deployment history also remained separate.

DR-002 selected this split instead of a monorepo. The decision accepted the cost of maintaining two repositories because the platform expected a second zone and a promotion path. Changes could move through the deployment repository under controls that differed from application development.

The reversal condition was narrow. If the platform remained single-zone and never required promotion, the additional repository boundary could stop earning its maintenance cost. Until then, the separation protected the delivery path. Application code produced an artifact, while the GitOps repository declared which version and configuration should run. Argo CD followed only the desired-state history.

## Connecting the Repository and Fencing the Application

### Goals for this step

I connected the private GitOps repository to Argo CD with a read-only Personal Access Token. Read access was enough because the reconciliation engine needed to pull desired state rather than write changes back to the repository.

I then created the platform-dev AppProject as a permission fence. The policy restricted the application to one approved source repository and one destination namespace, applying least privilege to both where manifests came from and where resources could be created.

The fence separated repository connectivity from application authorization. A valid token could make a repository reachable, but the AppProject still had to permit that source and destination. I tested the boundary with disallowed repositories instead of treating the written configuration as proof. The resulting evidence showed whether Argo CD blocked an attempted application that crossed the declared source rule.

### Proving the permission boundary by attempting a violation

I first attempted to create bad-app from https://github.com/someone/other-repo.git. The request failed on connectivity because authentication was required and the repository was not found. Since that source was unreachable, the request never reached the AppProject check.

I repeated the violation with the reachable public repository https://github.com/argoproj/argocd-example-apps.git. Argo CD refused it because the application repository was not permitted in project 'platform-dev'.

bad-app was never created. The second test provided the required boundary evidence because the repository could be reached but still failed the project fence. The first failure tested connectivity, while the second tested authorization. Keeping those outcomes separate prevented an unreachable repository from being reported as proof that the AppProject permission rule had blocked access.

## Deploying Through the GitOps Path

### Goals for this step

I deployed the workload through the GitOps path by committing the Application and desired manifests instead of applying the NGINX workload directly to the development namespace.

Argo CD watched overlays/dev, pulled the repository state, and synchronized the declared resources. platform-dev-app used automated sync and CreateNamespace=true, allowing the engine to create Namespace dev, Service nginx, and Deployment nginx.

This flow proved that Git carried the deployment request and Argo CD performed the cluster change. kubectl apply was still used for the AppProject and Application custom resources in the argocd namespace because those objects established the GitOps control path. It was not used to install the NGINX workload in dev. That distinction kept the delivery claim limited to resources created from committed repository state.

### Deployment by commit, never by kubectl apply

I never ran kubectl apply against the dev namespace for the NGINX workload. After the Application reached main, Argo CD watched overlays/dev, pulled the desired state, and created Namespace dev, Service nginx, and Deployment nginx.

Automated synchronization on platform-dev-app performed that work through the platform-dev AppProject fence. CreateNamespace=true allowed the destination namespace to be created through the controlled application path.

I used kubectl apply only for the AppProject and Application custom resources in the argocd namespace. Those resources configured Argo CD itself; they did not directly create the NGINX Deployment or Service. The evidence therefore supported deployment by commit for the workload. Git contained the desired state, Argo CD observed it, and the engine created the development resources without a direct workload apply.

## Running the Drift Drill: Self-Heal Proven

### Goals for this step

I tested automated synchronization and self-healing as separate behaviors by manually scaling the live NGINX Deployment away from the replica count stored in Git.

The first run kept automated sync enabled but left self-healing off. This allowed me to observe whether Argo CD detected drift without correcting it. The second run enabled self-healing and repeated the same type of manual change.

The comparison measured three outcomes: whether Argo CD marked the Application OutOfSync, whether the manual replica count remained, and how long the engine took to restore the Git value after self-healing was active. Using the same Deployment and desired count kept the two conditions comparable. The drill showed that automated sync handles committed desired-state changes, while self-healing controls correction of uncommitted live drift.

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/91712a8d-f5d2-4173-8a11-4448d6ecc6ea_5sknkbgq)

### Before and after enabling self-heal

Before self-healing was enabled, kubectl scale changed the Deployment to 3 replicas. Argo CD marked platform-dev-app OutOfSync, but after 30 seconds the Deployment still showed 3/3. The engine detected the difference without correcting it.

I then ran argocd app set platform-dev-app --self-heal. A later manual scale changed the Deployment to 5 replicas, again creating drift from the replicas: 1 value stored in Git.

With self-healing active, Argo CD returned the live cluster to the state declared in Git. The Deployment moved from 5 replicas back to 1/1 in about 6 seconds, and the Application returned to Synced and Healthy. The two runs proved that drift detection and drift correction were separate capabilities. Automated sync saw the change, while self-healing authorized the control loop to reverse it.

## Secret Mission: Promoting to Integration and the Two-Speed Model

![Image](https://nextwork.ai/refreshed_maroon_timid_jujube/uploads/91712a8d-f5d2-4173-8a11-4448d6ecc6ea_1opnl6dc)

### Dev self-heals, int requires approval

The development and Integration Applications responded differently when live state no longer matched Git. platform-dev-app used automated sync with self-healing, so a manual kubectl scale caused OutOfSync and was then reversed to replicas: 1.

The recorded development test scaled the Deployment to 5 and returned it to 1/1 in about 6 seconds. docs/drift-evidence.txt preserved that result. This gave development a fast correction loop for changes that were not committed.

platform-int-app used an empty syncPolicy, so synchronization remained manual. Argo CD could report OutOfSync, but it would not create pods or reverse a hand edit until someone ran argocd app sync platform-int-app. DR-004 defined this two-speed model: development corrected drift automatically, while higher zones waited for explicit approval before applying desired state.

## Reflections and Wrap-Up

### Key tools and concepts

I used Argo CD for declarative GitOps delivery, Kustomize for shared bases and environment overlays, Kind for the local Kubernetes cluster, and GitHub for version control and pull-request-based changes.

The central design was a two-speed deployment model. Development used automated sync and self-healing for fast feedback, while Integration used manual synchronization to preserve an approval point. The AppProject fence restricted source and destination access around the development application.

I also learned to separate drift detection from drift correction. Without self-healing, Argo CD reported OutOfSync but left the manually scaled Deployment at 3/3. After self-healing was enabled, a scale to 5 returned to 1/1 in about 6 seconds. These tests showed how Git remained the source of desired state while each environment applied a different reconciliation policy.

### Time and challenges

This build took approximately 70 minutes. The hardest part was understanding how Kustomize patches changed the base Deployment for each environment, especially where an overlay replaced the replica count.

I also had to configure the AppProject fence so the approved repository could deploy into the intended namespace while the Integration Application retained manual synchronization. Repository connectivity and AppProject authorization required separate tests.

The drift drill added another challenge because automated sync and self-healing were easy to treat as the same behavior. The first scale remained at 3/3 for 30 seconds even though Argo CD detected OutOfSync. Only after enabling self-healing did the later scale to 5 return to 1/1. Recording those states separately made the control behavior clear and kept the conclusions tied to the observed cluster results.

I completed this build to learn the GitOps workflow with Argo CD, Kustomize manifest management, and a two-speed delivery model. Development used automated self-healing, while Integration retained manual synchronization as an approval boundary.

The build proved that workloads could move from Git into the cluster without a direct kubectl apply against dev. It also showed that detecting drift did not guarantee correction until self-healing was enabled.

My next step is to add automated end-to-end testing to the CI/CD pipeline before promotion into Integration. Those checks should validate rendered Kustomize output, AppProject boundaries, Argo CD health, expected replica counts, and drift behavior. This would add release evidence before the manual Integration sync while preserving Git as the source of truth.

---

*Built with [NextWork](https://nextwork.ai) - [View this project](https://nextwork.ai/projects/91712a8d-f5d2-4173-8a11-4448d6ecc6ea)*
