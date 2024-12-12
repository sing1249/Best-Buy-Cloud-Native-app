# Project - Building a Cloud-Native App for Best Buy

## Application Architecture
The following diagram represents the architecture of the app and how the services are connected:

![ApplicationArchitecture](ApplicationArchitecture.png)

## Application and Architecture Explanation
The architecture represents receiving orders and completing them for Best Buy. Below is an explanation of how the services interact:

1. **Store-Front**: Used by employees to add orders to the cart. Orders are sent to `order-service` via a managed backend service (Azure Service Bus).
2. **Product-Service**: Acts as a database for all products Best Buy sells. The `store-front` fetches products from this service.
3. **Store-Admin**: Used by the company to manage products and complete orders. It connects to `product-service` to add new products and `makeline-service` to store completed orders in the **Order Database** (MongoDB).
4. **AI-Service**: Helps generate descriptions and images for new products. It integrates with `store-admin`, using Azure OpenAI services (DALL-E-3 and GPT-4) to generate content.

## Deployment Instructions
To deploy the architecture, Azure Kubernetes Service (AKS) is used. Follow these steps:

### Setting Up the Cluster
1. Create a resource group in Azure Portal (e.g., **Project2**).
2. Configure an AKS cluster:
   - 1 system node
   - 2 worker nodes
   - Manual scaling enabled

### Configuring AI Services
1. Deploy Azure OpenAI resources in the same resource group.
2. Add two deployments:
   - **GPT-4** for generating descriptions.
   - **DALL-E-3** for generating images.
3. Obtain the endpoint and API key:
   - Add the endpoint to the environment variables of the `ai-service` in `bestbuy-all-in-one.yaml`.
   - Base64 encode the API key using:
     ```bash
     echo -n "key" | base64
     ```
   - Add the encoded value to `secrets.yaml`.

### Deploying Services
1. Connect to the AKS cluster via Azure Portal using the provided commands.
2. Switch to the directory containing the deployment files.
3. Apply the configuration files:
   - Deploy config maps and secrets:
     ```bash
     kubectl apply -f config-maps.yaml
     kubectl apply -f secrets.yaml
     ```
   - Deploy all services using:
     ```bash
     kubectl apply -f bestbuy-all-in-one.yaml
     ```
4. Expose `store-front` and `store-admin` services via Load Balancer. Access them using the IP addresses provided in the Azure Portal under Services > Ingress.

### Deploying Individual Services
Each service can be deployed individually using its specific YAML file:
```bash
kubectl apply -f <service-specific-file>.yaml
```

## Youtube video link
This the link for youtube video demo:
https://youtu.be/UQa1D2H8C0E 

## Table of Microservice Repositories:

| Service Name       | Github Repository Link                                           |
|--------------------|------------------------------------------------------------------|
| **Order Service**   | https://github.com/sing1249/order-service-bb.git|
| **Product Service** | https://github.com/sing1249/product-service-bb.git|
| **Makeline Service**| https://github.com/sing1249/makeline-bb.git |
| **Store Admin**     | https://github.com/sing1249/store-admin-bb.git |
| **AI Service**      | https://github.com/sing1249/ai-service-bb.git |
| **Store Front**     | https://github.com/sing1249/store-front-bb.git|
| **RabbitMQ**        | https://github.com/sing1249/rabbitmq.git |
| **MongoDB**         | https://github.com/sing1249/mongo.git |


## Table of Docker Images:

| Service Name       | Docker Image Link                                           |
|--------------------|------------------------------------------------------------------|
| **Order Service**   | [sing1249/order-service](https://hub.docker.com/repository/docker/sing1249/store-front/tags) |
| **Product Service** | [sing1249/product-service](https://hub.docker.com/repository/docker/sing1249/product-service/tags) |
| **Makeline Service**| [sing1249/makeline-service](https://hub.docker.com/repository/docker/sing1249/makeline-service/tags) |
| **Store Admin**     | [sing1249/store-admin](https://hub.docker.com/repository/docker/sing1249/store-admin/tags) |
| **AI Service**      | [sing1249/ai-service](https://hub.docker.com/repository/docker/sing1249/ai-service/tags) |
| **Store Front**     | [sing1249/store-front](https://hub.docker.com/repository/docker/sing1249/store-front/tags) |



## Bonus Task: Implement a CI/CD Pipeline for Each Microservice 
The pipeline will push a new docker image and deploy a new container using that new image. This will take place without the service being down. 
In order to implement a CI/CD pipeline, we will first add our secrets and variables in each repostiories. 
The secrest will be added in Github for each repository.
In order to add them we will follow these steps:
1. We will go to settings for each repository
2. And then go to Secrets and Variable and then Action
3. Under the secrets we will add these 3 secrets:
- DOCKER_USERNAME - This represents my docker username
- DOCKER_PASSWORD - This represents my docker password
- KUBE_CONFIG_DATA - This is obtained by using the following commands. In my case I used these commands. 
"kubectl config view --minify --flatten --output yaml > kube_config_minimal.yaml"
"cat kube_config_minimal.yaml | base64 -w 0 > kube_config_base64.txt"
4. Under variables, we will add 3 variables:
- CONTAINER_NAME - Any container name we want to give, for example store-admin
- DEPLOYMENT_NAME - Name of the deployment we want to give, for example store-admin
- DOCKER_IMAGE_NAME - This will be the new name of docker image that will be created after a new push is pushed to github.

### Screenshots showing the implementation:
![](Screenshots/newpush.png)
![](Screenshots/successful.png)
![](Screenshots/newimage.png)

## Issues or limitations in the implementation:
I was not able to connect the services using Azure service bus, for this I have tried changing the docker files to include env variables for Azure Service bus, modified the env variables in order-service and makeline-service and ensured they get the base64 encoded secret for Azure Service Bus from secrets.yaml but the solution did not work.
So I have finished the project using RabbitMQ instead. 

### Quota for Dall-e-3
When I was initially doing the deployment, my pod kept crashing for AI-Service and it was showing "CrashLoopBackOff" error in AKS. After research, I figured the image might not be built properly or maybe the Open AI resource was not properly deployed. After rebuilding the image, when I went into deploying the dall-e-3 version again, it showed me that I had insufficient quota. 

In order to overcome this, I tried creating the Open AI resource under my student subscription, and also tried creating AKS cluster under student subscription but the AKS cluster did not work properly as we can not choose 2 workernodes in it. 
So I tried deploying the AKS cluster and open AI in 2 different subscriptions. When I was generating the image and description for new product it did not work.
To overcome this I used kubectl logs "pod name" which showed me that there was an authorization problem. 

The Open AI resource was then deployed in the CDO subscription but since I had insufficient quota I could not deploy dall-e-3 and could only do dall-e-2 but when I chose dall-e-2 gpt-4 could not be deployed as they were available in different region so I chose dall-e-2 and gpt-40 and then that worked for me. 
![](Screenshots/Quota.png)



### Kube Config Data Length:
When I was obtaiting the kube config data and putting it in Github, the length was too long. After some research I found it was showing the data for all the clusters. So in order to get the data for current cluster I used the following commands:
"kubectl config view --minify --flatten --output yaml > kube_config_minimal.yaml"
"cat kube_config_minimal.yaml | base64 -w 0 > kube_config_base64.txt"

### Bonus: Deployment fail for some services
I deleted the AKS cluster after 1-2 services to avoid any charges, because I had to do multiple attempts in making AKS, I wanted to minimize the cost as much as I could that is why some services might show deployment failed.
