# Project - Building a Cloud-Native App for Best Buy

## Application Architecture
The following diagram shown represents the architecture of the app and how the services are connected to each other. 
![ApplicationArchitecture](ApplicationArchitecture.png)


## Docker images link:

| Service Name       | Docker Image Link                                           |
|--------------------|------------------------------------------------------------------|
| **Order Service**   | [sing1249/order-service](https://hub.docker.com/repository/docker/sing1249/store-front/tags) |
| **Product Service** | [sing1249/product-service](https://hub.docker.com/repository/docker/sing1249/product-service/tags) |
| **Makeline Service**| [sing1249/makeline-service](https://hub.docker.com/repository/docker/sing1249/makeline-service/tags) |
| **Store Admin**     | [sing1249/store-admin](https://hub.docker.com/repository/docker/sing1249/store-admin/tags) |
| **AI Service**      | [sing1249/ai-service](https://hub.docker.com/repository/docker/sing1249/ai-service/tags) |
| **Store Front**     | [sing1249/store-front](https://hub.docker.com/repository/docker/sing1249/store-front/tags) |


| Service Name       | Github Repository Link                                           |
|--------------------|------------------------------------------------------------------|
| **Order Service**   | https://github.com/sing1249/order-service-bb.git|
| **Product Service** | https://github.com/sing1249/product-service-bb.git|
| **Makeline Service**| https://github.com/sing1249/makeline-bb.git |
| **Store Admin**     | https://github.com/sing1249/store-admin-bb.git |
| **AI Service**      | https://github.com/sing1249/ai-service-bb.git |
| **Store Front**     | https://github.com/sing1249/store-front-bb.git|


## Issues or limitations in the implementation
I was not able to connect the services using Azure service bus, for this I have tried changing the docker files to include env variables for Azure Service bus, modified the env variables in order-service and makeline-service and ensured they get the base64 encoded secret for Azure Service Bus from secrets.yaml but the solution did not work.
So I have finished the project using RabbitMQ instead. 

### Quota for Dall-e-3
When I was initially doing the deployment, my pod kept crashing for AI-Service and it was showing "CrashLoopBackOff" error in AKS. After research, I figured the image might not be built properly or maybe the Open AI resource was not properly deployed. After rebuilding the image, when I went into deploying the dall-e-3 version again, it showed me that I had insufficient quota. 

In order to overcome this, I tried creating the Open AI resource under my student subscription, and also tried creating AKS cluster under student subscription but the AKS cluster did not work properly as we can not choose 2 workernodes in it. 
So I tried deploying the AKS cluster and open AI in 2 different subscriptions. When I was generating the image and description for new product it did not work.
To overcome this I used kubectl logs "pod name" which showed me that there was an authorization problem. 

The Open AI resource was then deployed in the CDO subscription but since I had insufficient quota I could not deploy dall-e-3 and could only do dall-e-2 but when I chose dall-e-2 gpt-4 could not be deployed as they were available in different region so I chose dall-e-2 and gpt-40 and then that worked for me. 
