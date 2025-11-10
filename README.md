# docker-Java-kubernetes-project


INSTALL MINIKUBE 
***********************************************************

KUBECTL
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.20.4/2021-04-12/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source $HOME/.bashrc
kubectl version --short --client

DOCKER
yum install docker -y
systemctl  start docker
systemctl enable docker

https://minikube.sigs.k8s.io/docs/start/
***********************************************************



INSTALL EKS SETUP
#############################################################
Step1: Take EC2 Instance with t2.MEDIUM instance type
Step2: Create IAM Role with Admin policy for eks-cluster and attach to ec2-instance
Step3: Install kubectl

curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin
cp ./kubectl $HOME/bin/kubectl
export PATH=$HOME/bin:$PATH
echo 'export PATH=$HOME/bin:$PATH' >> ~/.bashrc
source $HOME/.bashrc
kubectl version --short --client

Step4: Install eksctl:
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/bin
eksctl version

Step5: MASTER Cluster creation:
eksctl create cluster --name=eksdemo \
                  --region=us-west-1 \
                  --zones=us-west-1b,us-west-1c \
                  --without-nodegroup 

Step6: Add Iam-Oidc-Providers:
eksctl utils associate-iam-oidc-provider \
    --region us-west-1 \
    --cluster eksdemo \
    --approve 

Allowing the service to connect with EKS


Step7: WORKER NODE Create node-group:
eksctl create nodegroup --cluster=eksdemo \
                   --region=us-west-1 \
                   --name=eksdemo-ng-public \
                   --node-type=t2.medium \
                   --nodes=2 \
                   --nodes-min=2 \
                   --nodes-max=4 \
                   --node-volume-size=10 \
                   --ssh-access \
                   --ssh-public-key=Praveen-test \
                   --managed \
                   --asg-access \
                   --external-dns-access \
                   --full-ecr-access \
                   --appmesh-access \
                   --alb-ingress-access	


 
//eksctl delete nodegroup --cluster=eksdemo --region=us-east-1 --name=eksdemo-ng-public



//eksctl delete cluster --name=eksdemo    --region=us-west-1	




#############################################################


HANDSON


Deploying Java Applications with Docker and Kubernetes

1) Build each project ->> mvn clean install -DskipTests

2) Create docker hub account

3) Build the image in local -> docker build -t praveensingam1994/shopfront:latest .

docker build -t praveensingam1994/productcatalogue:latest .

docker build -t praveensingam1994/stockmanager:latest .

4) Push the image to your docker hub -> docker push praveensingam1994/shopfront:latest 

docker push praveensingam1994/productcatalogue:latest

docker push praveensingam1994/stockmanager:latest

5) Go to kubernetes folder and create the pods -> 

kubectl apply -f shopfront-service.yaml

kubectl apply -f productcatalogue-service.yaml

kubectl apply -f stockmanager-service.yaml

6) minikube service servicename  -> 

minikube service shopfront
minikube service productcatalogue
minikube service stockmanager

7) Hit the url in browser -> 

ORDER TO BUILD AND DEPLOY 

shopfront -> productcatalogue -> stockmanager

Endpoint for product --> /products
Endpoint for stock --> /stocks




#############################################################
RUNNING ON WINDOWS 11 WITH DOCKER DESKTOP
#############################################################

This guide is tailored for running the project on Windows 11 using Docker Desktop (which includes built-in Kubernetes support). It assumes basic knowledge of Java, Maven, Docker, and Kubernetes for learning purposes. The project deploys three microservices: shopfront (Spring Boot), productcatalogue (Dropwizard), and stockmanager (Spring Boot).

Prerequisites
-------------
1. Java 8: Download from Oracle or use OpenJDK. Set JAVA_HOME and add to PATH.
2. Maven: Download from Apache Maven. Set MAVEN_HOME and add %MAVEN_HOME%\bin to PATH. Verify with mvn -version.
3. Docker Desktop: Download and install. Enable Kubernetes in Settings > Kubernetes. Verify with kubectl version.
4. Git: Install if needed. Ensure project is in d:/Projects/Kubernetes_java_Deployment.
5. Docker Hub Account: Create account. Replace 'praveensingam1994' with your username in commands.

Use Command Prompt as Administrator.

Step 1: Verify Project
----------------------
- cd d:\Projects\Kubernetes_java_Deployment
- Ensure all files are present.

Step 2: Build Java Applications
-------------------------------
- cd shopfront && mvn clean install -DskipTests
- cd ..\productcatalogue && mvn clean install -DskipTests
- cd ..\stockmanager && mvn clean install -DskipTests
- Check target/*.jar files created.

Step 3: Build Docker Images
---------------------------
- cd shopfront && docker build -t yourusername/shopfront:latest .
- cd ..\productcatalogue && docker build -t yourusername/productcatalogue:latest .
- cd ..\stockmanager && docker build -t yourusername/stockmanager:latest .
- docker images to verify.

Step 4: Push to Docker Hub
--------------------------
- docker login
- docker push yourusername/shopfront:latest
- docker push yourusername/productcatalogue:latest
- docker push yourusername/stockmanager:latest

Step 5: Deploy to Kubernetes
----------------------------
- cd ..\kubernetes
- kubectl apply -f shopfront-service.yaml
- kubectl apply -f productcatalogue-service.yaml
- kubectl apply -f stockmanager-service.yaml
- kubectl get pods and kubectl get services to verify.

Step 6: Access Services
-----------------------
- kubectl get services for ports.
- Browser: http://localhost:8010 (shopfront), http://localhost:8020/products (productcatalogue), http://localhost:8030/stocks (stockmanager).

Step 7: Monitor and Troubleshoot
--------------------------------
- kubectl logs <pod-name>
- kubectl describe pod/service <name>
- Clean up: kubectl delete -f <file>.yaml

Step 8: Deploy Kubernetes Dashboard
--------------------------------
- kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

Create Admin User Service Account
--------------------------------
- kubectl create serviceaccount admin-user -n kubernetes-dashboard

Grant Cluster Admin Permissions
--------------------------------
- kubectl create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user

Get Access Token
-----------------
- kubectl -n kubernetes-dashboard create token admin-user

Start Dashboard Proxy
---------------------
- kubectl proxy

Access Dashboard
-------------------
- Open your browser and navigate to:http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

Login to Dashboard
-------------------
Select "Token" authentication method
Paste the token from Step 4
Click "Sign in"

Check dashboard pods are running
-----------------------------------

kubectl get pods -n kubernetes-dashboard

Verify service account creation
-------------------------------
kubectl get serviceaccount -n kubernetes-dashboard

Check cluster role binding
--------------------------
kubectl get clusterrolebinding admin-user

Troubleshooting
----------------
Token not working: Generate a new token with kubectl -n kubernetes-dashboard create token admin-user
Proxy connection issues: Ensure kubectl proxy is running in a separate termina
Dashboard not accessible: Verify pods are running with kubectl get pods -n kubernetes-dashboard

Stopping the Dashboard
-----------------------
Press Ctrl + C in the terminal where kubectl proxy is running.

Troubleshooting: If Services Not Accessible (e.g., ERR_CONNECTION_REFUSED)
-------------------------------------------------------------------------
1. Check Pod Status: kubectl get pods - Ensure all pods are Running (not Pending/CrashLoopBackOff).
2. Check Service Exposure: kubectl get services - Verify NodePort services are created with external IPs.
3. Check Logs: kubectl logs <pod-name> - Look for startup errors or health check failures.
4. Port Conflicts: Ensure ports 8010, 8020, 8030 are free. Use netstat -ano | findstr :8010 to check.
5. Kubernetes Context: kubectl config current-context - Should be 'docker-desktop'.
6. Docker Desktop: Ensure Kubernetes is enabled and running in Docker Desktop dashboard.
7. Firewall: Temporarily disable Windows Firewall or allow ports in Advanced Settings.
8. Restart: Restart Docker Desktop and redeploy if issues persist.
9. Health Checks: Pods may fail liveness probes; check YAML paths (e.g., /health for shopfront/stockmanager, /healthcheck for productcatalogue).
10. Image Pull: If pods stuck in ImagePullBackOff, ensure images are pushed and accessible.

Learning Tips
-------------
- Concepts: Pods, Deployments, Services (NodePort), Liveness Probes.
- Experiment: Change replicas in YAML and reapply.
- Resources: Kubernetes Docs, Docker Desktop Kubernetes.
- Issues: Ensure Docker Desktop/Kubernetes enabled.

#############################################################



