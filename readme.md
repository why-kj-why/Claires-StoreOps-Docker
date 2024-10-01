# Deploying Applications and configuring Network Security Groups on an AKS Cluster


## Introduction

**Azure Kubernetes Service (AKS)** is a managed container orchestration service provided by Microsoft Azure that simplifies the deployment, management, and scaling of Kubernetes clusters. AKS reduces the complexity of Kubernetes management by automating key processes like scaling, updates, and monitoring, allowing developers to focus on building and managing their applications while Azure handles the underlying infrastructure.

**Azure Container Registry (ACR)** is a managed, private Docker registry service on Azure that allows users to store and manage container images for all types of container deployments. ACR integrates seamlessly with Azure Kubernetes Service (AKS) and other Azure services, enabling automated image building, patching, and deployment workflows while ensuring image security and compliance.

**Network Security Groups (NSGs)** in Azure are security configurations used to control inbound and outbound traffic to and from Azure resources. NSGs consist of a set of security rules that can allow or deny network traffic based on various parameters such as source IP address, destination IP address, port, and protocol. By applying NSGs to Azure resources like Virtual Machines, Subnets, or Network Interfaces, you can enforce security at different layers. NSGs are crucial for controlling traffic to and from the AKS nodes and services. When deploying AKS services that utilize an Azure Load Balancer, NSGs are used to ensure only the desired traffic reaches the service while restricting unauthorized access.


## Gathering all Ingredients

**Step 1: Resource Group**

1. Sign in to Azure and navigate to the Resource Group tab on the Azure portal, before pressing **Create**.
2. In the Basics tab, select the desired **Subscription** plan, give the Resource Group a **Name** and select the **Region**.
3. Add the **Tags** (if any) and proceed to the Review tab. Azure runs a quick validation before allowing you to press **Create**.

**Step 2: AKS Cluster**

1. Select the new Resource Group and press **Create**, which redirects to the Marketplace.
2. Select Azure Kubernetes Service (AKS) and press **Create**. In the Basics tab, select the **Resource Group**, give a **Name** to the cluster and use the default settings set by Azure (Kubernetes version can be upgrade to 1.30.x).
3. In the Networking tab, select the **Azure CNI Node Subnet** option in the **Network configuration** setting.
4. In the Integrations tab, press **Create new** under the Container Registry section and give the ACR a **Name** (keep a note of the resultant URL).
5. Add any **Tags**, before moving to the **Review** tab. Azure runs a validation check before approving the AKS cluster and the Azure Container Registry.


## Identify the AKS Cluster and Network Security Group

**Step 1: Retrieve AKS Cluster Information**
* Navigate to the Azure Portal: Open the Azure Portal and sign in with your credentials.
* Go to AKS Cluster: In the left-hand menu, click on "Kubernetes services" and select your AKS cluster from the list.
* View Networking Details: In the AKS cluster overview page, click on "Networking" under the "Settings" section.
	* This will show you the virtual network (VNet) and subnet associated with your AKS cluster, which will typically have an NSG attached.

**Step 2: Locate the Network Security Group (NSG)**
* **Go to the Virtual Network**: Click on the Virtual Network linked to your AKS cluster (shown in the Networking section).
* **Navigate to Subnets**: In the virtual network’s page, select "Subnets" under the "Settings" section.
* **Identify the Subnet**: Locate the subnet your AKS nodes are deployed into. There will usually be an NSG attached to this subnet, which you can view in the "Network security group" column.
* **Go to NSG**: Click on the NSG name to open the Network Security Group settings.

## Setting Up NSG Rules for AKS Service with Load Balancer

**Step 1: Define Traffic Requirements**
Before creating the NSG rules, determine the necessary parameters:

* **Source**: Could be a specific IP range, or "Any" for broader access.
* **Destination**: The public IP address of the AKS Load Balancer, could be "Any" for broader access.
* **Port**: The port your AKS service is listening on (80 for HTTP, or a custom port).
* **Protocol**: TCP

**Step 2: Create Inbound NSG Rules in the Azure Portal**
* **Open the NSG Settings**: From the NSG page (identified in the previous section), click on "Inbound security rules" in the left-hand menu.
* **Add a New Rule**: Click the "Add" button to create a new rule.
* **Source**: Set to Any or specify a source IP range.
* **Source Port Ranges**: Set to Any (or specific port ranges if needed).
* **Destination**: Select "IP Addresses" and enter the Public IP of your Load Balancer (which can be found in the "Load Balancers" section in Azure).
* **Destination Port Ranges**: Specify the port your service listens on (e.g., 80 for HTTP, 443 for HTTPS).
* **Protocol**: Select TCP.
* **Action**: Choose Allow to permit traffic.
* **Priority**: Set a priority number (e.g., 1000 for HTTP, 1001 for HTTPS).
* **Name**: Provide a descriptive name like Allow-HTTP-Inbound or Allow-HTTPS-Inbound.
* **Save the Rule**: After filling in the details, click "Add" to save the inbound rule.

**Step 3: Create Outbound NSG Rules in the Azure Portal**
* **Open the NSG Settings**: From the NSG page (identified in the previous section), click on "Inbound security rules" in the left-hand menu.
* **Add a New Rule**: Click the "Add" button to create a new rule.
* **Source**: Set to Any or specify a source IP range.
* **Destination**: Select "Any" to allow access from the internet.
* **Destination Port Ranges**: This specifies on which ports traffic will be allowed or denied by this rule. (e.g., 80 for HTTP).
* **Protocol**: Select TCP.
* **Action**: Choose Allow to permit traffic.
* **Priority**: Set a priority number (e.g., 1000 for HTTP, 1001 for HTTPS).
* **Name**: Provide a descriptive name like Allow-HTTP-Inbound or Allow-HTTPS-Inbound.
* **Save the Rule**: After filling in the details, click "Add" to save the inbound rule.

If the NSG rules are configured correctly, you should be able to access the application using your external IP.


## What comes in the Box?

* Docker image of the Streamlit app connected to the Tellmore IP, ready to answer all Retail questions out-of-the-box.
* Deployment YAML file which defines the deployment and service resources for the Kubernetes cluster, and sets the host and target ports as 8080.


## Here comes the Docker!
**Step 1: Login to Azure using Azure CLI**

Before deploying the Docker image, one must login to the correct Azure account via the CLI. Running the following command would redirect the user to a small window using which the user can sign into the Azure account:

```
az login
```


**Step 2: Login to the Azure Container Registry**

Login to the Azure Container Registry using DockerHub to create a repository for deploying the Docker image.

```
docker login <ACR name>.azurecr.io
```


**Step 3: Load the Docker image**

Load the tar file to intialise the Docker image.

```
docker load -i <docker-image-name>.tar
```


**Step 4: Run the Docker image on the local host**

Let the Docker image run on the local host for the duration of the deployment procedure. The host-port and target-port can be set in the Dockerfile accordingly, though it is recommended to set both as 8080.

```
docker run -p <host-port>:<target-port> --env-file .env <docker-image-name>:<tag>
```


**Step 5: Tag a repository on the ACR**

Ideally, the user must open a new Terminal window without terminating the previous task. The following command will tag the Docker image to a repository on the Azure Container Registry:

```
docker tag <docker-image-name>:<tag> <ACR name>.azurecr.io/<docker-image-name>:<tag>
```


**Step 6: Push the Docker image on to the repository**

All of the commands shall be prompted on the new Terminal, while the Docker image runs uninterrupted on the first Terminal. Use the following command to push the Docker image on to the working repository:

```
docker push <ACR-name>.azurecr.io/<docker-image-name>:<tag>
```


**Step 7: Install the Kubernetes CLI**

Install the Kubernetes CLI within the Azure Kubernetes Service cluster.

```
az aks install-cli
```


**Step 8: Retrieve credentials for AKS cluster**

Retrieve the credentials for the desired AKS cluster. It must be noted that the cluster must be present in an Azure Resource Group with access to vCPUs.

```
az aks get-credentials --resource-group <Azure-resource-group-name> --name <AKS-cluster-name>
```


**Step 9: Create a Kubernetes resource in the AKS cluster**

The deployment.yaml file contains information for deployment and service resources. Use the following command to push this file on to the Kubernetes instance running on the AKS cluster:

```
kubectl create -f deployment.yaml
```


**Step 10: Retrieve all active services**

The below-mentioned command returns a table the specific service hosted on the AKS cluster. The user can find the service-name in second section of the deployment.yaml file. Retrieve the External IP to access the Streamlit app hosted on the cluster. The Load Balancer of the AKS cluster may take 20-30 seconds to assign an External IP to the service, so it is advisable to wait for that duration before running this command.

```
kubectl get service <service-name>
```