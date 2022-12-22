BUILDING ELASTIC KUBERNETES SERVICE (EKS) WITH TERRAFORM
-----------------------------------------------------------------------
EKS is a managed Kubernetes service that makes it easy for you to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane. In this projetc we will be using Terraform to set it up and futher in the project we will use Helm, which is a package manager for Kubernetes is used to deploy multiple applications.


The following outlines the steps:

STEP 1: Configuring The Terraform Module For EKS
------------------------------------------------------------------------

Creating AWS S3 bucket from a CLI to store the Terraform state:

<img width="1164" alt="Screenshot 2022-09-14 at 8 15 44 PM" src="https://user-images.githubusercontent.com/105562242/209226280-3d7e01ee-2236-48e9-8124-09bef82bd5f0.png">

<img width="1169" alt="Screenshot 2022-09-14 at 8 22 43 PM" src="https://user-images.githubusercontent.com/105562242/209226347-2dc9ca90-7738-470d-b123-f014a81aefbd.png">

<img width="1308" alt="Screenshot 2022-09-14 at 9 23 57 PM" src="https://user-images.githubusercontent.com/105562242/209226523-51e166a6-b771-4677-b3d0-5b9520564a44.png">

<img width="1476" alt="Screenshot 2022-09-14 at 9 53 43 PM" src="https://user-images.githubusercontent.com/105562242/209226784-04c198fd-56a8-45df-9578-49199a422c96.png">


STEP 2: Installing Helm From Script
------------------------------------------------------------------------

* Fetching the script:$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
* Changing the permission of the script:$ chmod 700 get_helm.sh
* Executing the script:$ ./get_helm.sh

<img width="1485" alt="Screenshot 2022-09-14 at 10 26 11 PM" src="https://user-images.githubusercontent.com/105562242/209227056-3d4e4e18-da70-4b27-90ce-f08200cb3f20.png">


STEP 3: Deploying Jenkins With Helm
-------------------------------------------------------------------------
* Adding the Jenkins' repository to helm so it can be easily downloaded and deployed:$ helm repo add jenkins https://charts.jenkins.io
* Updating helm repo:$ helm repo update

<img width="1128" alt="Screenshot 2022-09-14 at 10 27 10 PM" src="https://user-images.githubusercontent.com/105562242/209227416-f4d7ae29-70e8-4001-8f6e-3ce71793f8ab.png">

* Installing the chart:$ helm install myjenkins jenkins/jenkins

<img width="1482" alt="Screenshot 2022-09-14 at 10 31 18 PM" src="https://user-images.githubusercontent.com/105562242/209227495-5d88344d-e7e1-4614-8ccf-22b5d1ca7a80.png">

* Running some commands to inspect the installation:

<img width="1463" alt="Screenshot 2022-09-14 at 10 31 36 PM" src="https://user-images.githubusercontent.com/105562242/209227941-a5e09fc4-ecef-42fb-b216-788ea7014ea0.png">

<img width="806" alt="Screenshot 2022-09-14 at 10 32 11 PM" src="https://user-images.githubusercontent.com/105562242/209227971-b3faac9e-a4bb-4dc4-9a12-3f19daad7cc7.png">

<img width="1465" alt="Screenshot 2022-09-14 at 10 32 52 PM" src="https://user-images.githubusercontent.com/105562242/209228055-8215379f-7013-4975-8970-2dca00509193.png">

<img width="1481" alt="Screenshot 2022-09-14 at 10 34 11 PM" src="https://user-images.githubusercontent.com/105562242/209228169-fd0a4f4a-8dbd-4a7b-b9bd-d2f6cb5f15bd.png">


* Testing it:$ kubectl get po
* To display the current context in use:$ kubectl config get-context

<img width="1486" alt="Screenshot 2022-09-14 at 10 36 33 PM" src="https://user-images.githubusercontent.com/105562242/209228522-f6be21c5-2255-46ef-b63c-6837be87d906.png">

* To acquire the Jenkins administrator's password credential:$ kubectl exec --namespace default -it svc/jenkins -c jenkins -- /bin/cat /run/secrets/chart-admin-password && echo

* Port forwarding to access Jenkins from the UI:$ kubectl --namespace default port-forward svc/myjenkins 8080:8080

<img width="1482" alt="Screenshot 2022-09-14 at 10 37 56 PM" src="https://user-images.githubusercontent.com/105562242/209233107-9a641287-2f60-4a09-b373-8648c1ff2bd0.png">

STEP 4: Deploying Artifactory With Helm
---------------------------------------------------------
* Adding the Artifactory's repository to helm:$ helm repo add jfrog https://charts.jfrog.io 
* Updating helm repo:$ helm repo update

<img width="1314" alt="Screenshot 2022-09-14 at 11 02 13 PM" src="https://user-images.githubusercontent.com/105562242/209235498-cbce36b8-bff2-4790-a6e9-d5761e37202d.png">

* Installing the chart:$ helm upgrade --install artifactory --namespace artifactory jfrog/artifactory

<img width="1484" alt="Screenshot 2022-09-14 at 11 02 22 PM" src="https://user-images.githubusercontent.com/105562242/209235594-3241ca7f-2524-4a53-bfe0-8af39f561d9e.png">

STEP 5: Deploying Hashicorp Vault With Helm
----------------------------------------------------------

* Adding the Hashicorp's repository to helm:$ helm repo add hashicorp https://helm.releases.hashicorp.com 
* Updating helm repo:$ helm repo update
* Installing the chart:$ helm install vault hashicorp/vault

<img width="1340" alt="Screenshot 2022-09-14 at 11 05 20 PM" src="https://user-images.githubusercontent.com/105562242/209235863-ad737100-7081-41ac-8f44-4093707ca243.png">

* Inspecting the installation:
<img width="1327" alt="Screenshot 2022-09-14 at 11 07 03 PM" src="https://user-images.githubusercontent.com/105562242/209235952-cc2e1f7d-20be-488a-86be-39b817d3fcd4.png">

* Port forwarding to access Hashicorp vault from the UI:kubectl port-forward svc/vault 8089:8200
* Accessing the app from the browser:http://localhost:8089

<img width="1012" alt="Screenshot 2022-09-14 at 11 10 54 PM" src="https://user-images.githubusercontent.com/105562242/209236831-f1bf2044-110f-4b3b-b680-ed18242affe7.png">


<img width="1271" alt="Screenshot 2022-09-14 at 11 10 48 PM" src="https://user-images.githubusercontent.com/105562242/209236784-ae782641-b9be-43a5-97da-031a5a22b348.png">

STEP 6: Deploying Prometheus With Helm
-------------------------------------------------------------

* Adding the prometheus's repository to helm:$ helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
* Updating helm repo:$ helm repo update
* Installing the chart:$ helm install myprometheus prometheus-community/prometheus

<img width="1483" alt="Screenshot 2022-09-14 at 11 13 38 PM" src="https://user-images.githubusercontent.com/105562242/209237127-c33a2ddd-de17-4e2e-8a35-c9eef2b07671.png">

<img width="1491" alt="Screenshot 2022-09-14 at 11 13 48 PM" src="https://user-images.githubusercontent.com/105562242/209237166-3dae1fff-e823-4fb9-9ce1-e30df21a2d22.png">

* Inspecting the installation shows that there are various pods and services created
* Port forwarding to access prometheus for alert manager from the UI:$ kubectl port-forward svc/myprometheus-alertmanager 8000:80

<img width="1312" alt="Screenshot 2022-09-14 at 11 15 00 PM" src="https://user-images.githubusercontent.com/105562242/209237286-27db310a-7148-41c3-a29d-80c46dd8c67b.png">

* Accessing the app from the browser:http://localhost:8000

<img width="1324" alt="Screenshot 2022-09-14 at 11 14 34 PM" src="https://user-images.githubusercontent.com/105562242/209237373-a8ebac25-3c1c-4afe-8cdc-81b5a11244d1.png">

* Port forwarding to access prometheus for kube state metrics from the UI:$ kubectl port-forward svc/myprometheus-kube-state-metrics 8000:8080


<img width="1490" alt="Screenshot 2022-09-14 at 11 16 20 PM" src="https://user-images.githubusercontent.com/105562242/209237443-b56e7bfe-b25f-4fdf-9679-175930d8d820.png">

* Accessing the app from the browser:http://localhost:8000

<img width="1077" alt="Screenshot 2022-09-14 at 11 16 26 PM" src="https://user-images.githubusercontent.com/105562242/209237494-0783b678-ae21-440c-8bf9-1ba57dec7b65.png">

* Port forwarding to access prometheus for pushgateway from the UI:$ kubectl port-forward svc/myprometheus-pushgateway 8000:9091

<img width="1190" alt="Screenshot 2022-09-14 at 11 17 35 PM" src="https://user-images.githubusercontent.com/105562242/209237550-61a74ad8-49db-44bc-93a9-ff8692923421.png">

* Accessing the app from the browser:http://localhost:8000

<img width="1726" alt="Screenshot 2022-09-14 at 11 17 27 PM" src="https://user-images.githubusercontent.com/105562242/209237617-c69871cd-bcfa-40c5-9b4a-d50b1d4a49cd.png">

STEP 7: Deploying Grafana With Helm
---------------------------------------------------------------

* Adding the grafana's repository to helm:$ helm repo add grafana https://grafana.github.io/helm-charts
* Updating helm repo:$ helm repo update
* Installing the chart:$ helm install grafana-tool grafana/grafana
* Port forwarding to access grafana from the UI:$ kubectl port-forward svc/grafana-tool 8000:80
* Accessing the app from the browser:http://localhost:8000

<img width="1232" alt="Screenshot 2022-09-14 at 11 25 29 PM" src="https://user-images.githubusercontent.com/105562242/209238125-87fc2aa4-53bf-45d9-97ca-41f058f8fe75.png">

<img width="1485" alt="Screenshot 2022-09-14 at 11 25 39 PM" src="https://user-images.githubusercontent.com/105562242/209238145-fe24f8a4-141d-465a-84b5-183d50304b04.png">

<img width="1193" alt="Screenshot 2022-09-14 at 11 27 19 PM" src="https://user-images.githubusercontent.com/105562242/209238224-fa3fce2d-0ce5-4ce9-8007-4e3a9d75cf7f.png">

<img width="1319" alt="Screenshot 2022-09-14 at 11 27 27 PM" src="https://user-images.githubusercontent.com/105562242/209238271-839e0571-3145-4e2b-8f6f-e9a724b2bb5c.png">

STEP 6: Deploying Elasticsearch With Helm
---------------------------------------------------------------

* Adding the Artifactory's repository to helm:$ helm repo add  
* Updating helm repo:$ helm repo update
* Installing the chart:$ helm install ...

