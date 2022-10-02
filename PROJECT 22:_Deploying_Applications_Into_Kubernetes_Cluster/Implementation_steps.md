
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
A Deployment is another layer above ReplicaSets and Pods, It manages the deployment of ReplicaSets and allows for easy updating of a ReplicaSet as well as the ability to roll back to a previous version of deployment. To see it in action:

<img width="1231" alt="Screenshot 2022-09-10 at 3 49 29 PM" src="https://user-images.githubusercontent.com/105562242/193435625-0ee9ef86-d62d-4420-bfac-5d04e714153d.png">

<img width="1231" alt="Screenshot 2022-09-10 at 3 49 29 PM" src="https://user-images.githubusercontent.com/105562242/193435655-b2b0b866-0a8b-4fe5-bb21-3e7841008b89.png">

<img width="1190" alt="Screenshot 2022-09-10 at 3 53 41 PM" src="https://user-images.githubusercontent.com/105562242/193435665-a503ff9a-0c62-48cd-81f8-ace30ddacb49.png">

<img width="1127" alt="Screenshot 2022-09-10 at 4 00 04 PM" src="https://user-images.githubusercontent.com/105562242/193435670-fa2ceb80-a293-4070-a688-7fea40b82716.png">

<img width="1315" alt="Screenshot 2022-09-10 at 4 01 17 PM" src="https://user-images.githubusercontent.com/105562242/193435681-6936bb4e-1130-41cf-bddc-b9ac384efaf2.png">

* We can easily scale our deployment up by specifying the desired number of replicas in an imperative command, like this:

<img width="1085" alt="Screenshot 2022-09-10 at 4 10 39 PM" src="https://user-images.githubusercontent.com/105562242/193435722-17b05917-0727-4d32-8f9e-1e0678ce3b7f.png">

<img width="1136" alt="Screenshot 2022-09-10 at 4 14 02 PM" src="https://user-images.githubusercontent.com/105562242/193435741-a3ba7879-e922-499b-a151-d9e53a8af0e5.png">

#### STEP 5: Deploying Tooling Application With Kubernetes
----------------------------------------------------------------------------------
The tooling application that was containerised with Docker on Project 20, the following shows how the image is pulled and deployed as pods in Kubernetes:

* We will create service for tooling pod to access it from browser 

<img width="1321" alt="Screenshot 2022-09-10 at 4 34 00 PM" src="https://user-images.githubusercontent.com/105562242/193435859-f5208709-77d7-4b6a-8bc7-de372a4bcb23.png">

* Here we can see we are getting error 

<img width="1442" alt="Screenshot 2022-09-10 at 4 34 23 PM" src="https://user-images.githubusercontent.com/105562242/193436027-daccabd2-53e9-41a8-b8cb-0785b05d7d26.png">

* we will deploy mysql and configure service to connect to tooling deployment
* our mysql deployment and mysql service files look like as below :

```
apiVersion: apps/v1
kind: Deployment
metadata:
 name: mysql
 labels:
   tier: mysql-db
spec:
 replicas: 1
 selector:
   matchLabels:
     tier: mysql-db
 template:
   metadata:
     labels:
       tier: mysql-db
   spec:
     containers:
     - name: mysql
       image: mysql:5.7
       env:
       - name: MYSQL_DATABASE
         value: toolingdb
       - name: MYSQL_USER
         value: saikat
       - name: MYSQL_PASSWORD
         value: password123
       - name: MYSQL_ROOT_PASSWORD
         value: password1234
       ports:
       - containerPort: 3306
       
```
```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  selector:
    tier: mysql-db
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
      
```
<img width="1202" alt="Screenshot 2022-09-10 at 5 13 47 PM" src="https://user-images.githubusercontent.com/105562242/193436096-3375c57e-6a73-49f7-baeb-449bd97452bf.png">

<img width="1144" alt="Screenshot 2022-09-10 at 5 14 19 PM" src="https://user-images.githubusercontent.com/105562242/193436104-44aaabdc-cb36-49e6-9a1d-62656b69e93e.png">

<img width="1149" alt="Screenshot 2022-09-10 at 5 20 20 PM" src="https://user-images.githubusercontent.com/105562242/193436116-0cd1aa0e-5e31-4a9b-983e-d2dc2fd09b85.png">

<img width="1496" alt="Screenshot 2022-09-10 at 5 20 40 PM" src="https://user-images.githubusercontent.com/105562242/193436125-4fe1537f-3cc5-44c1-bf42-3c876492737d.png">

<img width="1221" alt="Screenshot 2022-09-10 at 5 21 16 PM" src="https://user-images.githubusercontent.com/105562242/193436130-348bf178-7747-4162-85e9-f2fec49b4c2d.png">

<img width="1269" alt="Screenshot 2022-09-10 at 5 21 28 PM" src="https://user-images.githubusercontent.com/105562242/193436144-1cccc010-79d4-4388-9a8c-58e1639450da.png">

<img width="1343" alt="Screenshot 2022-09-10 at 8 51 58 PM" src="https://user-images.githubusercontent.com/105562242/193436164-d714f753-2eaa-43a0-bd43-b75e4e86d4b7.png">

<img width="1256" alt="Screenshot 2022-09-10 at 8 52 23 PM" src="https://user-images.githubusercontent.com/105562242/193436183-efe14fe8-60a0-4b63-ab68-fc25d7e02e49.png">

#### STEP 6: Using AWS Load Balancer To Access The Nginx Application
-----------------------------------------------------

* We will deploy ningx and try to access it with LoadBlanacer service type 

<img width="1218" alt="Screenshot 2022-09-11 at 12 22 52 AM" src="https://user-images.githubusercontent.com/105562242/193436491-64ccc025-f95d-40a0-a7f0-2cd0b1969db4.png">

<img width="1189" alt="Screenshot 2022-09-11 at 12 41 24 AM" src="https://user-images.githubusercontent.com/105562242/193436499-05f36a5e-af7e-4ebd-9442-9ea4a0e6b507.png">

<img width="1184" alt="Screenshot 2022-09-11 at 12 49 44 AM" src="https://user-images.githubusercontent.com/105562242/193436505-97c45b3b-575b-44fc-89af-32ab3cdf0a11.png">

<img width="1469" alt="Screenshot 2022-09-11 at 12 49 57 AM" src="https://user-images.githubusercontent.com/105562242/193436511-e6adfab4-c8de-4723-a1c8-95b118577ea2.png">

<img width="1395" alt="Screenshot 2022-09-11 at 12 50 13 AM" src="https://user-images.githubusercontent.com/105562242/193436515-cad462dd-12bf-4567-a618-c486adb047fb.png">

<img width="1072" alt="Screenshot 2022-09-11 at 12 50 59 AM" src="https://user-images.githubusercontent.com/105562242/193436517-f7159a60-8d06-46f2-9130-0abb9797f855.png">

<img width="1116" alt="Screenshot 2022-09-11 at 12 54 27 AM" src="https://user-images.githubusercontent.com/105562242/193436521-217e89ef-3bf6-42cd-bde7-ca5d83cc586e.png">


#### STEP 7: PERSISTING DATA FOR PODS
--------------------------------------
* We will deploy nginx pod and modify its default web page. We will see that custom web page only work until that pods are running state. When we will destry the pods and deployment will re-create the pods we will lost the custom config data. 

<img width="1493" alt="Screenshot 2022-09-11 at 12 59 19 AM" src="https://user-images.githubusercontent.com/105562242/193436610-1ece8e7d-1092-4ece-a096-78263c9dc7e0.png">

<img width="1441" alt="Screenshot 2022-09-11 at 1 03 50 AM" src="https://user-images.githubusercontent.com/105562242/193436616-09b58ebd-445f-4404-bf53-8bd42ba1edf2.png">

<img width="1295" alt="Screenshot 2022-09-11 at 1 05 15 AM" src="https://user-images.githubusercontent.com/105562242/193436622-36e71300-1bd8-40c8-b2e9-366f3d58a26a.png">
