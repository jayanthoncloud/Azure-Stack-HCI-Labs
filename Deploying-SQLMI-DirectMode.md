# How to Deploy SQL Managed Instance on Azure Stack HCI Cluster in Direct Connectivity Mode #
## Prerequisites ##
1. Install Kubectl Utility and Azure CLI on the management station
```
#Run the following command from a windows management station from command prompt
curl.exe -LO "https://dl.k8s.io/release/v1.25.0/bin/windows/amd64/kubectl.exe"
#Install Azure CLI and set execution path from PowerShell
$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; rm .\AzureCLI.msi
$Env:PATH += ";C:\Program Files (x86)\Microsoft SDKs\Azure\CLI2\wbin"
$env:PATH -split ';'
```
2. Copy the kube config from one the Nodes of Azure Stack HCI Cluster to management system's C:\Users\UserName\ .kube folder (Note: create a .kube folder if it's not already available)

```
#Run the following PS command from one of the Nodes
Get-AksHciCredential -name <name of the workload cluster>
```
![image](https://user-images.githubusercontent.com/49147976/201463449-631317a6-d6a2-4d1c-a314-bddf07eb74e0.png)
![image](https://user-images.githubusercontent.com/49147976/201464381-b7adb832-43bc-4432-a602-fba0b3600627.png)
![image](https://user-images.githubusercontent.com/49147976/201465681-e7a745f5-0c66-429e-a22f-f362dcea2b18.png)

3. Install arcdata extension for Azure CLI
```
#Run the below commands from PowerShell and login to Azure from Azure CLI using the same azure subscription and credentials used to register the Azure Stack HCI cluster
az login
#Install arcdata extension
az extension add --name arcdata
```
4. Install Azure Data Studio on the management station and Azure Arc extension for Azure Data Studio. Azure Data Studio can be downloaded and installed from this URL: https://learn.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver16. 
- Once the installer in downloaded, proceed to install Azure Data Studio.
- Login to Azure Data Studio using the same azure subscription and credentials used to register the Azure Stack HCI cluster
- Click on Extensions menu on the left and choose to install Azure Arc.

5. 
