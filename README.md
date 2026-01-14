# Lesson 4.1: Integrating Kustomize into CI/CD Pipelines

## Objective
Gain hands-on experience in automating Kubernetes configuration management using **Kustomize**, integrated into a **GitHub Actions CI/CD workflow**.

---

## 1. Setting Up GitHub Actions

### 1.1 GitHub Repository
Ensure you have a GitHub repository for your Kubernetes project that contains your **Kustomize configurations**.

#### Example Repository Structure
```text
.
├── base
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays
│   ├── dev
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   └── prod
│       ├── kustomization.yaml
│       └── patch.yaml
└── .github
    └── workflows
        └── main.yml
```        
**1.2 GitHub Actions Workflow**

Create a workflow file in your repository:

```
.github/workflows/main.yml
```
This file will define the CI/CD pipeline used to build and deploy your Kubernetes manifests using Kustomize.

### 2. Configuring the CI/CD Pipeline
**2.1 Workflow Definition**

Start by defining:

- The name of the workflow

- The trigger event (for example, a push to the main branch)

Example

```
name: Deploy with Kustomize

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      #  Checkout repo
      - name: Checkout repository
        uses: actions/checkout@v3

      #  Configure AWS credentials
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      #  Verify AWS identity (optional but useful for debugging)
      - name: Verify AWS identity
        run: aws sts get-caller-identity

      #  Set up kubectl
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: v1.32.0

      #  Update kubeconfig for EKS cluster
      - name: Update kubeconfig for EKS
        run: |
          aws eks update-kubeconfig \
            --region us-east-1 \
            --name my-eks-cluster

      # Test connectivity to cluster (optional)
      - name: Test cluster connectivity
        run: kubectl cluster-info

      #  Deploy using Kustomize
      - name: Deploy to Kubernetes using Kustomize
        run: |
          # Apply the overlay
          kubectl apply -k overlays/prod

          # Wait for deployment rollout to complete
          kubectl rollout status deployment/nginx-deployment -n default
 ```         

 ---

## 5. Applying Kustomize Configuration

Apply the Kustomize configuration to your Kubernetes cluster.

- Ensure your GitHub Actions runner is properly configured to communicate with your Kubernetes cluster.
- The runner must have access to a valid `kubeconfig` or cluster credentials stored securely as GitHub Secrets.

### Example
```yaml
- name: Deploy to Kubernetes
  run: |
    kubectl apply -k ./overlays/prod/
```
Note: I AM must have permission.The user must have 

```
{
  "Effect": "Allow",
  "Action": [
    "eks:DescribeCluster",
    "eks:ListClusters",
    "sts:GetCallerIdentity"
  ],
  "Resource": "*"
}
```
Also give Administractoraccess


### 6. Testing the CI/CD Pipeline
**6.1 Make Configuration Changes**

Modify a Kustomize configuration in your repository.
For example:

- Change the replica count in an overlay

- Update an image tag

```
name: Deploy with Kustomize

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      #  Checkout repo
      - name: Checkout repository
        uses: actions/checkout@v3

      #  Configure AWS credentials
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      #  Verify AWS identity (optional but useful for debugging)
      - name: Verify AWS identity
        run: aws sts get-caller-identity

      #  Set up kubectl
      - name: Set up kubectl
        uses: azure/setup-kubectl@v3
        with:
          version: v1.32.0

      #  Update kubeconfig for EKS cluster
      - name: Update kubeconfig for EKS
        run: |
          aws eks update-kubeconfig \
            --region us-east-1 \
            --name my-eks-cluster

      # Test connectivity to cluster (optional)
      - name: Test cluster connectivity
        run: kubectl cluster-info

      #  Deploy using Kustomize
      - name: Deploy to Kubernetes using Kustomize
        run: |
          # Apply the overlay
          kubectl apply -k overlays/prod

          # Wait for deployment rollout to complete
          kubectl rollout status deployment/nginx-deployment -n default
```

**6.2 Commit and Push Changes**

Commit your changes and push them to the main branch to trigger the pipeline.

```
git add .
git commit -m "Update replica count in production overlay"
git push origin main
```
**6.3 Observe Pipeline Execution**

1.Navigate to the Actions tab in your GitHub repository.

2.You should see the workflow running automatically.

3.Click on the workflow run to view:

- nJob execution steps

- Logs

- Success or failure status

Verify that the changes have been successfully applied to your Kubernetes cluster.

Note :

 **Check nodes**

```
 kubctl get node
```
**Check pods**
```
 kubectl get pods -n default
kubectl rollout status deployment/nginx-deployment -n default
```
```
# Create new nodegroup with bigger disk (20 GB) and same instance
aws eks create-nodegroup \
  --cluster-name my-eks-cluster \
  --nodegroup-name standard-ng \
  --node-role arn:aws:iam::707913648704:role/EKSNodeRole \
  --subnets subnet-09ae18a9b6e04c682 subnet-0d5bc198393997fe0 \
  --instance-types t3.micro \
  --ami-type AL2023_x86_64_STANDARD \
  --disk-size 20 \
  --scaling-config minSize=1,maxSize=1,desiredSize=1 \
  --region us-east-1
  ```

### 7 Verify Rollout Status (Best Practice ⭐)
**7.1 Check Rollout Progress**

```
kubectl rollout status deployment/nginx-deployment
```

kubectl rollout status deployment/nginx-deployment
Expected:
```
deployment "nginx-deployment" successfully rolled out
```

---

# Lesson 4.2: Handling Complex Configurations

## Objective
Explore strategies for managing **large-scale** and **complex Kubernetes configurations** using Kustomize.

---

## Tasks and Detailed Steps

### 1. Organize Configuration Structure

Structure your project to efficiently handle multiple applications or services.

- Use a **hierarchical directory structure** to organize configurations.
- Group configurations by:
  - Application
  - Environment (dev, staging, production)

#### Example Structure
```text
.
├── apps
│   ├── frontend
│   │   ├── base
│   │   └── overlays
│   │       ├── dev
│   │       └── prod
│   └── backend
│       ├── base
│       └── overlays
│           ├── dev
│           └── prod
```
2. Use of Kustomize Features

Leverage built-in Kustomize features to manage variations across environments or applications.

Overlays – environment-specific customization

Patches – targeted configuration changes

Generators – dynamic creation of ConfigMaps and Secrets

These features help avoid duplication and improve maintainability.

### Lesson 4.3: Best Practices and Tips

### 1. Performance Optimization
**1.1 Implementing Caching in CI/CD Pipelines**

Use caching mechanisms provided by your CI/CD platform to speed up builds and deployments.

- For GitHub Actions, use the actions/cache action.

- For AWS CodePipeline, consider using Amazon S3 for caching artifacts.

- Cache Docker layers, dependencies, or build outputs when possible.

```
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.kube
    key: ${{ runner.os }}-kube-${{ hashFiles('**/kustomization.yaml') }}
```
### 1.2 Optimizing Kustomize Configurations

- Break large Kustomize configurations into smaller, manageable units

- Use base and overlay structures effectively to maximize reuse

- Regularly review and refactor configurations for simplicity and clarity

### 2. Avoiding Common Pitfalls
**2.1 Using ConfigMaps and Secrets for Dynamic Values**

Avoid hardcoding environment-specific or sensitive values.

- Do not hardcode:

    - Image tags

   -   Environment variables

   - Credentials

- Use Kustomize ConfigMap and Secret generators

- Encrypt sensitive data using tools like AWS Key Management Service (KMS) before storing secrets

Example

```
configMapGenerator:
  - name: app-config
    literals:
      - APP_ENV=production
      - LOG_LEVEL=info
```

---

## 3. Using ConfigMaps and Secrets with Kustomize

Kustomize provides built-in generators to dynamically create **ConfigMaps** and **Secrets**, helping you avoid hardcoding environment-specific or sensitive values.

---

### 3.1 Using ConfigMap Generator

In your `kustomization.yaml` file, you can define a `configMapGenerator` to create a `ConfigMap` from literal values or files.

#### Example
```yaml
# kustomization.yaml
configMapGenerator:
  - name: my-app-config
    literals:
      - app_name=myKustomizeApp
      - log_level=debug
```

### 4. Applying Updates Safely with Kustomize and GitOps

To ensure stability and reliability when managing Kubernetes configurations at scale, updates should be applied in a controlled and observable manner.

**4.1 GitOps Workflow**

Adopt a GitOps approach where Git is the single source of truth for all Kubernetes manifests.

- All changes to base and overlays are made via Git commits.

- Pull Requests are used to review configuration changes.

- CI/CD pipelines automatically apply changes after approval and merge.

This ensures:
 
 - Auditability of changes

- Easy rollback using Git history

- Reduced risk of misconfigurations
5. Environment-Specific Configuration with Overlays

Kustomize overlays allow you to manage environment-specific differences cleanly without duplicating manifests.

**Common Use Cases for Overlays**

Different replica counts (dev vs prod)

Different image tags

Environment-specific ConfigMaps and Secrets

Resource limits and requests

### 6. Configuration Management Using ConfigMap and Secret Generators
**6.1 Why Use Generators in Overlays?**

Best practice is to define configMapGenerator and secretGenerator in overlays, not in the base.

Reason:

- The base remains environment-agnostic

- Overlays control environment-specific values

- Avoids hardcoding values in base manifests

Example (overlay kustomization.yaml)
```
configMapGenerator:
  - name: nginx-config
    literals:
      - ENV=production
      - LOG_LEVEL=info
  ```
  Kustomize automatically appends a hash to the ConfigMap name, ensuring pods restart when values change.

 ### 7. Referencing ConfigMaps and Secrets in the Base

Although generators live in overlays, the Deployment in the base must reference them.

Example: Base Deployment (deployment.yaml) 

```
 envFrom:
  - configMapRef:
      name: nginx-config
```

Kustomize resolves the hashed name automatically at build time.

This separation ensures:

- Clean base manifests

- Flexible environment control

- Safe configuration updates
  
**8. Verifying Changes in the Kubernetes Cluster**

After applying Kustomize configurations, always verify changes.

**8.1 Verify Resources**

```
kubectl get deployments
kubectl get pods
kubectl get configmaps
kubectl get secrets
```
**8.2 Verify Rollout Status**
```
kubectl rollout status deployment/nginx-deployment
```

**9. CI/CD Integration Best Practices (GitHub Actions)**

When using GitHub Actions with EKS:

- Cache dependencies to speed up pipelines

- Validate cluster access early (kubectl cluster-info)

- Always wait for rollout completion

- Fail fast on deployment errors

- This ensures faster feedback and safer deployments.
