# How to Deploy SQL Managed Instance on Azure Stack HCI Cluster in Direct Connectivity Mode #
## Prerequisites ##
1. This lab assumes that Azure Stack HCI cluster is already created and registered with Azure. When deploying Azure Stack HCI cluster, ensure that you have enough compute and memory resources on the nodes to deploy SQL Managed Instance. (Note: For this lab, 2-nodes azure stack hci cluster was created with each node asigned with 64GB of RAM and 8 vCPU. Both nodes have access to a cluster shared volume of 1TB). You can easily deploy Azure Stack HCI cluster with MSLab (Ref: https://github.com/DellGEOS/AzureStackHOLs)

2. Second step is to deploy AKS and create an AKS cluster. Please ensure that you have enough resources on the worker nodes to host SQL MI containers. In this lab, 1 linux node pool with 8 vCPU and 32GB RAM (Standard_D8s_v3) was used. Connect Kubernetes cluster to Azure using Azure Arc-enabled Kubernetes.
Once kubernetes cluster is successfully connected to Azure using Azure Arc. Verify azure-arc namespace pods are created
```
kubectl get pods -n azure-arc
```
![image](https://user-images.githubusercontent.com/49147976/201471181-b7869a43-be75-4c33-ba1c-832fb04dfbd4.png)

3. Install Kubectl Utility and Azure CLI on the management station
```
#Run the following command from a windows management station from command prompt
curl.exe -LO "https://dl.k8s.io/release/v1.25.0/bin/windows/amd64/kubectl.exe"
#Install Azure CLI and set execution path from PowerShell
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
$Env:PATH += ";C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\wbin"
$env:PATH -split ';'
```
4. Copy the kube config from one the Nodes of Azure Stack HCI Cluster to management system's C:\Users\UserName\ .kube folder (Note: create a .kube folder if it's not already available)

```
#Run the following PS command from one of the Nodes
Get-AksHciCredential -name <name of the workload cluster>
```
![image](https://user-images.githubusercontent.com/49147976/201463449-631317a6-d6a2-4d1c-a314-bddf07eb74e0.png)
![image](https://user-images.githubusercontent.com/49147976/201464381-b7adb832-43bc-4432-a602-fba0b3600627.png)
![image](https://user-images.githubusercontent.com/49147976/201465681-e7a745f5-0c66-429e-a22f-f362dcea2b18.png)

5. Install arcdata extension for Azure CLI
```
#Run the below commands from PowerShell and login to Azure from Azure CLI using the same azure subscription and credentials used to register the Azure Stack HCI cluster
az login
#Install arcdata extension
az extension add --name arcdata
```
6. Install Azure Data Studio on the management station and Azure Arc extension for Azure Data Studio. Azure Data Studio can be downloaded and installed from this URL: https://learn.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver16. 
- Once the installer in downloaded, proceed to install Azure Data Studio.
- Login to Azure Data Studio using the same azure subscription and credentials used to register the Azure Stack HCI cluster
- Click on Extensions menu on the left and choose to install Azure Arc.

7. Register the Microsoft.AzureArcData provider for the subscription where the Azure Arc-enabled data services will be deployed, as follows:
```
az provider register --namespace Microsoft.AzureArcData
```
## Deploying Data Controller ##
Before we deploy the Data Controller it is important to install the following tools
- Helm Version 3.3+
- Azure CLI extensions

To Install Helm on the Management station running windows using PowerShell:
```
#First install Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
#Install Helm
choco install kubernetes-helm
```
Now Install the az cli extensions
```
az extension add --name k8s-extension
az extension add --name connectedk8s
az extension add --name k8s-configuration
az extension add --name customlocation
```

Let's now proceed to Deploying Data Controller from Azure Portal
