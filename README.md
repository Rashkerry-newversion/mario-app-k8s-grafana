
# 🎮 Mario Web App on Kubernetes (with Helm Monitoring)

This guide walks you through deploying the **`sevenajay/mario`** Docker image to your local Kubernetes cluster and adding **monitoring with Prometheus and Grafana** . This project showcases a Kubernetes deployment of a Mario-themed web application using handcrafted deployment.yaml and service.yaml files. The Docker image is pulled directly from Docker Hub.
Grafana is installed alongside the app to monitor cluster performance and application metrics, and Lens is used to visually explore the live Kubernetes environment.
This project is ideal for beginners learning Kubernetes deployment, service exposure, observability with Grafana, and visual debugging with Lens.

---

## 🧱 Prerequisites

- Docker (for Minikube)
- kubectl
- Helm
- Minikube (or other local K8s)
- Lens (optional)

```
├──mario-site-helm/
├── mario-deployment.yml
└── service.yml
```

## Project Folder Structure

![Folder Structure](images/image25.png)

---

## 📦 Step 1 – Start Minikube

- Run the following command in the terminal.

```bash
minikube start
```

![Minikube Start](images/image1.png)

---

## 📄 Step 2 – Create `mario-deployment.yaml` And `service.yaml`

- Create a file in the project directory with following details:

- Filename: `mario-deployment.yaml`
- File contents: copy and paste below yaml script

```yaml
apiVersion: apps/v1 
kind: Deployment
metadata:
  name: mario-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      name: mario
  template:
    metadata:
      labels:
        name: mario
    spec:
      containers:
        - name: mario
          image: sevenajay/mario
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
```

- Create another file with details:
- File name: `service.yaml`
- File contents:copy and paste below yaml script

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mario-service
spec:
  type: NodePort
  selector:
    name: mario
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```  

![Deployment Scripts](images/image2.png)

---

## ☸️ Step 3 – Deploy Mario

- In your terminal run below commands to deploy

```bash
kubectl apply -f mario-deployment.yaml
kubectl apply -f service.yaml
```  

![Kubctl Apply](images/image3.png)

---

## 🌍 Step 4 – Open Mario Site

- To view site. Run:

```bash
minikube service mario-service
```

![Minikube Service view](images/image4.png)
![Live Site Via MiniKube](images/image5.png)

- Use Ctrl+C to end transmission

---

## 📊 Step 5 – Add Monitoring with Helm

### 5.1 Add repo & update

- Run the following commands to add Helm charts monitoring. Skip if helm is alredy installed.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

![Helm install](images/image6.png)

### 5.2 Install kube‑prometheus‑stack

- From there install Prometheus for monitoring by running below in your terminal:

```bash
helm install mario-monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace
```

![Prometheus Installation](images/image7.png)

### 5.3 Access Grafana

- Before port forwarding, make sure to have your grafana password. To get it:
Run

```bash
kubectl --namespace monitoring get secret mario-monitoring-grafana -o yaml
```

- From the display search for
data:
  admin-password: [password] #which is in base64

  ![Grafana Password](images/image9.png)

- Copy it and go to below link to decode
    <https://www.base64decode.org/>

  ![Decoding](images/image10.png)

- Access Grafana  and port forward by running:

```bash
kubectl -n monitoring port-forward svc/mario-monitoring-grafana 30000:80
```

![Grafana Port Forwarding](images/image11.png)

- Go to **<http://localhost:30000>**
- Enter your username 'admin' and decoded password

![Grafana GUI](images/image12.png)

### Note: Ignore if above works smoothly for you

- If its in pending mode or creating mode wait a bit for it to fully run
- Also if you have old stack confusing current monitoring stack. You can delete them by running:

```bash
helm list -A       #To view all stacks
helm uninstall <stack-name> -n <corresponding-namespace> #Repeat depending on the stacks you want to delete using the name and namespace
kubectl delete namespace <namespace-attached-name>      #To delete namespace, to be on the safer side
helm repo update
helm install mario-monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace   #Re-run to create mario monotoring stack.
kubectl -n monitoring port-forward svc/mario-monitoring-grafana 30000:80  #Run to forward monitoring
```

![Grafana Forwarding](images/image8.png)

Go to **<http://localhost:30000>**  
Login `admin / decoded-password`

![Grafana dashboard](images/image12.png)

- On the dashboard, go to connections to add new connections(prometheus,Loki)

![Dashboard Connections](images/image14.png)

- Add new data source

![Add New Data Source](images/image15.png)

- Go to terminal and run

```bash
kubectl get svc mario-monitoring-kube-prom-prometheus -n <namespace> -o wide #to get ip address
```

![CLUSTER-IP](images/image13.png)

- Give it a name for the URL: it should look like `http://<CLUSTER-IP>.114:9090`

![Add URL link](images/image16.png)

- Go to Explore to view visual of data source

![Explore View](images/image17.png)
![Explore View](images/image18.png)

**Note**
Make sure the terminal is still port forwarding. Always re-forward if connection is lost or stopped to keep grafana GUI running.

---

## 🔍 Step 6 – (Opt) Lens Visualization

Open Lens, connect to Minikube, inspect resources.

- Launch Lens Desktop App
- On the left sidebar, click “Clusters”
- You should already see your minikube cluster (or whatever name you used)
- Click Connect
- You’ll see your cluster’s dashboard (CPU, memory, nodes, etc.)
- Click on Overview for full minikube clusters

![Minikube Overview](images/image19.png)

- Explore Namespaces
    On the left menu, click “Namespaces”
    Select or filter to:
    monitoring (for Grafana, Prometheus, Alertmanager)
    default or your Mario app’s namespace

![Minikube Namespaces](images/image20.png)

-View Mario App
    Click “Workloads” > “Deployments”
    Look for mario-deployment
    Click “Services”
    Locate mario-service (NodePort 30080)
    You can click to view:
       Live pod logs
       Events
       Resource usage (CPU, memory)

![Minikube Namespaces](images/image21.png)

- Port Forward Easily from Lens
- Instead of CLI, do this:
    Select a Network > Services > mario-service
    Click the “Port Forward” button in top-right corner
    Lens will expose the service at something like:

![Lens Port forwarding](images/image22.png)  

![Port Forwading Success](images/image23.png)

---

## 🧨 Step 7 – Teardown

- To tear down resources run below commands

```bash
kubectl delete -f mario-deployment.yaml
kubectl delete -f service.yaml
helm uninstall mario-monitoring -n monitoring
kubectl delete namespace monitoring
```

![Code Cleanup](images/image24.png)

---

## Recap & Summary

- Deployed Docker Hub image on K8s
- Exposed via NodePort
- Added Prometheus + Grafana with Helm
- Clean removal steps

---

**Rashida Mohammed**  
[LinkedIn](https://www.linkedin.com/in/rashida-mohammed-cloud)
[GitHub](https://github.com/Rashkerry-newversion)

---

MIT Rashida Mohammed © 2025
