# How to Deploy SQL Managed Instance on Azure Stack HCI Cluster in Directly Connectivity Mode #
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

8. Verify completed provider registration for Microsoft.ExtendedLocation
```
az provider register --namespace Microsoft.ExtendedLocation
az provider show -n Microsoft.ExtendedLocation -o table
```

9. Enable custom location on the cluster
```
az connectedk8s enable-features -n <clusterName> -g <resourceGroupName> --features cluster-connect custom-locations
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

1. Login to Azure portal where Azure Stack HCI cluster is registered to
2. Go to Azure Arc -> Data Controller and Click on Create Azure Arc Data Controller and select "Azure Arc-enabled kubernetes cluster (Direct connectivity mode)
3. Click on Data Controller Details
4. Select the subscription and resource group where the Azure Kubernetes cluster is deployed.
5. Enter a user friendly name for the data controller
6. Under Custom Location, click on create new and select the cluster (In this case it is the workload cluster), and enter the name for custom location and napespace
   ![image](https://user-images.githubusercontent.com/49147976/201472816-75fa6386-69fe-45fc-8896-0bee4afa9796.png)
7. Under Kubernetes Configuration select the following parameters
   ![image](https://user-images.githubusercontent.com/49147976/201472887-09e51f08-df1d-4cf2-bd0e-44501c84cb20.png)
8. Enter the username and password for Grafana and Kibana dashboards and click Next for Additional Settings
9. Ensure to select a tick for "Enable Metrics upload" option. You can optionally choose to select option "Enable logs upload"
10. Select Tags and Click on Review and Create. Once successfully created the output should look like below
![image](https://user-images.githubusercontent.com/49147976/201474825-4bd4d3af-229d-49f2-a9d3-2929f0dabf75.png)
11. Note that Data controller deployment takes about 15 mins for the status to become ready. Once it is ready it should look like below
![image](https://user-images.githubusercontent.com/49147976/201475383-6240d568-ff76-4ba6-8a28-c53fe323bed9.png)

## Deploying First SQL Managed Instance ###
1. Login to Azure portal where Azure Stack HCI cluster is registered to
2. Go to Azure Arc -> SQL managed instances and Click Create
3. Select the subscription and the respective resource group where the AKS cluster and Data Controller is deployed
4. Enter a user friendly name for the SQL managed instance
5. Choose the custom location created while creating the data controller
6. Select the service type as "Load Balancer"
7. Seelct the appropriate compute + storage SKU. (Note: For testing purposes you can select the check box "For Development use only" under General Tier for which there is no charge)
8. Under the section Instance Compute and Instance STorage you can leave it to the default values
9. Enter SQL managed instance admin name and password
10. Optionally you can enabled AD authentication
11. Click on Review + Create and click create to deploy SQL managed instance.
12. The SQL managed instance takes around 10-15 minutes to fully deploy all the pods. You can monitor the progress of the pods using the "kubectl" command in the namespace. Ensure that all pods are in running state.
![image](https://user-images.githubusercontent.com/49147976/201503098-4196ae32-de4d-4c65-8988-af5a58b9aab9.png)
13. Check the Azure portal to see if the status of sql managed instance is in "Ready" state
![image](https://user-images.githubusercontent.com/49147976/201503221-c035fa2f-0cb0-4229-b58e-cb83b14f15f6.png)

## Accessing the Data Controller and SQL Managed Instance from Azure Data Studio
1. Make sure to login to Azure Data Studio using the same Azure subscription and credentials used to register Azure Stack HCi cluster
2. Open Azure Data Studio from the management station and go to "Connections" tab on the left hand side
3. In the left pane down at the bottom, click on "Azure ARC Controllers" and connect the Data Controller that we created in the above step
![image](https://user-images.githubusercontent.com/49147976/201503439-4dbedc14-6087-48a3-9da1-e1eaa88d0942.png)
4. Once it is connected, it displays the sql mi database that we created under the connection name. Right click on the database and click "Manage". You should see the Azure Arc Dashboard
![image](https://user-images.githubusercontent.com/49147976/201503562-df11fe80-0890-4ec8-925b-4471262febe9.png)
5. Let's connect to the database we created using the "New Connection". Before that we need to get the IP address of load balancer service of the sql db. One way to find out is to use "kubectl get service" command in that namespace
![image](https://user-images.githubusercontent.com/49147976/201503793-d6ef46e8-3dd4-467c-bcc1-179aec97e690.png)
6. Enter the IP address and the login credentials we provided when we created the sql mi, as mentioned below
![image](https://user-images.githubusercontent.com/49147976/201503990-116ea571-ae9a-4d79-afb5-f13d43ae858f.png)
7. Once connected, you should be able to manage the sql managed instance db from Azure Data Studio
![image](https://user-images.githubusercontent.com/49147976/201826150-ccc4494c-c041-4391-907d-f6203480a212.png)

## Accessing the SQL Managed Instance using Grafana and Kibana Dashboards
Grafan and Kibana tools are automatically installed when you select the option to enable "Metrics Upload" option during data controller deployment. There are two ways to fetch the URL for Grafana and Kibana dashboards. One using the kubectl get services command, other using the Azure Data Studio dashboard under the overview tab as shown below
![image](https://user-images.githubusercontent.com/49147976/201827922-c5451773-f503-462a-a786-7ef32f80dea0.png)
### Grafana Dashboard ###
1. Let's click on the link for the Grafana dashboard and when promted, enter the username and password that you provided during the data controller creation wizard. 
2. Once logged in you will be able to see all the metrics related to transations, performance characteristics for the sql managed instace. You can click on search button to view other parameters such as Host Node Mentrics, Pods and Containers Metrics. Below are the snapshots of various metrics.

**SQL Managed Instance Metrics**
![image](https://user-images.githubusercontent.com/49147976/201828856-732519b1-e487-4db4-aff3-a6ea533efa23.png)

**Host Node Metrics**
![image](https://user-images.githubusercontent.com/49147976/201829089-67e6751f-3a44-4ec7-878c-033b17da00bb.png)

**Pods and Containers Metrics**
![image](https://user-images.githubusercontent.com/49147976/201829238-d1d4e1d5-42f5-414c-a897-592c07c89553.png)
### Kibana Dashboard ###
1. Let's click on the link for Kibana dashboard from Azure Data Studio and when prompted enter the username and password that you provided during the data controller deployment. 

**Discover View**
![image](https://user-images.githubusercontent.com/49147976/201830231-a8511a5a-0e89-4491-946c-53f96f5bfa0e.png)



