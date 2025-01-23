# Use a canary deployment strategy for Kubernetes

You use the associated workflow to deploy the code and compare the baseline and canary app deployments. Based on the evaluation, you decide whether to promote or reject the canary deployment.

This tutorial uses Docker Registry and Azure Resource Manager service connections to connect to Azure resources. For an Azure Kubernetes Service (AKS) private cluster or a cluster that has local accounts disabled, an Azure Resource Manager service connections is a better way to connect.

# Prerequisites

## Create new Resource Group
```
az group create --name aks-cluster-template_rg001 --location centralus
az group show --name aks-cluster-template_rg001

```

## SSH Public Key for Linux VMs
variable "ssh_public_key" {
  default = "~/.ssh/aks-prod-sshkeys-terraform/aksprodsshkey.pub"
  description = "This variable defines the SSH Public Key for Linux k8s Worker nodes"  
}

## Call get-versions API via command line
az aks get-versions --location centralus -o table

## log on AZ
- az login
- az version -o=json
- az account list

## find choco install azure cli
- where az

# Deploy template

## Azure CLI
```
echo "Enter the same project name:"
read projectName
echo "Enter the template file path and file name:"
read templateFile
resourceGroupName="${projectName}rg"

az deployment group create --name DeployLocalTemplate --resource-group $resourceGroupName --template-file $templateFile --parameters projectName=$projectName --verbose
```

## Portal
- From portal search for "Deploy a", this will lead you to custom deployment (Deploy from a custom deployment).
- Build your own template...
- upload your custom  template and go for it

## Test cluster
```
az aks get-credentials --resource-group aks-cluster-template_rg001 --name aks101cluster
kubectl get nodes
kubectl describe node
kubectl top node aks-agentpool-14651446-vmss000000
```
## Deploy m-tier store application
![Alt text](https://learn.microsoft.com/en-us/azure/aks/learn/media/quick-kubernetes-deploy-rm-template/aks-store-architecture.png#lightbox "a title")

- Store front: Web application for customers to view products and place orders.
- Product service: Shows product information.
- Order service: Places orders.
- Rabbit MQ: Message queue for an order queue.

```
kubectl apply -f aks-store-quickstart.yaml
```

## Test m-tier app
```
kubectl get pods
kubectl get service store-front --watch
```
- Once the EXTERNAL-IP address changes from pending to an actual public IP address, use CTRL-C to stop the kubectl watch process.
- Open a web browser to the external IP address of your service to see the Azure Store app in action.


## Cleanup
```
az group delete --name aks-cluster-template_rg001
```

## References
- [Quickstart deploying AKS cluster using ARM templates](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-rm-template?tabs=azure-cli)
