# Kubernetes Deployment  

## What is a Deployment in Kubernetes?  
A **Deployment** is a Kubernetes object used to manage applications.  
It ensures the **right number of Pods** are running, enables **easy scaling**, and supports **rolling updates and rollbacks**.  

Think of it as the **app manager** that takes care of:  
- Starting Pods based on your template.  
- Replacing failed Pods automatically.  
- Updating to new versions without downtime.  
- Scaling up/down based on your needs.  

---

## Uses of Deployment  
- Run multiple replicas of your app for **high availability**.  
- **Update** your app with zero downtime (rolling update).  
- **Rollback** to an earlier version if something breaks.  
- Keep your app running reliably without manual intervention.  

---

## What a Deployment file contains  
A typical Deployment YAML includes:  

1. **apiVersion** – API group/version (e.g., `apps/v1`).  
2. **kind** – Resource type (`Deployment`).  
3. **metadata** – Name, labels, etc.  
4. **spec** – Desired state of the Deployment, including:  
   - **replicas**: Number of Pod copies to run.  
   - **selector**: Labels to match Pods controlled by this Deployment.  
   - **template**: Pod definition (metadata + containers).  
     - **containers**: Name, image, ports, resources.  

---

## Example Deployment YAML  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  #Below template is for POD, Label is for POD      
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## Explanation of Example  
- Creates a **Deployment** named `nginx-deployment`.  
- Runs **3 Pods** of the Nginx web server.  
- Ensures Pods have the label `app: nginx`.  
- Each Pod runs a container from the `nginx:latest` image on port **80**.  

---

✅ This YAML is the **most common starting point** for deploying applications in Kubernetes.  
