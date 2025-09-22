pages 205-251
# Argo CD Bootstrap K8s Cluster
The main topics we will cover are as follows:
- Amazon EKS with Terraform
- Bootstrapping EKS with Argo CD
- Using the app of apps pattern
- Bootstrapping in practice

That’s the point in the book where the authors move from running Argo CD in a single, already-set-up cluster (covered in earlier chapters) toward bootstrapping an entire Kubernetes environment through GitOps.

Here’s the key focus of that chapter:
# Bootstrap philosophy
- Instead of manually creating namespaces, CRDs, or operators, you store everything in Git and let Argo CD “pull” and apply it.
- The cluster becomes self-describing: install Argo CD once → it reconciles itself and the rest of the platform components.

# Cluster bootstrap workflow
- Install Argo CD in the cluster (sometimes via kubectl or helm).
- Configure Argo CD to point to a “bootstrap repo” containing manifests for:
  - Argo CD configuration itself (projects, applications, RBAC, SSO)
  - Platform services (Ingress, monitoring stack, logging, secrets store, operators, etc.)
- Argo CD then applies these automatically — giving you a fully reproducible base platform.

# Patterns used
- App of Apps pattern: A “root” Argo CD Application deploys other Applications.
- This lets you separate infrastructure (e.g., monitoring, ingress) from workloads (apps, services).

# Benefits
- New clusters can be spun up consistently by just pointing Argo CD at the bootstrap repo.
- Makes DR and multi-cluster setups far easier (a key GitOps benefit).
- Reduces drift because Git remains the single source of truth.

# Practical examples
- The book shows example YAMLs for a “root application” and nested applications.
- Explains how to organize Git repos for bootstrap vs. workloads.
- Covers secrets handling and pitfalls like ordering dependencies.

# Questions
Chapter 5 Self-Check Cheat Sheet — Argo CD Bootstrap K8s Cluster
Question 1

- - Q: What is the main advantage of bootstrapping a Kubernetes cluster using Argo CD compared to manual setup?
- - A: Bootstrapping with Argo CD ensures the entire cluster configuration is declarative, version-controlled, and reproducible. Manual setup is error-prone and hard to replicate, while GitOps allows rapid rebuilding, consistency across clusters, and easier disaster recovery. The platform is prepared for future workloads automatically.

Question 2

- - Q: In the GitOps bootstrap model, where is the source of truth for the cluster’s configuration stored?
- - A: The source of truth is the Git repository that contains all declarative manifests for infrastructure, Argo CD configuration, and applications. Git provides versioning, auditing, and a single place for Argo CD to reconcile the desired state with the actual cluster state.

Question 3

- Q: What is the “App of Apps” pattern and why is it commonly used?
- A: The App of Apps pattern uses a root Argo CD application to manage multiple child applications. Instead of applying many separate Argo CD Applications manually, the root app automatically deploys and manages all other apps. It simplifies cluster bootstrapping, enables logical grouping (infra vs workloads), and provides a single point of control.

Question 4

- Q: How does Argo CD ensure idempotency when bootstrapping a cluster?
- A: Argo CD ensures idempotency by continuously reconciling the actual cluster state with the desired state in Git. If drift is detected, Argo CD automatically or manually syncs resources to match Git. Declarative Kubernetes manifests are inherently idempotent, so repeated application always results in the same state, making the bootstrap safe and repeatable.

Question 5

- Q: What risks or pitfalls might arise if secrets or dependencies aren’t handled properly during bootstrap?
- A: Missing or misconfigured secrets can cause deployment failures. Secrets should be stored securely in Vault or encrypted in Git (e.g., SealedSecrets, SOPS). Dependencies between resources or apps must be carefully defined; incorrect deployment order (like applying CRDs after dependent resources) can break the bootstrap. Use App of Apps, hooks, and sync waves to manage order and reduce risks.

Question 6

- Q: When bootstrapping a new cluster, what is the first step before Argo CD can reconcile everything else?
- A: The first step is to install a minimal Argo CD instance in the cluster, typically via kubectl apply or Helm. Once Argo CD is running, it can sync the bootstrap repository and automatically deploy all other applications and infrastructure.

Question 7

- Q: In a bootstrap repo, how might you separate infrastructure apps from workload apps?
- A: Separate them using different Git repositories or folders and the App of Apps pattern. For example:

bootstrap-infra/ → ingress, monitoring, operators

apps/ → workloads (dev, staging, prod)
This separation improves clarity, allows different RBAC policies, and reduces the risk of accidental changes to critical infrastructure.

Question 8

- Q: How does Argo CD handle the ordering of applications in the App of Apps model?
- A: Ordering is managed using the argocd.argoproj.io/sync-wave annotation. Applications with lower wave values are synced first. This ensures dependencies (e.g., CRDs, namespaces) are deployed before dependent apps or workloads, guaranteeing the correct order during bootstrapping.

Question 9

- Q: For multi-cluster bootstrap, what key design decisions would you make in your Git repo structure?
- A:

Use per-cluster directories or separate repos to isolate configurations.

Maintain common base manifests with cluster-specific overlays (Kustomize or Helm values).

Deploy each cluster via a root Argo CD application pointing to its folder.

Implement branching strategies and RBAC controls to manage deployments per cluster.

Question 10

- Q: After a cluster is bootstrapped, how would you perform disaster recovery using GitOps?
- A: Spin up a new cluster from scratch, install a minimal Argo CD instance, and point it to the bootstrap Git repository. Argo CD will automatically reconcile all resources, redeploying infrastructure, applications, and configurations. This tests the full bootstrap process, exposes potential issues, and ensures Git remains the single source of truth. All changes should be carefully version-controlled and tested.