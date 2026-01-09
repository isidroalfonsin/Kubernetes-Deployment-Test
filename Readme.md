# Deploying a Basic Python Application to Kubernetes

In this project we will deploy a basic app with FastApi and uvicorn to test the deployment process in kubernetes.

---

## Setup

### Virtual Environment

First create a virtual environment and install the required packages:

```bash
python3 -m venv ./venv
source ./venv/bin/activate
```

### Installing Dependencies

```bash
pip install fastapi
```

Need to update it to the latest version using:

```bash
pip install --upgrade fastapi
```

Now install uvicorn with:

```bash
pip install uvicorn
```

uvicorn is an ASGI server implementation for serving the fastapi application.

With `pip freeze` you can see the list of packages installed in the virtual environment.

Create a requirements.txt file with fastapi and uvicorn.

---

## Application Development

### Creating the FastAPI Application

From the FastApi documentation create a simple application and save it in a file called `main.py`.

Change some of the code so instead of hello world it returns hello Sargento.

Run the application with:

```bash
uvicorn main:app --reload
```

### Adding Environment Variables

Change the `main.py` file to add an environment variable called ENV and use it in the response and now build the docker image again.

---

## Docker Configuration

### Building the Docker Image

Create a Dockerfile with the content from FastApi documentation and use docker build command to build the docker image from the Dockerfile:

```bash
docker build .
```

Now you should have a docker image called k8s-fast-api locally.

### Running the Docker Container

Use docker run to run the docker image now in port 8000:

```bash
docker run -p 8000:80 k8s-fast-api
```

Use curl to test the application:

```bash
curl http://localhost:8000
```

### Uploading to Docker Hub

Upload the docker image to docker hub with:

```bash
docker push isidroalfonsin/kubernetes-deployment-test:0.0.1
```

---

## Kubernetes Cluster Setup

The kubernetes cluster is created in Civo with 3 nodes and a simple firewall configuration.

### Configuration Files

Create a `deployment.yaml` file with the content from the kubernetes documentation for a basic deployment and add the resources configuration.

Create a `service.yaml` file with the content from the kubernetes documentation for a basic service and add the selector configuration.

Create the `ingress.yaml` file with the content from the kubernetes documentation for a basic ingress and add the selector configuration.

### Kubeconfig Setup

Download the kubeconfig from Civo and save it in the kubernetes folder and create a gitignore file to ignore the kubeconfig file in my repository.

With the command set the kubeconfig file as the default kubeconfig file for kubectl:

```bash
export KUBECONFIG=/Users/isidro/Documents/Kubernetes-Deployment-Test/civo-kubeconfig
```

---

## Deployment

### Checking the Cluster

With the command you can see the nodes in the cluster:

```bash
kubectl get nodes
```

### Applying Configurations

With the command you can apply the deployment and service to the cluster:

```bash
kubectl apply -f .
```

### Monitoring

With the command you can see the pods in the cluster:

```bash
kubectl get pods
```

With the command you can see the services in the cluster:

```bash
kubectl get services
```

Also with the command you can watch the pods in the cluster until they are ready:

```bash
kubectl get pods -w
```

### Updating Environment Variables

In the `deployment.yaml` file add the environment variable ENV with the value CIVO and apply the changes to the cluster with the command so a new deployment is created with the environment variable:

```bash
kubectl apply -f deployment.yaml
```

Apply the `ingress.yaml` file with the command:

```bash
kubectl apply -f .
```

---

## Troubleshooting

### Issue 1: Pods in Pending State (Memory Constraints)

After checking with `kubectl describe pod fast-api-5854f6fd66-qh8nl`, the pods were in pending state due to memory constraints.

**Error:**

```
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  28m                  default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node.kubernetes.io/memory-pressure: }, 2 Insufficient memory. preemption: 0/3 nodes are available: 1 Preemption is not helpful for scheduling, 2 No preemption victims found for incoming pod.
```

**Solution:** Reduced memory requests and limits in `deployment.yaml` from 300Mi/500Mi to 64Mi/128Mi and reduced replicas from 3 to 1.

### Issue 2: ErrImagePull (Architecture Mismatch)

The docker image was built for ARM64 (MacBook) but the Civo cluster uses AMD64.

**Solution:**

```bash
docker buildx create --use --name multiarch-builder
docker buildx build --platform linux/amd64 -t isidroalfonsin/kubernetes-deployment-test:0.0.1 -f Dockerfile . --push
kubectl delete pod <pod-name>  # Force recreation with new image
```

### Issue 3: DNS Configuration

Initially attempted to use a custom domain with Cloudflare, but this requires purchasing a domain and configuring nameservers at the domain registrar.

**Final Solution:** Use Civo's auto-generated hostname instead. Added the Civo hostname to `ingress.yaml`:

```yaml
spec:
  rules:
    - host: 4cc5ad2f-7a0f-4a2e-a0d8-dca012fa3267.k8s.civo.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: fast-api
                port:
                  number: 80
```

---

## Accessing the Application

**URL:** http://4cc5ad2f-7a0f-4a2e-a0d8-dca012fa3267.k8s.civo.com

Or test with curl:

```bash
curl http://4cc5ad2f-7a0f-4a2e-a0d8-dca012fa3267.k8s.civo.com
```

Expected response:

```json
{"Hello Sargento":"From: CIVO"}
```

Attach screenshots of the application running in the cluster. The cluster has been deleted to save resources.

---

## Useful Commands

- `kubectl exec -it <pod-name> -- bash` - Enter the pod to check logs or environment variables
- `kubectl get pods -w` - Watch pods status in real-time
- `kubectl describe pod <pod-name>` - Get detailed pod information and events
- `kubectl logs <pod-name>` - View pod logs