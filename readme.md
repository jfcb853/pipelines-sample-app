# Test app for demonstrating containerizing web app

Super-simple Node web app for containerization demos

## Instructions for use

1. Fork the repo 
2. Clone repo locally
3. Build Docker iamge `docker image build -t <tag> .` from within the root directory of the repo 
4. Push image to container registry
5. Run container/Pod using the created image

## Prerequsites
1. Create an Azure DevOps Organization first 
https://dev.azure.com

2. You must have access to an Azure and GitHub account

## Instructions to use Azure DevOps , create a Project with Pipeline , push the docker to ACR and deploy to AKS using Azure DevOps Pipeline
1. First, add the Azure DevOps extension to our cloud shell session:
```bash
az extension add --upgrade --name azure-devops
```
2. Next, let’s add context for our shell to reference our DevOps organization
```bash
az devops configure --defaults organization=https://dev.azure.com/javiercaparo574/
```

3. From cloud shell, we must login again, so our account can access Azure DevOps. Follow the prompts in the terminal to proceed with the login process:
```bash
az login
```

4. create a new DevOps project:
```bash
az devops project create --name project1
```

5. set the default project to work with:
```bash
az devops configure --defaults project=project1
```

6. create a resource group, in order to logically organize the Azure resources we’ll create in the proceeding steps:
```bash
az group create --name aks-rg --location centralus 
```

7. create a service principal to use for our AKS cluster. Our AKS cluster will use this service principal to access the Azure Container Registry and pull container images.
IMPORTANT: copy the output of the following command to a notepad, you will need it later:
```bash
az ad sp create-for-rbac --skip-assignment
```
Example:
The output includes credentials that you must protect. Be sure that you do not include these credentials in your code or check the credentials into your source control. For more information, see https://aka.ms/azadsp-cli
{
  "appId": "6cdbcbc1-ae3f-4f82-8020-0d4fcd255405",
  "displayName": "azure-cli-2020-12-17-15-47-39",
  "name": "http://azure-cli-2020-12-17-15-47-39",
  "password": "Rs~HncAbvQq-B9bdj0n-9ZVYa~n4p2Z4uy",
  "tenant": "9139da17-5827-48df-81f3-e7122fe090bd"
}


8. create an AKS cluster to deploy our app into (here’s where you use the output from the previous command to paste the “appId” after “–service-principal” and paste the “password” after “–client-secret):
```bash
az aks create -g aks-rg -n myakscluster \
   --node-count 1 \
   --service-principal "6cdbcbc1-ae3f-4f82-8020-0d4fcd255405" \
   --client-secret "Rs~HncAbvQq-B9bdj0n-9ZVYa~n4p2Z4uy" \
   --generate-ssh-keys
 ```  
create an Azure Container Registry (ACR). This will be the repository for our containers used in AKS. The ACR name will have to be globally unique, meaning nobody in the world can have the same name. Time to get creative:
```bash
az acr create -g aks-rg -n jcMyACRforAKS --sku Basic --admin-enabled true
```  

In order to allow AKS to pull images from ACR, we must set our Azure RBAC permissions for the service principal (fill in your unique ACR name here):
```bash
ACR_ID=$(az acr show --name jcMyACRforAKS --resource-group aks-rg --query "id" --output tsv)
CLIENT_ID=$(az aks show -g aks-rg -n myakscluster --query "servicePrincipalProfile.clientId" --output tsv)
az role assignment create --assignee $CLIENT_ID --role acrpull --scope $ACR_ID
``` 

--> Github clone ( the app)
 before create a Personal Access Token in GitHub ( settings --> developers --> pat)
Github PAT ( 522369d5cddce35ae1d2e192196f92bc5c09e334)

Fork this GitHub repo (open this link in a new tab and click “fork”): https://github.com/jfcb853/pipelines-sample-app

Once forked, clone it down to the terminal session within cloud shell with a “git clone” but change the github username to your username:
```bash
~$ git clone https://github.com/<your-github-username-goes-here>/pipelines-sample-app.git

~$ cd pipelines-sample-app
```
now create a pipeline in Azure DevOps:
```bash
az pipelines create --name "pipeline1"
```
    Follow the prompts in your terminal to set up the pipeline:
	
1st & 2do prompt: Enter your GitHub PAT 

3rd prompt: Confirm by entering your github password again; press enter

4th prompt: Enter a service connection name (e.g. pipeline); press enter

5th prompt: Choose [3] to deploy to Azure Kubernetes Service; press enter

6th prompt: Select the k8s cluster you just created; press enter

7th prompt: Choose [2] for the “default” kubernetes namespace; press enter

8th prompt: Select the ACR you just created; press enter

9th prompt: Enter a value for image name (press enter to accept the default); press enter

10th prompt: Enter a value for the service port (press enter to accept the default); press enter

11th prompt: Enter a value for enable review app flow for pull requests (press enter without typing a value)

12th prompt: Choose [1] to continue with generated YAML; press enter

13th prompt: Choose [1] to commit directly to the master branch; press enter

 

CONGRATULATIONS! You’ve created an Azure DevOps Project! Wait about four minutes for the app to build the container, push to ACR, then deploy to AKS.

 

Access your AKS cluster, by getting the kubeconfig credentials:
```bash
az aks get-credentials –-resource-group aks-rg –name myakscluster
```
  --> Merged "myakscluster" as current context in /home/javier/.kube/config

View the Kubernetes resources your project has created:
```bash
~$ kubectl get all
``` 

Enter to the Web app with the Extrenal IP and port 8080
Example:
```bash
http://52.154.159.210:8080/
```
