# flask-app
21-11-2026 | Basic Flask App
```
ACR - Azure Container Registry - Docker Images Storage
AKS - Azure Kubernetes Service -  Managed k8s Service 

Kubernetes - open source production grade container orchaetsration system
Docker - Containerization tool

app
------
lib
------
OS (WIN)

app >> Dockerfile >> Docker Image >> Run >> container (app) 


container (app)


// Docker Desktop - Runtime Container Env 

https://docs.docker.com/desktop/setup/install/windows-install/

// Docker Desktop Running 

// Docker Hub Sign Up

https://hub.docker.com/ 

docker --version
docker login -u username  

// VS Code 

Extensions >> DockerDx, Docker, Dev Containers


// mac 

dev >> code 

app.py 
requirements.txt 

pip install -r requirements.txt 
python app.py 

// windows 

git --version 
choco install git -y 
git clone https://github.com/atulkamble/flask-app.git
cd flask-app
choco install python -y
python --version
pip --version

python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python app.py

http://localhost:5000


git pull
docker buildx build --platform linux/amd64,linux/arm64 -t docker.io/atuljkamble/pythonapp --push .
docker pull atuljkamble/pythonapp:latest
docker run -d -p 5000:5000 atuljkamble/pythonapp:latest
docker container stop 8fb0c3c6fb45
docker container start 8fb0c3c6fb45

http://localhost:5000



az login 

// create ACR Registry

// Setting Access keys 

docker login server URL
docker login aulkamble.azurecr.io

// add username and password 
>> aulkamble

>> FQEDz9ha5db37KnLCYh3pi5am6HpaDFKWtWpldLzUk7PS7khV82xJQQJ99CKACBsN54Eqg7NAAACAZCRAmIR


docker buildx build --platform linux/amd64,linux/arm64 -t aulkamble.azurecr.io/cloudnautic/pythonapp --push .
docker pull aulkamble.azurecr.io/cloudnautic/pythonapp:latest
docker run -d -p 5000:5000 aulkamble.azurecr.io/cloudnautic/pythonapp:latest

AKS 

az aks create --resource-group containers --name mycluster --node-count 2 --enable-managed-identity --generate-ssh-keys

// connect with cluster 

az account set --subscription 08b7b8d4-af42-4972-9517-11ea256ea068

az aks get-credentials --resource-group containers --name mycluster --overwrite-existing

kubectl get nodes 
kubectl get deployments 
kubectl get pods 
kubectl get svc OR kubectl get services 

git pull

cd k8s 
kubectl apply -f deployment.yml 
kubectl apply -f service yml
kubectl get svc 

http://4.173.230.157:5000/
```
