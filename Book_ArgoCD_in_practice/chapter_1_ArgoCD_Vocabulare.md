deploy Kubernetes resources more effectively, but they serve different purposes:
the path within the repository where the Kubernetes manifests are located, the destination cluster,
synchronization settings, health checks, and more.
deployed in that project and by whom.

# ArgoCD Vocabulary

In ArgoCD, **Application** and **Project** are both organizational constructs that help manage and deploy Kubernetes resources more effectively, but they serve different purposes:

## Application
An **Application** in ArgoCD represents a collection of Kubernetes resources (such as deployments, services, configmaps, etc.) that are managed and deployed as a unit. It typically corresponds to a single microservice, application component, or a set of related components that together form a logical unit of deployment.

ArgoCD allows you to define various settings for an application, including:
- Source repository
- Path within the repository where the Kubernetes manifests are located
- Destination cluster
- Synchronization settings
- Health checks
- And more

## Project
A **Project** in ArgoCD is a way to organize and group applications for easier management and access control. Projects provide a level of isolation and access control, allowing you to restrict who can view, deploy, or modify applications within that project.

Projects help in organizing applications based on teams, environments, or any other logical grouping that suits your organization's needs. Projects can have policies associated with them to enforce restrictions on which applications can be deployed in that project and by whom.

> **Summary:**
> - An **Application** represents a deployable unit of Kubernetes resources.
> - A **Project** provides organizational boundaries and access control mechanisms for managing multiple applications within ArgoCD.

---

## Key Terms

### Application source type
The tool used to build applications, such as **Helm**, **Kustomize**, and **jsonnet**.

### Target state
The desired state of an application, as represented in a Git repository, which is the source of truth.

### Live state
The live state of the application, meaning the actual Kubernetes resources currently deployed.

### Sync status
Indicates whether the live state matches the target state. In other words, whether the application deployed in Kubernetes matches the desired state described in the Git repository.

### Sync
The process of moving an application to the target state by applying changes in the Kubernetes cluster.

**Sync operation status:** The result of the sync phase (e.g., failed or succeeded).

### Refresh
Compares the latest code in Git with the live state and checks for differences.

### Health status
Indicates whether the application is running and able to serve requests.

