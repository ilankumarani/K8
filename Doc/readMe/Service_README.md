# 📘 Kubernetes Service

## 🔹 What is a Service?
A **Service** is another type of Kubernetes object.  
It provides a stable way to access Pods, even if the underlying Pods change or restart.  

---

## 🔹 Key Points
- Pod IPs are **unreliable**, but **Service IPs are stable**.  
- Services are **durable (unlike Pods)**:  
  - Provide a **static IP address**.  
  - Provide a **static DNS name**.  
  - Follow the format: `[servicename].[namespace].svc.cluster.local`.  
- Services are the **gateway** to access Pods.  
- They target Pods using **selectors** (match labels).  

---

👉 **In short**: A Service gives a **stable network identity** to access Pods reliably inside (or outside) the cluster.  
