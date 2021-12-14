# dsti-devops-project
Project Ropository for S21 DevOps Assignment

## Project Description

The Project provides a Fibonacci Calculator that uses a multi-container deployment hosted on Microsoft Azure Kubernetes Service.
In the Fibonacci Calculator, the user will enter an Index, and the calculator will provide the correct value. 
Also the interface will display the users last entered values.

To Recall the fibonacci sequence, please see the following picture. A value is always the addition of the two previous values.
![Fibonacci Sequence](image/fib_sequence.JPG)

This is a mockup of the final interface: 
![Mockup](image/mockup.JPG)

## The (local) Core Application
![Core Architecture](image/local_architecture.JPG)
- each of the components are containerized
- Once the user visits the application in the browser, it will be going to an Nginx Webserver
- Nginx will route between a react application with all the front-end assets and an Express Server that functions as an API e.g. when a user submits a value in the browser
- In fact, two instances of Nginx should be used: one only responsible for the routing and one that is tight to the React App and will serve its files on Port 3000
- The "Values I have seen" will be stored in a postgres database as they are more permanent
- The Calculated Values will be stored in a Redis Database (Key/Value Pairs)
- The worker is a node.js process which looks for new values in Redis, calculates the value and puts it back to Redis
See the full backend flow here:
![Backend Architecture](image/backend_architecture.JPG)

## Kubernetes Architecture
![Kubernetes Architecture](image/kubernetes_architecture.JPG)
- Going to Porduction and Kubernetes, I will not use the Nginx Routing instance, but instead rely on an Ingress Service that routes to the different ClusterIPs
- I am using 5 different deployments with 1 to 3 Replicas and a ClusterIP added to the deployments (except the worker deployment as it doesnt need to be accessed)
- For Postgres I am additionally using a Postgres PVC
- For the conenction from the Server deployment to redis as well as to Postgres is done by storing environment variables about the ports, hosts and users
- The Password for Postgres is stored in a Kubernetes Secret that was created by the automated deployment

## Prod Deployment
- For the Prod Deployment I am using GitHub Actions as a CI Tool
- As the hosting platform, I am using the Microsoft Azure Kubernetes Serveice (AKS)
- The main deployment pipeline is described in the workflow.yaml which essentially provides all the different build and deployment steps needed
- The containers will be hosted in a private Azure Container Registry (ACR), access is granted via a Service Principal

### Kubernetes Setup
- For the initial Kubernetes setup, a Microsoft ARM Template was used as per documentation https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-rm-template
- Due to Quota Limit in the Azure Students subscription, only 1 node cluster could be deployed of size Standard_D4s_v3

### GitHub Action
- I created a Service Principal for GitHub in azure with the following command: az ad sp create-for-rbac --name "GitHubSP" --role contributor --scopes /subscriptions/0bef94e0-e086-44d4-9dc7-be9a1cf2c728/resourceGroups/DSTI-DevOps-Project --sdk-auth
- The Service Principal was added to Github Secrets for Access to Azure
- The Postgres Password is stored as a Github Secret and then via the Deployment created as a Secret in Kubernetes
- The SP has pull and push permissions for the ACR
- Also, I added the Username and Password of ACR Service Principal as Github Secrets