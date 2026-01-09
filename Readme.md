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

I created a domain name in cloudflare and added the CNAME record to the ingress but there was an error with the site not loading and sending a DNS_PROBE_POSSIBLE error.

i checked with kubectl get pods and the pods were in pending state.
after checking with kubectl describe pod fast-api-5854f6fd66-qh8nl i saw that the pods were in pending state due to memory constraints. 
the error was 
"Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Warning  FailedScheduling  28m                  default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node.kubernetes.io/memory-pressure: }, 2 Insufficient memory. preemption: 0/3 nodes are available: 1 Preemption is not helpful for scheduling, 2 No preemption victims found for incoming pod.
  Warning  FailedScheduling  2m52s (x5 over 22m)  default-scheduler  0/3 nodes are available: 1 node(s) had untolerated taint {node.kubernetes.io/memory-pressure: }, 2 Insufficient memory. preemption: 0/3 nodes are available: 1 Preemption is not helpful for scheduling, 2 No preemption victims found for incoming pod."

  Then a new error appear in the pod the ErrImagePull and the problem was the docker image. after some research i found the problem was the docker image was build for arm64 and the cluster was using amd64. this happen because i build the docker image on my macbook which is arm64 and the Civo cluster is using amd64.

  to fix this i use the docker buildx create --use --name multiarch-builder and then i use the docker buildx build --platform linux/amd64 -t isidroalfonsin/kubernetes-deployment-test:0.0.1 -f Dockerfile . --push
  then i delete the old pod with the command kubectl delete pod fast-api-866b7d49d4-l9w4w and then i apply the deployment.yaml file with the command kubectl apply -f deployment.yaml

  now the site is loading but the response is not correct the problem now seems to be the A record in cloudflare. but using the curl command i can see the correct response.