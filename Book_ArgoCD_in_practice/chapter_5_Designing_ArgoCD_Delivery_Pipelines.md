pages 251-302
# Designing Argo CD Delivery Pipelines

## Main Topics

- **Motivation**
- **Deployment strategies**
- **Keeping secrets safe**
- **Real CI/CD pipeline**
- **Microservices CI/CD in practice**

---

## Deployment Strategies

Argo CD supports several deployment strategies:

- **Blue-Green Deployments:** Run two identical environments (blue and green) and switch traffic between them.
- **Canary Releases:** Gradually roll out changes to a subset of users before a full release.
- **Rolling Updates:** Incrementally update pods with new versions while maintaining availability.
- **A/B Testing:** Deploy different versions to different user segments for testing purposes.

---

### Blue/Green Deployment Strategy

Example Argo CD blue-green `Rollout` CRD:

```yaml
kind: Rollout
metadata:
  name: rollout-bluegreen
spec:
  replicas: 2
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: bluegreen
  template:
    metadata:
      labels:
        app: bluegreen
    spec:
      containers:
        - name: bluegreen-demo
          image: spirosoik/cho06:v1.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
  strategy:
    blueGreen:
      activeService: bluegreen-v1
      previewService: bluegreen-v2
      autoPromotionEnabled: false
```

**Key fields:**

- `activeService`: The current (blue) version served until promotion.
- `previewService`: The new (green) version to be served after promotion.
- `autoPromotionEnabled`: If `false`, promotion is manual.

**Manual promotion:**

```bash
kubectl argo rollouts promote rollout-bluegreen
```

---

### Canary Deployment Strategy

Example Argo CD canary `Rollout` CRD:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollout-canary
spec:
  replicas: 5
  revisionHistoryLimit: 2
  selector:
    matchLabels:
      app: rollout-canary
  template:
    metadata:
      labels:
        app: rollout-canary
    spec:
      containers:
        - name: rollouts-demo
          image: spirosoik/ch06:blue
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {}
        - setWeight: 40
        - pause: {duration: 40s}
        - setWeight: 60
        - pause: {duration: 20s}
        - setWeight: 80
        - pause: {duration: 20s}
```

**Key fields:**

- `steps`: List of steps for the canary deployment.
- `setWeight`: Percentage of traffic to the new version at each step.
- `pause`: Pause deployment, either indefinitely or for a set duration.

---

## Keeping Secrets Safe

Managing sensitive information securely is critical. Tools and techniques include:

- **Sealed Secrets:** Encrypt Kubernetes secrets for safe storage in version control.
- **HashiCorp Vault:** Manage secrets and sensitive data, integrated with Argo CD.
- **SOPS:** Encrypt files, including Kubernetes manifests, with various encryption methods.

**Resource model components:**

- **SecretStore:** Authenticates to external APIs to retrieve secrets, scoped to a namespace.
- **ExternalSecret:** Defines what data to retrieve from an external API, referencing a `SecretStore`.
- **ClusterSecretStore:** Global secret store, accessible from any namespace.

**Example `SecretStore`:**

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: SecretStore
metadata:
  name: secretstore-sre
spec:
  controller: dev
  provider:
    aws:
      service: SecretsManager
      role: arn:aws:iam::123456789012:role/sre-team
      region: us-east-1
      auth:
        secretRef:
          accessKeyIDSecretRef:
            name: awssm-secret
            key: access-key
          secretAccessKeySecretRef:
            name: awssm-secret
            key: secret-access-key
```

**Example `ExternalSecret`:**

```yaml
apiVersion: external-secrets.io/v1alpha1
kind: ExternalSecret
metadata:
  name: db-password
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: secretstore-sre
    kind: SecretStore
  target:
    name: secret-sre
    creationPolicy: Owner
  data:
    - secretKey: dbPassword
      remoteRef:
        key: devops-rds-credentials
        property: db.password
```

---

## Summary: Designing Argo CD Delivery Pipelines

### Purpose / Motivation

- Builds on infrastructure bootstrapped in Chapter 5.
- Demonstrates designing real CI/CD pipelines using Argo CD and Argo Rollouts.
- Minimizes deployment failures with appropriate strategies, automation, and secure secret handling.

### Key Concepts & Tools

- **Argo Rollouts:** Advanced deployment strategies (blue-green, canary).
- **Sync Phases & Resource Hooks:** Insert steps before/after sync for tests and verification.
- **Deployment Strategies:** When and how to use blue-green and canary.
- **Secret Management:** Secure storage and encryption with External Secrets Operator.

### Pipeline Design & Practices

- Sample CI/CD pipeline (e.g., GitHub Actions):
  - Builds/tests code and container images.
  - Uses Argo CD sync phases/hooks for integration tests.
  - Promotes new versions via blue-green or canary after validation.
- **Separation of Concerns:** Use Argo Projects, RBAC, and tokens for scoped access.
- **Monorepo & Microservices:** Repository structuring, versioning, and dependency management.

---

### ✅ Best Practices and Trade-offs

- **Choose deployment strategy based on risk tolerance:**  
  - Blue-green: clean switch, double resources.
  - Canary: gradual rollout.
- **Use hooks & sync phases wisely:**  
  - Clean up test resources after failures.
  - Preserve logs for debugging.
- **Secret management:**  
  - Avoid plain text secrets in Git.
  - Use operators/external stores and encryption.
- **Manage permissions carefully:**  
  - CI/CD pipelines need minimum necessary access.
  - Use Projects + tokens.

---

### 🔍 Real Life Example

- Walkthrough of setting up Argo Rollouts in a bootstrapped cluster.
- Automate blue-green promotion via GitHub Actions, integrating CI, Argo CD sync phases, and rollouts.
