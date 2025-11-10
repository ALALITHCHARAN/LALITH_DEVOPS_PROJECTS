
Here’s a markdown-ready version of the article from **DevOpsCube** — “How to Setup EFK Stack on Kubernetes: Step by Step Guides” — formatted for GitHub, and I’ve included placeholders for images which you should host (e.g., in your repo’s `images/` folder) and link accordingly.

---

````markdown
# How to Setup EFK Stack on Kubernetes: Step by Step Guides

*By Shishir Khandelwal – Dec 9, 2021* :contentReference[oaicite:1]{index=1}  
*(Approx. 11 min read)*

---

## On This Page  
In this Kubernetes tutorial, you’ll learn how to setup the EFK stack on a Kubernetes cluster for log streaming, log analysis, and log monitoring. :contentReference[oaicite:2]{index=2}

---

## What is the EFK Stack?  
EFK stands for **Elasticsearch**, **Fluentd**, and **Kibana**. It is a popular open-source choice for Kubernetes log aggregation and analysis. :contentReference[oaicite:3]{index=3}  
- **Elasticsearch**: A distributed, scalable search engine used to store logs and retrieve them. :contentReference[oaicite:4]{index=4}  
- **Fluentd**: A log-shipper/collector agent; supports multiple data sources & outputs. :contentReference[oaicite:5]{index=5}  
- **Kibana**: A UI tool for querying, visualizing and building dashboards against log data in Elasticsearch. :contentReference[oaicite:6]{index=6}  

---

## Setup EFK Stack on Kubernetes  
We will follow a step-by-step process using Kubernetes manifests. All manifests used in this guide are available in the GitHub repo:  
```bash
git clone https://github.com/scriptcamp/kubernetes-efk
````

> Note: All components are deployed in the **default** namespace unless you change it. ([devopscube.com][1])

### Architecture

![Image](https://devopscube.com/content/images/2025/03/image-7-56.png)

![Image](https://devopscube.com/content/images/2025/03/02-k8s-architecture-1.gif)

![Image](https://appscode.com/blog/post/logging-in-kubernetes-with-elasticsearch-fluentd-kibana/hero_hufff4a29dcea4dfa1a8ad1e024da79a8e_73878_1300x650_fill_q75_box_smart1.jpg)

![Image](https://miro.medium.com/1%2AEMHMHqgAgRHfDdgeg2FWUw.png)

![Image](https://media.licdn.com/dms/image/v2/D5612AQExLEmKkSl1Ww/article-inline_image-shrink_1000_1488/article-inline_image-shrink_1000_1488/0/1708598476573?e=2147483647\&t=ZnIm3JJ-FkVczxnCzHNiBcj0IcxPoUXO3DB8LX7WmRU\&v=beta)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2A1MEZxe002QFIbRBg-Ucyvw.jpeg)

At a high level:

* Fluentd is deployed as a DaemonSet — logs from all nodes are collected. ([devopscube.com][1])
* Elasticsearch is deployed as a StatefulSet — persists log data. ([devopscube.com][1])
* Kibana is deployed as a Deployment — connects to Elasticsearch for visualization. ([devopscube.com][1])

---

### Deploy Elasticsearch StatefulSet

#### Step 1: Create a headless service (`es-svc.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch
  labels:
    app: elasticsearch
spec:
  selector:
    app: elasticsearch
  clusterIP: None
  ports:
    - port: 9200
      name: rest
    - port: 9300
      name: inter-node
```

> Save as `es-svc.yaml` and apply:

```bash
kubectl create -f es-svc.yaml
```

#### Step 2: Create the StatefulSet (`es-sts.yaml`)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: es-cluster
spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
        - name: elasticsearch
          image: docker.elastic.co/elasticsearch/elasticsearch:7.5.0
          resources:
            limits:
              cpu: 1000m
              memory: 2Gi
            requests:
              cpu: 500m
              memory: 500Mi
          ports:
            - containerPort: 9200
              name: rest
            - containerPort: 9300
              name: inter-node
```

> Note: In the original article they mention a 3 Gi PVC for demonstration; production would require much more. ([devopscube.com][1])

Apply the manifest:

```bash
kubectl create -f es-sts.yaml
```

---

### Deploy Fluentd as DaemonSet

Configure and create the necessary ServiceAccount, ClusterRole & ClusterRoleBinding for Fluentd, then deploy it as a DaemonSet so that **every node** runs a Fluentd instance collecting logs and forwarding them to Elasticsearch. (Details and exact YAMLs are in the GitHub repo) ([devopscube.com][1])

---

### Deploy Kibana

Create a Deployment + Service so that Kibana connects to the Elasticsearch cluster.
Example snippet:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
        - name: kibana
          image: docker.elastic.co/kibana/kibana:7.5.0
          ports:
            - containerPort: 5601
```

Apply it similarly. Then access Kibana, create an index pattern (e.g., `logstash-*`), select `@timestamp`, and explore logs.

![Image](https://i.sstatic.net/oVLP7.png)

![Image](https://logit.io/uploads/general/4LTwtS9brpe2xpYnNwSSHn_kibana-index-patterns.png)

![Image](https://appscode.com/blog/post/logging-in-kubernetes-with-elasticsearch-fluentd-kibana/hero_hufff4a29dcea4dfa1a8ad1e024da79a8e_73878_1300x650_fill_q75_box_smart1.jpg)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AC8JyBP2GPAkTaovL)

![Image](https://images.contentstack.io/v3/assets/bltefdd0b53724fa2ce/blt7cd3cc05a1ab4047/5eec4a68dee8140c761c17c2/blog-k8s-o11y-logs-nginx.jpg)

![Image](https://www.bigbinary.com/blog_images/kubernetes/kibana_dashboard.png)

---

### Verification

1. Check Elasticsearch pods and health (`kubectl get pods`, `kubectl get statefulset`).
2. Check Fluentd DaemonSet: ensure there is a pod on each worker node.
3. Launch a sample test pod producing logs.
4. In Kibana, create the index pattern, navigate to *Discover*, and validate logs from your test pod are visible. ([devopscube.com][1])

---

## Best Practices

* Elasticsearch uses significant heap memory — allocate carefully (e.g., ~40-50% of total memory) so you don’t starve the OS. ([devopscube.com][1])
* Old indices can quickly fill up storage — automate cleanup (e.g., via CronJobs) to avoid disk exhaustion. ([devopscube.com][1])
* Replication is important — default replication factor is 1; consider at least 2 for HA. ([devopscube.com][1])
* Use dedicated nodes (e.g., via `nodeSelector`, `taints/tolerations`) for your logging stack so application workloads don’t interfere with performance.
* Consider cold-storage (S3, etc) for long-term logs and archive older indices.
* Monitor resource usage (CPU, memory, disk I/O) of Elasticsearch; Kubernetes environments often need tuning.

---

## Conclusion

By now you’ve seen how to deploy a fully functional EFK (Elasticsearch + Fluentd + Kibana) stack on a Kubernetes cluster with manifests.
– You’ve created the headless service + StatefulSet for Elasticsearch.
– You’ve deployed Fluentd as a DaemonSet to collect logs from containerized workloads.
– You’ve deployed Kibana for visualizing and analyzing log data.
– And you’ve validated logs from test pods appear in Kibana.

*If you intend to use this in production*, you’ll need to plan for higher-capacity storage, redundancy, dedicated resources, and log retention policies.

Hope this guide helps you understand the architecture and components for setting up a logging infrastructure on Kubernetes.

---

## Repository & Resources

* GitHub repo for manifests: [https://github.com/scriptcamp/kubernetes-efk](https://github.com/scriptcamp/kubernetes-efk) ([devopscube.com][1])
* Original blog: [https://devopscube.com/setup-efk-stack-on-kubernetes/](https://devopscube.com/setup-efk-stack-on-kubernetes/) ([devopscube.com][1])

---

## License & Attribution

This markdown is derived from the DevOpsCube blog by Shishir Khandelwal. Please ensure any usage respects the original article’s licensing terms.

```

---

### ✅ What you need to do:
- Download the images referenced (e.g., the architecture diagram, Kibana screenshots) and store them under your repo (e.g., `images/efk-architecture.png`, `images/kibana-logs.png`).
- Replace the `![](…)` links or use Markdown image syntax referencing the correct paths.
- Ensure code blocks maintain correct indentation and YAML formatting.
- Optionally split into multiple files if you prefer (e.g., `01-architecture.md`, `02-manifests.md`).
- Add a README link to this file in your GitHub repo.

If you like—I can generate **two versions**: one with full image inline markdown references (with actual image names) and one minimal (just text + code blocks) to choose from. Would you like that?
::contentReference[oaicite:21]{index=21}
```

[1]: https://devopscube.com/setup-efk-stack-on-kubernetes/?utm_source=chatgpt.com "How to Setup EFK Stack on Kubernetes: Step by Step Guides"
