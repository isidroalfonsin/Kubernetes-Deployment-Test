Deploying a basic python application to kubernetes

In this project we will deploy a basic app with FastApi and uvicorn to test the deployment process in kubernetes

First create a virtual environment and install the required packages

python3 -m venv ./venv

source ./venv/bin/activate

pip install fastapi 

Need to update it to the latest version using pip install --upgrade fastapi

Now install uvicorn with pip install uvicorn #uvicorn is an ASGI server implementation for 
serving the fastapi application

with pip freeze you can see the list of packages installed in the virtual environment

create a requirements.txt file with fastapi and uvicorn 

from the FastApi documentation create a simple application and save it in a file called main.py
change some of the code so instead of hello world it returns hello Sargento

run the application with uvicorn main:app --reload

create a Dockerfile with the content from FastApi documentation and use docker build . command to build the docker image from the Dockerfile now you should have a docker image called k8s-fast-api locally

Change the main.py file to add an environment variable called ENV and use it in the response and now build the docker image again 

use docker run -p 8000:80 k8s-fast-api to run the docker image now in port 8000

use curl http://localhost:8000 to test the application

upload the docker image to docker hub with docker push isidroalfonsin/kubernetes-deployment-test:0.0.1

The kubernetes cluster is created in Civo with 3 nodes and a simple firewall configuration

create a deployment.yaml file with the content from the kubernetes documentation for a basic deployment and add the resources configuration.

create a service.yaml file with the content from the kubernetes documentation for a basic service and add the selector configuration.

Download the kubeconfig from Civo and save it in the kubernetes folder and create a gitignore file to ignore the kubeconfig file in my repository

with the command export KUBECONFIG=/Users/isidro/Documents/Kubernetes-Deployment-Test/civo-kubeconfig set the kubeconfig file as the default kubeconfig file for kubectl.

with the command kubectl get nodes you can see the nodes in the cluster

with the command kubectl apply -f . you can apply the deployment and service to the cluster

with the command kubectl get pods you can see the pods in the cluster

with the command kubectl get services you can see the services in the cluster

also with the command kubectl get pods -w you can watch the pods in the cluster until they are ready

in the deployment.yaml file add the environment variable ENV with the value CIVO and apply the changes to the cluster with the command kubectl apply -f deployment.yaml so a new deployment is created with the environment variable

create the ingress.yaml file with the content from the kubernetes documentation for a basic ingress and add the selector configuration.

apply the ingress.yaml file with the command kubectl apply -f .

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

## Useful Commands

- `kubectl exec -it <pod-name> -- bash` - Enter the pod to check logs or environment variables
- `kubectl get pods -w` - Watch pods status in real-time
- `kubectl describe pod <pod-name>` - Get detailed pod information and events
- `kubectl logs <pod-name>` - View pod logs

