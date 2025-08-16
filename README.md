# Part 9: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline

    Part1: Manual Deployment using comand line tools (AzCLI, Docker Desktop and kubectl)  
    GitHub: https://github.com/santosh-gh/k8s-01
    YouTube: https://youtu.be/zoJ7MMPVqFY
    
    Part2: Automated Deployment using Azure DevOps Pipeline
    GitHub: https://github.com/santosh-gh/k8s-02
    YouTube: https://youtu.be/nnomaZVHg9I
    
    Part3: Automated Infra Deployment using Bicep and Azure DevOps Pipeline
    GitHub: https://github.com/santosh-gh/k8s-03
    YouTube: https://www.youtube.com/watch?v=5PAdDPHn8F8
    
    Part4: Deploying microservice applications in AKS using Helm Chat
    GitHub: https://github.com/santosh-gh/k8s-04
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0
    
    Part5: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline
    GitHub: https://github.com/santosh-gh/k8s-04
    YouTube: https://www.youtube.com/watch?v=MnWe2KGRrxg&t=883s
    
    Part6: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline
           Dynamically update the image tag in values.yaml
    GitHub: https://github.com/santosh-gh/k8s-06
    YouTube: https://www.youtube.com/watch?v=Nx0defm8T6g&t=11s
    
    Part7: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline
           Store the helm chart in ACR
           Dynamically update the image tag in values.yaml
           Dynamically update the Chart version in Chart.yaml
    
    GitHub: https://github.com/santosh-gh/k8s-07
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0
    
    Part8: Deploying microservice applications in AKS using Helm Chat and Azure Pipeline
           Store the helm chart in ACR
           Dynamically update the image tag in values.yaml
           Dynamically update the Chart version in Chart.yaml
           Deploy into multiple environments (dev, test, prod) with approval gates
    
    GitHub: https://github.com/santosh-gh/k8s-08
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0
    
    Part9: Deploying microservice applications in AKS using KUSTOMIZATION command line          
           Deploy into multiple environments (dev, test, prod) command line
    
    GitHub: https://github.com/santosh-gh/k8s-09
    YouTube: https://www.youtube.com/watch?v=VAiR3sNavh0

# Architesture

![Store Architesture](aks-store-architecture.png)

    # Store front: Web application for customers to view products and place orders.
    # Product service: Shows product information.
    # Order service: Places orders.
    # RabbitMQ: Message queue for an order queue.


# Directory Structure

![Directory Structure](image.png)

# Tetechnology Stack

    Azure Pipelines
    Infra (AzCLI/Bicep)
    AKS
    ACR
    kustomization

# Steps

    1. Infra deployment using AzCLI/Bicep command line or 
       Pipelines azcli-infra-pipeline.yml/bicep-infra-pipeline.yml

    2. Build and push images to ACR: CI Pipelines
       order-pipeline.yml, product-pipeline.yml, store-front-pipeline.yml

    3. kustomize install
       Kustomize comes pre bundled with kubectl version >= 1.14

       Linux: sudo install -o root -g root -m 0755 kustomize /usr/local/bin/kustomize
       Mac: brew install kustomize
       Windows: choco install kustomize

       kustomize version 

    4. Manifest Directory structure for kustomize     

    5. App deployment

        # Review
        kustomize build ./manifests/order/overlays/dev

        kubectl kustomize ./manifests/config/base/
        kubectl kustomize ./manifests/rabbitmq/overlays/dev
        kubectl kustomize ./manifests/order/overlays/dev
        kubectl kustomize ./manifests/product/overlays/dev
        kubectl kustomize ./manifests/store-front/overlays/dev

        kubectl kustomize ./manifests/config/base/
        kubectl kustomize ./manifests/rabbitmq/overlays/test
        kubectl kustomize ./manifests/order/overlays/test
        kubectl kustomize ./manifests/product/overlays/test
        kubectl kustomize ./manifests/store-front/overlays/test

        kubectl kustomize ./manifests/config/base/
        kubectl kustomize ./manifests/rabbitmq/overlays/prod
        kubectl kustomize ./manifests/order/overlays/prod
        kubectl kustomize ./manifests/product/overlays/prod
        kubectl kustomize ./manifests/store-front/overlays/prod

        # Apply Patches

         kustomize build ./manifests/order/overlays/dev | kubectl apply -f . -n dev

        kubectl apply -k ./manifests/config/base/ -n dev
        kubectl apply -k ./manifests/rabbitmq/overlays/dev -n dev
        kubectl apply -k ./manifests/order/overlays/dev -n dev
        kubectl apply -k ./manifests/product/overlays/dev -n dev
        kubectl apply -k ./manifests/store-front/overlays/dev -n dev

        kubectl apply -k ./manifests/config/base/ -n test
        kubectl apply -k ./manifests/rabbitmq/overlays/test -n test
        kubectl apply -k ./manifests/order/overlays/test -n test
        kubectl apply -k ./manifests/product/overlays/test -n test
        kubectl apply -k ./manifests/store-front/overlays/test -n test

        kubectl apply -k ./manifests/config/base/ -n prod
        kubectl apply -k ./manifests/rabbitmq/overlays/prod -n prod
        kubectl apply -k ./manifests/order/overlays/prod -n prod
        kubectl apply -k ./manifests/product/overlays/prod -n prod
        kubectl apply -k ./manifests/store-front/overlays/prod -n prod

    5. Validate and Access the application

    6. Clean the Azure resources
    
# Infra deployment

    # Login to Azure

        az login
        az account set --subscription=<subscriptionId>
        az account show

    # Show existing resources

        az resource list

    # Create RG, ACR and AKS

        # AzCLI
        ./infra/azcli/script.sh

        OR

        # Bicep
        az deployment sub create --location uksouth --template-file ./infra/bicep/main.bicep --parameters ./infra/bicep/main.bicepparam

    # Connect to cluster

        RESOURCE_GROUP="rg-onlinestore-dev-uksouth-001"
        AKS_NAME="aks-onlinestore-dev-uksouth-001"
        az aks get-credentials --resource-group $RESOURCE_GROUP --name $AKS_NAME --overwrite-existing

        # Short name for kubectl
        alias k=kubectl

        # Create name spaces
        k create ns dev
        k create ns test
        k create ns prod    

        # Show all existing objects
        k get all   

Docker Build and Push
# Log in to ACR

    ACR_NAME="acronlinestoredevuksouth001"
    az acr login --name $ACR_NAME

    # Build and push the Docker images to ACR

    # Order Service
    docker build -t order ./app/order-service 
    docker tag order:latest $ACR_NAME.azurecr.io/order:v1
    docker push $ACR_NAME.azurecr.io/order:v1

    # Product Service
    docker build -t product ./app/product-service 
    docker tag product:latest $ACR_NAME.azurecr.io/product:v1
    docker push $ACR_NAME.azurecr.io/product:v1

    # Store Front Service
    docker build -t store-front ./app/store-front 
    docker tag store-front:latest $ACR_NAME.azurecr.io/store-front:v1
    docker push $ACR_NAME.azurecr.io/store-front:v1

    docker images

# Review and apply

    

# Clean the k8s namespace

    k delete all --all -n default

# Verify the Deployment

    k get pods
    k get services
    curl <LoadBalancer public IP>:80
    Browse the app using http://<LoadBalancer public IP>:80

# Clean the Azure resources

    az group delete --name rg-onlinestore-dev-uksouth-001 --yes --no-wait
