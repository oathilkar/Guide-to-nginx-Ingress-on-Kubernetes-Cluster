***** Step-by-Step Guide to create K8S nginx Ingress on Kubernetes Cluster *****

This a the guide to deploy Application on Kubernetes cluster along with creation of Ingress K8s Object.
It creates Ingress components as Ingress Resource & Ingress Controller to route external traffic to the deployed Application.

In broader perspective & understanding utilizing nginx as an Application for Deployment.

**Prerequisites:**
1. A running Kubernetes cluster
2. `kubectl` installed and configured to interact with your cluster
3. Basic understanding of Kubernetes objects like Pods, Services, and Deployments

Step 1: Set Up Your Kubernetes Cluster
Ensure your Kubernetes cluster is running and `kubectl` is configured correctly.

```sh
kubectl get nodes
```

This command should list all the nodes in your cluster.

Step 2: Deploy an Application
For demonstration, we will deploy a simple Nginx application.

1. **Create a Deployment:**

```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
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

Apply the deployment:

```sh
kubectl apply -f nginx-deployment.yaml
```

2. **Create a Service:**

```yaml
# nginx-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort
```

Apply the service:

```sh
kubectl apply -f nginx-service.yaml
```

Step 3: Install an Ingress Controller
An Ingress controller is necessary to manage Ingress resources. We'll use the Nginx Ingress Controller.

1. **Install Nginx Ingress Controller using Helm:**

First, add the Helm repository:

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

Then install the Ingress Controller:

```sh
helm install nginx-ingress ingress-nginx/ingress-nginx
```

Verify the installation:

```sh
kubectl get pods -n default -l app.kubernetes.io/name=ingress-nginx
```

Step 4: Create an Ingress Resource
Define an Ingress resource to route traffic to your application.

```yaml
# nginx-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

Apply the Ingress resource:

```sh
kubectl apply -f nginx-ingress.yaml
```

Step 5: Update DNS
Ensure your domain (`your-domain.com`) points to the external IP of the Ingress Controller. You can find the external IP using:

```sh
kubectl get services -o wide -w -n default -l app.kubernetes.io/name=ingress-nginx
```

Step 6: Test Your Setup
Open a web browser and navigate to `http://your-domain.com`. You should see the Nginx welcome page.

### Additional Steps (Optional)
- **TLS/SSL Termination:** To secure your application, you can use Let's Encrypt or any other CA to manage SSL certificates and configure them in your Ingress resource.
- **Advanced Configuration:** Customize your Ingress resource with more advanced rules, annotations, and configurations as per your requirements.

This guide provides a basic setup for ingress on a Kubernetes cluster. Depending on your use case, you might need to dive deeper into specific configurations and optimizations.
