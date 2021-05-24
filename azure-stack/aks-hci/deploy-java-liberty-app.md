---
title: Deploy a Java application with Open Liberty or WebSphere Liberty on an Azure Kubernetes Service (AKS) on Azure Stack HCI cluster
description: Deploy a Java application with Open Liberty or WebSphere Liberty on an Azure Kubernetes Service (AKS) on Azure Stack HCI cluster.
author: mattmcspirit
ms.author: mmcspirt
ms.topic: conceptual
ms.date: 05/19/2021
keywords: java, jakartaee, javaee, microprofile, open-liberty, websphere-liberty, aks-hci, kubernetes
---

# Deploy a Java application with Open Liberty or WebSphere Liberty on an AKS on Azure Stack HCI cluster

This article demonstrates how to:  
* Run your Java, Java EE, Jakarta EE, or MicroProfile application on the Open Liberty or WebSphere Liberty runtime.
* Build the application Docker image using Open Liberty container images.
* Deploy the containerized application to an on-premises AKS on Azure Stack HCI cluster using the Open Liberty Operator.   

The Open Liberty Operator simplifies the deployment and management of applications running on Kubernetes clusters. With Open Liberty Operator, you can also perform more advanced operations, such as gathering traces and dumps. 

For more details on Open Liberty, see [the Open Liberty project page](https://openliberty.io/). For more details on IBM WebSphere Liberty, see [the WebSphere Liberty product page](https://www.ibm.com/cloud/websphere-liberty).

This article is divided into two main parts.

**Part 1** focuses on deploying the necessary artifacts in Azure. This includes a resource group and Azure Container Registry (ACR). To perform these tasks, you can either utilize [Azure Cloud Shell](/azure/cloud-shell/quickstart) (easiest) which includes the latest version of Azure CLI, **or** you can utilize a local system with the following tools installed:

  * Prepare a local machine with Unix-like operating system installed (for example, Ubuntu, macOS, Windows Subsystem for Linux).
  * Install Azure CLI following the guidance above.
  * Install a Java SE implementation (for example, [AdoptOpenJDK OpenJDK 8 LTS/OpenJ9](https://adoptopenjdk.net/?variant=openjdk8&jvmVariant=openj9)).
  * Install [Maven](https://maven.apache.org/download.cgi) 3.5.0 or higher.
  * Install [Docker](https://docs.docker.com/get-docker/) for your operating system.

**Part 2** focuses on deploying the application from your Azure Container Registry down into the AKS on Azure Stack HCI environment. For this, **you cannot use Azure Cloud Shell**. For the AKS on Azure Stack HCI environment, the following items are required:

* An [AKS on Azure Stack HCI cluster](./setup.md) with at least one Linux worker node that's up and running.
* You have configured your local `kubectl` environment to point to your AKS on Azure Stack HCI cluster. You can use the [Get-AksHciCredential](get-akshcicredential.md) PowerShell command to access your cluster using `kubectl`.
* Git command-line tools installed on your system. On a Windows system, you can use [**Git Bash**](https://git-scm.com/downloads) as the main console, and within the Git Bash console, call PowerShell and Azure when required.

## Create a resource group in Azure

An Azure resource group is a logical group in which Azure resources are deployed and managed.  

Create a resource group called *java-liberty-project* using the [az group create](/cli/azure/group#az_group_create) command in the *eastus* location. This resource group will be used later for creating the Azure Container Registry instance. 

```azurecli-interactive
RESOURCE_GROUP_NAME=java-liberty-project
az group create --name $RESOURCE_GROUP_NAME --location eastus
```

## Create an ACR instance in Azure

Use the [az acr create](/cli/azure/acr#az_acr_create) command to create the ACR instance. The following example creates an ACR instance named *youruniqueacrname*. Make sure *youruniqueacrname* is unique within Azure.

```azurecli-interactive
REGISTRY_NAME=youruniqueacrname
az acr create --resource-group $RESOURCE_GROUP_NAME --name $REGISTRY_NAME --sku Basic --admin-enabled
```

After a short time, you should see a JSON output that contains:

```output
  "provisioningState": "Succeeded",
  "publicNetworkAccess": "Enabled",
  "resourceGroup": "java-liberty-project",
```

### Connect to the ACR instance in Azure

You will need to sign in to the ACR instance before you can push an image to it. Run the following commands to verify the connection. Make a note of your login details as you'll use those later when connecting to AKS on Azure Stack HCI.

```azurecli-interactive
LOGIN_SERVER=$(az acr show -n $REGISTRY_NAME --query 'loginServer' -o tsv)
USER_NAME=$(az acr credential show -n $REGISTRY_NAME --query 'username' -o tsv)
PASSWORD=$(az acr credential show -n $REGISTRY_NAME --query 'passwords[0].value' -o tsv)

echo $LOGIN_SERVER
echo $USER_NAME
echo $PASSWORD

docker login $LOGIN_SERVER -u $USER_NAME -p $PASSWORD
```

You should see `Login Succeeded` at the end of command output if you have logged into the ACR instance successfully.

## Build application image in Azure

To deploy and run your Liberty application on the AKS on Azure Stack HCI cluster, you first need to containerize your application as a Docker image using [Open Liberty container images](https://github.com/OpenLiberty/ci.docker) or [WebSphere Liberty container images](https://github.com/WASdev/ci.docker).

1. Clone the sample code for this guide. The sample is on [GitHub](https://github.com/Azure-Samples/open-liberty-on-aks).
1. Change directory to `javaee-app-simple-cluster` of your local clone.
1. Run `mvn clean package` to package the application.
1. Run `mvn liberty:dev` to test the application. You should see `The defaultServer server is ready to run a smarter planet.` in the command output if successful. Use `CTRL-C` to stop the application.
1. Run one of the following commands to build the application image and push it to the ACR instance.
   * Build with the Open Liberty base image if you prefer to use Open Liberty as a lightweight open source Java™ runtime:

     ```azurecli-interactive
     # Build and tag application image. This will cause the ACR instance to pull the necessary Open Liberty base images.
     az acr build -t javaee-cafe-simple:1.0.0 -r $REGISTRY_NAME .
     ```

   * Build with the WebSphere Liberty base image if you prefer to use a commercial version of Open Liberty:

     ```azurecli-interactive
     # Build and tag application image. This will cause the ACR instance to pull the necessary WebSphere Liberty base images.
     az acr build -t javaee-cafe-simple:1.0.0 -r $REGISTRY_NAME --file=Dockerfile-wlp .
     ```

### Connect to your AKS on Azure Stack HCI cluster

To manage a Kubernetes cluster, you use [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), the Kubernetes command-line client. Once installed, on Windows, kubectl can be run from the CMD prompt, PowerShell console, and Git Bash.

As one of the pre-requisites, you should have configured your local `kubectl` environment to point to your AKS on Azure Stack HCI cluster. You can use the [Get-AksHciCredential](get-akshcicredential.md) PowerShell command to access your cluster using `kubectl`:

```powershell
Get-AksHciCredential -name AksHciClusterName
```

> [!NOTE]
> The above command uses the default location for the [Kubernetes configuration file](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), which is `%USERPROFILE%\.kube`. You can specify a different location for your Kubernetes configuration file using *-outputLocation*.

Back in your console, to verify the connection to your cluster, use the [kubectl get]( https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command to return a list of the cluster nodes.

```console
kubectl get nodes
```

The following example output shows the single node created in the previous steps. Make sure that the status of the node is *Ready*:

```output
NAME                                STATUS   ROLES   AGE     VERSION
aks-nodepool1-xxxxxxxx-yyyyyyyyyy   Ready    agent   76s     v1.18.10
```

## Install Open Liberty Operator

After creating and connecting to the cluster, install the [Open Liberty Operator](https://github.com/OpenLiberty/open-liberty-operator/tree/master/deploy/releases/0.7.0) by running the following commands.

```console
OPERATOR_NAMESPACE=default
WATCH_NAMESPACE='""'

# Install Custom Resource Definitions (CRDs) for OpenLibertyApplication
kubectl apply -f https://raw.githubusercontent.com/OpenLiberty/open-liberty-operator/master/deploy/releases/0.7.1/openliberty-app-crd.yaml

# Install cluster-level role-based access
curl -L https://raw.githubusercontent.com/OpenLiberty/open-liberty-operator/master/deploy/releases/0.7.1/openliberty-app-cluster-rbac.yaml \
      | sed -e "s/OPEN_LIBERTY_OPERATOR_NAMESPACE/${OPERATOR_NAMESPACE}/" \
      | kubectl apply -f -

# Install the operator
curl -L https://raw.githubusercontent.com/OpenLiberty/open-liberty-operator/master/deploy/releases/0.7.1/openliberty-app-operator.yaml \
      | sed -e "s/OPEN_LIBERTY_WATCH_NAMESPACE/${WATCH_NAMESPACE}/" \
      | kubectl apply -n ${OPERATOR_NAMESPACE} -f -
```

## Deploy application on the AKS on Azure Stack HCI cluster

Follow steps below to deploy the Liberty application on the AKS on Azure Stack HCI cluster. You will need to retrieve your login details from your [earlier session](#connect-to-the-acr-instance-in-azure).

1. If you used Azure Cloud Shell earlier, and are now using a separate console to connect to AKS on Azure Stack HCI, you'll need to lock in your credentials again.

   ```console
   LOGIN_SERVER=YourLoginServerFromEarlier
   USER_NAME=YourUsernameFromEarlier
   PASSWORD=YourPwdFromEarlier
   ```

2. Create a pull secret so that the AKS on Azure Stack HCI cluster is authenticated to pull image from the ACR instance.

   ```console
   kubectl create secret docker-registry acr-secret \
      --docker-server=${LOGIN_SERVER} \
      --docker-username=${USER_NAME} \
      --docker-password=${PASSWORD}
   ```

3. Again, if you used Azure Cloud Shell earlier, and are now using a separate tool or session to connect to AKS on Azure Stack HCI, you will need to clone the sample code for this guide. The sample is on [GitHub](https://github.com/Azure-Samples/open-liberty-on-aks).
4. Verify the current working directory is `javaee-app-simple-cluster` of your local clone.
5. Run the following commands to deploy your Liberty application with 3 replicas to the AKS on Azure Stack HCI cluster. Command output is also shown inline.

   ```console
   # Create OpenLibertyApplication "javaee-app-simple-cluster"
   cat openlibertyapplication.yaml | sed -e "s/\${Container_Registry_URL}/${LOGIN_SERVER}/g" | sed -e "s/\${REPLICAS}/3/g" | kubectl apply -f -

   openlibertyapplication.openliberty.io/javaee-app-simple-cluster created

   # Check if OpenLibertyApplication instance is created
   kubectl get openlibertyapplication javaee-app-simple-cluster

   NAME                        IMAGE                                                   EXPOSED   RECONCILED   AGE
   javaee-app-simple-cluster   youruniqueacrname.azurecr.io/javaee-cafe-simple:1.0.0             True         59s

   # Check if deployment created by Operator is ready
   kubectl get deployment javaee-app-simple-cluster --watch

   NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
   javaee-app-simple-cluster   0/3     3            0           20s
   ```

6. Wait until you see `3/3` under the `READY` column and `3` under the `AVAILABLE` column, use `CTRL-C` to stop the `kubectl` watch process.

### Test the application

When the application runs, a Kubernetes load balancer service exposes the application front end to the internet. This process can take a while to complete.

To monitor progress, use the [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) command with the `--watch` argument.

```console
kubectl get service javaee-app-simple-cluster --watch

NAME                        TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
javaee-app-simple-cluster   LoadBalancer   10.109.118.3   192.168.0.152   80:30580/TCP     68s
```

Once the *EXTERNAL-IP* address changes from *pending* to an actual public IP address, use `CTRL-C` to stop the `kubectl` watch process.

Open a web browser to the external IP address of your service (`192.168.0.152` for the above example) to see the application home page. You should see the pod name of your application replicas displayed at the top-left of the page. Wait for a few minutes and refresh the page to see a different pod name displayed due to load balancing provided by the AKS on Azure Stack HCI cluster.

![Picture of a Java liberty application successfully deployed on AKS on Azure Stack HCI.](.\media\deploy-java-app/deploy-succeeded.png)

>[!NOTE]
> Currently, the application is not using HTTPS. It is recommended to [enable TLS with your own certificates](/azure/aks/ingress-own-tls).

## Clean up the resources

To avoid Azure charges, you should clean up unnecessary resources. When the cluster is no longer needed, use the [az group delete](/cli/azure/group?view=azure-cli-latest#az_group_delete) command to remove the resource group, container registry, and related Azure resources.

```azurecli-interactive
RESOURCE_GROUP_NAME=java-liberty-project
az group delete --name $RESOURCE_GROUP_NAME --yes --no-wait
```

To clean up the resources deployed on AKS on Azure Stack HCI, run the following commands from your local console:

```console
kubectl delete -f https://raw.githubusercontent.com/OpenLiberty/open-liberty-operator/master/deploy/releases/0.7.1/openliberty-app-crd.yaml
kubectl delete -f https://raw.githubusercontent.com/OpenLiberty/open-liberty-operator/master/deploy/releases/0.7.1/openliberty-app-operator.yaml
kubectl delete -f openlibertyapplication.yaml
```

## Next steps

You can learn more from references used in this guide:

* [Azure Kubernetes Service](https://azure.microsoft.com/free/services/kubernetes-service/)
* [Open Liberty](https://openliberty.io/)
* [Open Liberty Operator](https://github.com/OpenLiberty/open-liberty-operator)
* [Open Liberty Server Configuration](https://openliberty.io/docs/ref/config/)
* [Liberty Maven Plugin](https://github.com/OpenLiberty/ci.maven#liberty-maven-plugin)
* [Open Liberty Container Images](https://github.com/OpenLiberty/ci.docker)
* [WebSphere Liberty Container Images](https://github.com/WASdev/ci.docker)