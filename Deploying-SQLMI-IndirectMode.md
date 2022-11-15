# How to Deploy SQL Managed Instance on Azure Stack HCI Cluster in Indirectly Connected Mode #
## Prerequisites #
Note: Follow the same prerequisites documented in the github page: [Prerequisites for SQL Managed Instance Directly Connected mode](https://github.com/jayanthyk/Azure-Stack-HCI-Labs/blob/main/Deploying-SQLMI-DirectMode.md#prerequisites)
1. Login to Azure Portal -> Azure Arc -> Azure Arc Data Controller -> Click on Create
2. Select the option **Any other Kubernetes cluster (Indirect connectivity mode)** and click on "Data Controller Details"
3. Select the corresponding Azure subscription and resourcegroup under which AKS cluster is deployed. Enter the name for the data controller and location
4. Click "Next: Open in Azure Data Studio"
5. Copy the URL and paste it in the browser from the same system where Azure Data Studio is installed
6. When propted, click on Open to allow an extension to open this URI pop up
7. Step 1: Click Next on "Deployment pre-requisites" page
![image](https://user-images.githubusercontent.com/49147976/201912469-69f856cc-8cdd-426b-83fc-92b66b483271.png)
8. Step 2: Select Connectivity mode as "Indirect" and leave the default path of the kube config and cluster context
9. ![image](https://user-images.githubusercontent.com/49147976/201912300-4013f1f3-3f53-4892-87a0-530d04ca64e3.png)
10. Step 3: Azure Configuration details. Ensure it is correct
![image](https://user-images.githubusercontent.com/49147976/201912726-d18373ed-d638-43d4-9ac3-e1f2ed7d3d64.png)
11. Step 4: Controller Configuration: Enter the name of the controller, namespace and credentials for Metrics and Logs dashboards
![image](https://user-images.githubusercontent.com/49147976/201913352-15fc8bb3-3e12-4307-8041-0f456f809412.png)
Step 5: Review Configuration. Verify the details entered are accurate 
![image](https://user-images.githubusercontent.com/49147976/201913576-4cf84aa9-6bde-44bb-97f0-8bfbdd108ea4.png)
Click on Deploy
If this is the first time running runbook on Azure Data Studio, you need to configure Python Runtime. Leave it as defaults and click Next
![image](https://user-images.githubusercontent.com/49147976/201913992-e1f3419e-4316-4ece-8085-77b5f5b24dbf.png)
Click on Install for **Install Dependencies** step
Click on **Run all** button in the in Notebook
