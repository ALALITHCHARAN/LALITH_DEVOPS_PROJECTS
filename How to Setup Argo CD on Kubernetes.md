
# How to Setup Argo CD on Kubernetes [Beginners Guide]

*By Aswin Vijayan – Apr 25, 2024*  
*(Approx. 9 min read)*

---

## On This Page  
In this blog we will look at how to install Argo CD on a Kubernetes cluster using a step-by-step approach. After the installation, we will deploy a sample application the GitOps way, using a Git repository to drive the deployment. :contentReference[oaicite:2]{index=2}

---

## Setup Prerequisites  
To get started you should have the following:  
1. A working Kubernetes cluster  
2. `kubectl` configured with cluster-admin permissions  
3. `helm` installed and configured on your workstation  
:contentReference[oaicite:3]{index=3}

---

## Argo CD Setup Using Manifest Files  
The easiest way to install Argo CD is using the plain Kubernetes manifest files (from the Argo CD project) for initial learning. :contentReference[oaicite:4]{index=4}  
We will deploy Argo CD in a namespace called `argocd`.

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
````

It will create various Kubernetes objects (services, deployments/statefulsets, RoleBindings, ConfigMaps, Secrets) along with Argo CD custom resources such as `Applications`, `AppProjects`, etc. ([DevOpsCube][1])

```bash
# Access UI via port-forward (for local access)
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

To log in:

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode ; echo
```

The default username is `admin`.
([DevOpsCube][1])

---

## Argo CD Setup Using the Official Helm Chart

For realistic/project use-cases, using the official Helm chart for Argo CD is recommended. ([DevOpsCube][1])

### Step 1: Add the Argo Helm repo

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm search repo argo
```

From the search you should see the `argo/argo-cd` chart in the list. ([DevOpsCube][1])

### Step 2: Customize Helm Chart Configuration Values

```bash
helm show values argo/argo-cd > values.yaml
```

Edit `values.yaml` as needed — for example, change the `service.type` from `ClusterIP` to `NodePort` to expose the UI externally.
([DevOpsCube][1])

### Step 3: Deploy Argo CD

```bash
kubectl create namespace argocd
helm install --values values.yaml argocd argo/argo-cd --namespace argocd
```

Then verify the deployment:

```bash
kubectl get all -n argocd
```

([DevOpsCube][1])

### Step 4: Log into the Argo CD Web UI

If you exposed via `NodePort`, you can access via `<node-IP>:<nodePort>`.
Otherwise, you may use port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Login with `admin` and the decoded secret from above.
([DevOpsCube][1])

---

## Deploying Applications from GitHub with Argo CD (GitOps Way)

Now that Argo CD is set up, let’s deploy a sample application (e.g., Nginx) as a GitOps workflow.

Place your manifests in a Git repository, then in the Argo CD UI click **+ NEW APP**.
Select the repository URL, path, destination cluster/namespace, sync policy (automatic or manual), etc.
([DevOpsCube][1])

Once created, Argo CD will sync the repository and deploy the application to the cluster. You’ll see the status in the UI and can monitor changes, rollouts, etc.

---

## Clean-Up (Optional)

If you want to uninstall the Argo CD setup:

```bash
helm uninstall argocd -n argocd
kubectl delete namespace argocd
```

Alternatively, if installed via manifest:

```bash
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

([DevOpsCube][1])

---

## Conclusion

By following this guide you should now have:

* Installed Argo CD using either manifests or Helm
* Configured access to the Web UI
* Deployed a sample application via GitOps using Argo CD
  For production usage, you’ll want to:
* Use proper ingress/SSL for the UI
* Use secure authentication (OIDC/SAML) for Argo CD
* Set resource limits, monitoring, and backups
* Manage multiple environments (dev/prod) via branching and value files

---

## Image References

Below are some visual guides from the original article. Replace the `![](…)` links with actual paths in your repo.

![Argo CD Setup Using Helm](images/argo-cd/setup-argo-cd-using-helm-1.png)
![Argo CD Add New App](images/argo-cd/setup-argo-cd-using-helm-2.png)
![Argo CD UI Example](images/argo-cd/setup-argo-cd-using-helm-3.png)

---

## Attribution

This markdown is derived from the DevOpsCube article “How to Setup Argo CD on Kubernetes [Beginners Guide]”. Please ensure your usage respects the original content’s licensing and attribution.

---

```

---

### ✅ What you should do next:
- Download the images shown (or relevant screenshots from the original article) and save them under your GitHub repo (e.g., `images/argo-cd/`).  
- Update each `![](…)` link to point to the correct relative path in your repo.  
- Double-check code blocks for correct indentation and formatting.  
- Optionally split into multiple markdown files (e.g., `01-install.md`, `02-gitops.md`) if you prefer modular docs.  

If you like, I can also **generate a minimal “readme-style” version** (less verbose, mainly commands + synopsis) or **split into sections** for a multi-file documentation setup. Would you like that?
::contentReference[oaicite:15]{index=15}
```

[1]: https://devopscube.com/setup-argo-cd-using-helm/?utm_source=chatgpt.com "How to Deploy Argo CD on Kubernetes [Beginners Guide]"
