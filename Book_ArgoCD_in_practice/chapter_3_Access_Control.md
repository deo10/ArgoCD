pages 167-204

https://github.com/PacktPublishing/ArgoCD-in-Practice

# Access Control
To add a new user, the ConfigMap argocd-cm file should look like this:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
data:
  accounts.alina: apiKey, login
```

we can verify that the new user is created by running the following code:
```bash
argocd account list
```
The output should be like this:
NAME   ENABLED CAPABILITIES
admin  true    login
alina  true    apiKey, login

still need to set its password.
For this, we can use this command, using the admin password as the current password:
```bash
argocd account update-password --account alina --current-password pOLpcl9ah90dViCD --new-password k8pL-xzE3WMexWm3cT8tmn
```

new ConfigMap resource we need to modify, which is where we will set all our Role-Based Access Control (RBAC) rules. Create a new file named argocd-rbac-cm.yaml in the same location where we have the argocd-cm.yaml ConfigMap and add this content:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
data:
  policy.default: role:readonly
```

# Service accounts

modify the argocd-cm ConfigMap to add the new service account user, which we will name gitops-ci:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
data:
  accounts.alina: apiKey, login
  accounts.gitops-ci: apiKey
  admin.enabled: "false" # Disable admin account
```

So, let’s see how we can assign the rights to update accounts to the user
alina. To do this, we will modify the argocd-rbac-cm.yaml file to create a
new role for user updates named role:user-update and we will assign the
role to the user (the lines that are used to define the policies start with p,
while the lines to link users or groups to roles start with g):
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
data:
  policy.default: role:readonly
  policy.csv: |
    p, role:user-update, accounts, update, *, allow
    p, role:user-update, accounts, get, *, allow
    g, alina, role:user-update
```

# Generate API Key for Service Account
To generate an API key for the service account gitops-ci, we can use the following command:
```bash
argocd account generate-api-key --account gitops-ci
```

# Project roles and tokens
To create a project role that allows read and sync privileges for the argocd application, we can create a new file named argocd-project.yaml and add the following content:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: argocd
spec:
  roles:
  - name: read-sync
    description: read and sync privileges
    policies:
      - p, proj:argocd:read-sync, applications, get, argocd/*, allow
      - p, proj:argocd:read-sync, applications, sync, argocd/*, allow
  clusterResourceWhitelist:
    - group: '*'
      kind: '*'
  description: Project to configure argocd self-manage application
  destinations:
    - namespace: argocd
      server: https://kubernetes.default.svc
  sourceRepos:
    - https://github.com/PacktPublishing/ArgoCD-in-Practice.git
```

# RBAC groups
To create RBAC groups and assign roles to them, we can modify the argocd-rbac-cm.yaml file. Here is an example of how to set up roles for developers, SREs, and readonly users:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
data:
  policy.csv: |
    g, role:developer, role:readonly
    p, role:developer, applications, sync, */*, allow
    g, role:sre, role:developer
    p, role:sre, applications, create, */*, allow
    p, role:sre, applications, update, */*, allow
    p, role:sre, applications, override, */*, allow
    p, role:sre, applications, action/*, */*, allow
    p, role:sre, projects, create, *, allow
    p, role:sre, projects, update, *, allow
    p, role:sre, repositories, create, *, allow
    p, role:sre, repositories, update, *, allow
    g, Sre, role:sre
    g, Developer, role:developer
    g, Onboarding, role:readonly
```

