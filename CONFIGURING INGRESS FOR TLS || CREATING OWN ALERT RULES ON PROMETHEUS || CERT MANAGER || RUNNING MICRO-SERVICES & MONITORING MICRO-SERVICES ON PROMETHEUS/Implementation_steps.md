IN THIS PROJECT WE WILL USE BELOW TOOLS AND WILL TRY TO ACHIVE BELOW OBJECTIVES 
------------------------------------------------------------------------------------------

* HELM
* INGRESS
* CERT-MANAGER
* PROMETHEUS & GRAFANA
* ARTIFACTORY
* JENKINS
* MICRO-SERVICES WEBSITE

-------------------------------------------------------------------------------------------

1. **We will spin up EKS clusters and deploy micro-services website** 

   Link of micro-services website:- https://github.com/ssen280/microservices-demo


<img width="1096" alt="Screenshot 2023-01-01 at 11 42 48 PM" src="https://user-images.githubusercontent.com/105562242/210180765-f70ba23e-f290-4390-b631-1bfd7d836dda.png">


2. **We will create namespace website to deploy this website to this specific website ( Command already present in above mentioned github link for deployment)**

   <img width="1013" alt="Screenshot 2023-01-01 at 11 46 44 PM" src="https://user-images.githubusercontent.com/105562242/210180854-0abf8667-e0bd-4ac9-8c9d-e85cc6f2c843.png">

3. **Below is arcitecture of micro-services website and ports used by it micro services**

<img width="1125" alt="Screenshot 2023-01-01 at 11 49 53 PM" src="https://user-images.githubusercontent.com/105562242/210180917-130f8f5c-ecfa-40f3-99ec-b47b4d893459.png">

<img width="1289" alt="Screenshot 2023-01-01 at 11 54 28 PM" src="https://user-images.githubusercontent.com/105562242/210181097-7092a3c5-cebd-4e85-95fb-4546dc3baaa6.png">

-----------------------------------------------------------------------------------------------
 **We will install prometheus and configure custom alart rules and monitor our own mico-services**
 
<img width="1183" alt="Screenshot 2022-12-31 at 3 09 42 PM" src="https://user-images.githubusercontent.com/105562242/210181560-44aa2853-14e9-4407-9279-e7a8c63ea2f0.png">

<img width="1408" alt="Screenshot 2023-01-02 at 12 20 27 AM" src="https://user-images.githubusercontent.com/105562242/210181647-24cb969c-121c-4d9b-a2c9-d9f5749bb88e.png">

<img width="1531" alt="Screenshot 2023-01-02 at 12 39 10 AM" src="https://user-images.githubusercontent.com/105562242/210182075-ed7b9b53-a8c9-424e-bb3f-2fc7afc0d5be.png">


1. **We will create our own custom alert rules : In below screenshot we can see below rule format, we will use it to create our own alert rules.**

<img width="1723" alt="Screenshot 2023-01-02 at 12 24 13 AM" src="https://user-images.githubusercontent.com/105562242/210181743-e043c673-afe7-4558-bb01-9d904cd06fbc.png">

2. **We will create custom rules to monitor our EKS resources, We will use PromQL(Prometheus Query Language)which prometheus use, to make our alert rules**

    * When pods will restart 5 times, it will generate alerts
    * when CPU utilization is reach certain level, it will generate alerts

   **In below screenshots we are trying to build our custom queries to see if**
