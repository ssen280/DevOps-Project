
#### DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER

----------------------------------------------------------

#### INTRODUCTION
----------------------------------------------------------
This project demonstrates how containerised applications are deployed as pods in Kubernetes and how to access the application from the browser.

#### STEP 1: Creating A Pod For The Nginx Application
----------------------------------------------------------
* Creating nginx pod by applying the manifest file:```kubectl apply -f nginx-pod.yaml```

* nginx-pod.yaml manifest file

<img width="1205" alt="Screenshot 2022-09-10 at 2 34 26 PM" src="https://user-images.githubusercontent.com/105562242/193414899-e2bcc41f-b5f8-41c5-a34a-be539973648f.png">

<img width="1211" alt="Screenshot 2022-09-10 at 2 34 39 PM" src="https://user-images.githubusercontent.com/105562242/193414905-6f2ced54-77dc-408e-8555-36bad8b45ea0.png">

#### STEP 2: Accessing The Nginx Application Through A Browser
------------------------------------------------------------

* First of all, Let's try accessing the Nginx Pod through its IP address from within the Kubernetes cluster. To do this an image that already has curl software installed is needed.

* Running the kubectl command to run the container that has curl software in it as a pod:$ kubectl run curl --image=dareyregistry/curl -i --tty

* Running curl command and pointing it to the IP address of the Nginx Pod

<img width="1383" alt="Screenshot 2022-09-10 at 2 37 07 PM" src="https://user-images.githubusercontent.com/105562242/193415670-9517c7c8-a78f-427b-9136-251e8d055dcc.png">

