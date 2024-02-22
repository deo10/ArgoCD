
ArgoCD Vocabulare

In ArgoCD, "Application" and "Project" are both organizational constructs that help manage and
deploy Kubernetes resources more effectively, but they serve different purposes:

Application:
An Application in ArgoCD represents a collection of Kubernetes resources (such as deployments,
services, configmaps, etc.) that are managed and deployed as a unit.
It typically corresponds to a single microservice, application component, or a set of related
components that together form a logical unit of deployment.
ArgoCD allows you to define various settings for an application, including the source repository,
the path within the repository where the Kubernetes manifests are located, the destination cluster,
synchronization settings, health checks, and more.

Project:
A Project in ArgoCD is a way to organize and group applications for easier management and access control.
Projects provide a level of isolation and access control, allowing you to restrict who can view, deploy,
or modify applications within that project.
They help in organizing applications based on teams, environments, or any other logical grouping that
suits your organization's needs.
Projects can have policies associated with them to enforce restrictions on which applications can be
deployed in that project and by whom.

In summary, while an Application represents a deployable unit of Kubernetes resources, a Project provides
organizational boundaries and access control mechanisms for managing multiple applications within ArgoCD.

Application source type: The tool we use to build applications such as
Helm, Kustomize, and jsonnet.

Target state: The desired state of an application, as represented in a Git
repository, which is the source of truth.

Live state: The live state of that application, which means what kind of
Kubernetes resources are deployed.

Sync status: The status which shows that the live state matches the target
state. Simply put, it’s whether the application deployed in Kubernetes
matches the desired states as described in the Git repository.

Sync: The phase of moving an application to the target state, which
happens by applying the changes in the Kubernetes cluster.
Sync operation status: The status for the sync phase, whether it’s failed or succeeded.

Refresh: Compare the latest code in Git with the live state and check what
is different.

Health status: The health of the application that it is running and whether
it can serve requests.

