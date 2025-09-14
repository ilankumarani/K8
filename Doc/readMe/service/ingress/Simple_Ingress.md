# Kubernetes Ingress - Comprehensive Example

## Overview
This repository provides a **detailed Kubernetes Ingress configuration** with advanced features like **TLS, path-based routing, host-based routing, CORS, rate limiting, session affinity, and WebSocket support**.

## Prerequisites
Before applying the Ingress configuration, ensure that:

✅ An **NGINX Ingress Controller** is installed and running.
```sh
kubectl get pods -n kube-system | grep ingress
```
✅ TLS secrets are created for HTTPS.
✅ Backend services (`frontend-service`, `auth-service`, `api-v1-service`, etc.) are running.

## Directory Structure
```
k8s-ingress/
│── ingress.yaml  # Full Ingress configuration
│── README.md      # Documentation
```

## Kubernetes Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "10"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "60"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    
    # Load Balancing & Sticky Sessions
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/affinity: "cookie"
    nginx.ingress.kubernetes.io/session-cookie-name: "route"
    nginx.ingress.kubernetes.io/session-cookie-expires: "3600"
    
    # Rate Limiting
    nginx.ingress.kubernetes.io/limit-rps: "10"
    
    # Buffering
    nginx.ingress.kubernetes.io/proxy-buffering: "on"
    nginx.ingress.kubernetes.io/proxy-buffer-size: "16k"
    
    # CORS
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-origin: "*"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, OPTIONS"
    
    # WebSockets
    nginx.ingress.kubernetes.io/proxy-http-version: "1.1"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - myapp.example.com
      secretName: myapp-tls-secret
    - hosts:
        - api.example.com
      secretName: api-tls-secret
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
          - path: /login
            pathType: Prefix
            backend:
              service:
                name: auth-service
                port:
                  number: 80
    - host: api.example.com
      http:
        paths:
          - path: /v1
            pathType: Prefix
            backend:
              service:
                name: api-v1-service
                port:
                  number: 80
```

## Key Features Explained
| Feature | Description |
|---------|------------|
| **Path-Based Routing** | Routes traffic based on path (`/`, `/login`, `/v1`). |
| **Host-Based Routing** | Supports multiple domains (`myapp.example.com`, `api.example.com`). |
| **TLS/HTTPS** | Uses `cert-manager` for automatic SSL certificates. |
| **NGINX Annotations** | Optimizes proxy settings, rate limits, CORS, buffering, and WebSockets. |
| **Load Balancing & Sticky Sessions** | Enables session affinity with cookies. |
| **Rate Limiting** | Limits requests per second (`limit-rps: "10"`). |
| **WebSocket Support** | Configures headers to support WebSockets. |

## Applying the Ingress
To deploy the Ingress, use:
```sh
kubectl apply -f ingress.yaml
```

## Verifying Ingress
Check the Ingress status:
```sh
kubectl get ingress -n default
kubectl describe ingress my-app-ingress
```

Check if the TLS secret exists:
```sh
kubectl get secret myapp-tls-secret
kubectl get secret api-tls-secret
```

## Troubleshooting
If the Ingress is not working:
```sh
kubectl logs -n kube-system -l app.kubernetes.io/name=ingress-nginx
```
Ensure backend services are running:
```sh
kubectl get svc
```

## Conclusion
This Kubernetes Ingress setup provides **secure, scalable, and optimized routing** for multiple backend services using the **NGINX Ingress Controller**. 🚀

Let me know if you need any modifications! 😊

