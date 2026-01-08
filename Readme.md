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
 