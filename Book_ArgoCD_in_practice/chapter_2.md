# ArgoCD main components

Application Controller: The Application Controller is the core component of ArgoCD. It continuously monitors Git repositories for changes in application manifests (such as Helm charts, Kubernetes YAML files, Kustomize overlays, etc.) and reconciles the desired state defined in the Git repository with the actual state of the Kubernetes cluster.

API Server: The API Server provides a REST API that allows users and automation tools to interact with ArgoCD. It handles requests for managing applications, syncing them with the cluster, and performing various administrative tasks.

Repo Server: The Repo Server is responsible for managing the Git repositories that store the application manifests. It periodically syncs with the configured Git repositories to fetch the latest changes and notifies the Application Controller about any updates.

Dex (Optional): Dex is an OpenID Connect (OIDC) identity provider that can be integrated with ArgoCD to enable authentication and authorization using external identity providers like GitHub, Google, LDAP, etc. It allows users to log in to ArgoCD using their existing credentials from these identity providers.

Redis (Optional): Redis is an optional component used for caching and storing temporary data in ArgoCD. It can help improve the performance of ArgoCD by reducing the load on the main database and speeding up certain operations, such as syncing Git repositories and processing application updates.

Metrics Server (Optional): The Metrics Server collects and exposes various metrics about the performance and resource usage of ArgoCD components. It can be integrated with monitoring tools like Prometheus and Grafana to monitor the health and performance of ArgoCD deployments.