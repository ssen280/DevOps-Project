
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

* Now Let's try and access the application through the browser, but first we need to create a service for the Nginx pod.
* Creating service for the nginx pod by applying the manifest file:$ kubectl apply -f nginx-service.yaml

<img width="792" alt="Screenshot 2022-09-10 at 2 39 29 PM" src="https://user-images.githubusercontent.com/105562242/193415832-e69c1f22-ce6f-443e-bcef-a755006cdf84.png">

<img width="779" alt="Screenshot 2022-09-10 at 2 44 22 PM" src="https://user-images.githubusercontent.com/105562242/193415840-2f329a31-e629-4633-9487-725c79880502.png">

<img width="931" alt="Screenshot 2022-09-10 at 2 44 40 PM" src="https://user-images.githubusercontent.com/105562242/193415845-c7514593-7cf4-44fd-9f5e-65403e41564f.png">

<img width="1193" alt="Screenshot 2022-09-10 at 2 50 59 PM" src="https://user-images.githubusercontent.com/105562242/193415925-faa7d5be-ca5f-45b1-adde-9a3e77c5475c.png">

* Another way of accessing the Nginx app through browser is the use of NodePort which is a type of service that exposes the service on a static port on the node’s IP address and they range from 30000-32767 by default.

* Editing the nginx-service.yml manifest file to expose the Nginx service in order to be accessible to the browser by adding NodePort as a type of service:

<img width="1271" alt="Screenshot 2022-09-10 at 3 02 49 PM" src="https://user-images.githubusercontent.com/105562242/193416049-3be68d64-159e-4eb1-b3fc-39b761a392a5.png">

* Accessing the nginx application from the browser with the value of the nodeport

<img width="1199" alt="Screenshot 2022-09-10 at 3 02 56 PM" src="https://user-images.githubusercontent.com/105562242/193416069-40ca8e9d-b3d1-4673-91d5-d7cc4098a6cc.png">

#### STEP 3: Creating Deployment
------------------------------------
