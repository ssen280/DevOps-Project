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

   **In below screenshots we are trying to build our custom queries to create alerts**
   
   
<img width="1725" alt="Screenshot 2022-12-31 at 5 21 04 PM" src="https://user-images.githubusercontent.com/105562242/210182151-7d6213b7-fa86-4fb3-bcce-944e9326f2a9.png">

<img width="1725" alt="Screenshot 2022-12-31 at 5 21 20 PM" src="https://user-images.githubusercontent.com/105562242/210182162-9f2ff183-2cd9-4d29-b7b3-078cc408812e.png">

<img width="1725" alt="Screenshot 2022-12-31 at 5 38 54 PM" src="https://user-images.githubusercontent.com/105562242/210182169-f33e2bc7-5f5d-4c66-a50e-fbcc2931d80b.png">

<img width="1727" alt="Screenshot 2022-12-31 at 5 39 11 PM" src="https://user-images.githubusercontent.com/105562242/210182175-3705826f-fb10-4dd0-ada9-68e66c16c199.png">


 **We will create below rule and apply this rule yaml file:**
 
 ```
 apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: main-rules
  namespace: monitor
  labels:
    app: kube-prometheus-stack 
    release: monitoring
spec:
  groups:
  - name: main.rules
    rules:
    - alert: HostHighCpuLoad
      expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 50
      for: 2m
      labels:
        severity: warning
        namespace: monitor
      annotations:
        description: "CPU load on host is over 50%\n Value = {{ $value }}\n Instance = {{ $labels.instance }}\n"
        summary: "Host CPU load high"
    - alert: KubernetesPodCrashLooping
      expr: kube_pod_container_status_restarts_total > 5
      for: 0m
      labels:
        severity: critical
        namespace: monitoring
      annotations: 
        description: "Pod {{ $labels.pod }} is crash looping\n Value = {{ $value }}"
        summary: "Kubernetes pod crash looping"
        
   ```

<img width="1275" alt="Screenshot 2023-01-02 at 12 52 24 AM" src="https://user-images.githubusercontent.com/105562242/210182347-3ecf859b-8529-49bc-92c8-3b1e0c0f5cbf.png">

<img width="1225" alt="Screenshot 2022-12-31 at 5 43 19 PM" src="https://user-images.githubusercontent.com/105562242/210182449-c1c2bd24-7c8c-47ec-b285-e44215ddd8d8.png">

 **In below screenshot we can see configuration is updated in pod logs**
 
 <img width="1302" alt="Screenshot 2022-12-31 at 10 26 40 PM" src="https://user-images.githubusercontent.com/105562242/210182547-480c5295-c8b9-46b6-a54f-1cf345ad7496.png">

 **In below screenshot we can see our custom configured alert rules are visible in portel**
 
 <img width="1725" alt="Screenshot 2022-12-31 at 5 51 25 PM" src="https://user-images.githubusercontent.com/105562242/210182610-7d34299b-ef4f-4d2e-a7c0-129bab53081b.png">

<img width="1726" alt="Screenshot 2022-12-31 at 5 51 57 PM" src="https://user-images.githubusercontent.com/105562242/210182620-ddd3e022-8869-48f7-9215-9bb70b34da6e.png">

**There are two alerts mode : 1. pending 2.firing. Prometheus keep alerts in pending state forbore firing alerts.**
**To check if our alerts are working or not we will download cpustress from docker and run it as k8s command as pod**

<img width="1227" alt="Screenshot 2022-12-31 at 6 25 42 PM" src="https://user-images.githubusercontent.com/105562242/210183038-7da4c989-073a-46f7-b538-677b5738f509.png">


<img width="1711" alt="Screenshot 2022-12-31 at 6 12 04 PM" src="https://user-images.githubusercontent.com/105562242/210183136-b913d513-427e-47dc-89dc-d50e0f8c54f6.png">

<img width="1723" alt="Screenshot 2022-12-31 at 6 19 21 PM" src="https://user-images.githubusercontent.com/105562242/210183145-5c4073a6-42b9-4bea-88a9-8c5bcbd96657.png">

<img width="1727" alt="Screenshot 2022-12-31 at 6 19 42 PM" src="https://user-images.githubusercontent.com/105562242/210183160-b92fff65-ecb3-4b6b-a12a-7e1ff482dfa6.png">


**We will configure email notification when alerts will be firing : for that we have to use alart manager**
**Here we will run alert manager service and we can access alart manager portal with relevent port and we will get configuration details from alert manager portal and use the same to configure our own configuration*

<img width="1228" alt="Screenshot 2023-01-02 at 7 40 39 AM" src="https://user-images.githubusercontent.com/105562242/210272053-f35a7ba6-35f8-43ca-b39a-663c6a4ef5b6.png">

**We will get api details from below link**

https://docs.openshift.com/container-platform/4.11/rest_api/monitoring_apis/monitoring-apis-index.html#alertmanagerconfig-monitoring-coreos-comv1beta1

**below is our custom alert manager configuration**

```
apiVersion: monitoring.coreos.com/v1alpha1
kind: AlertmanagerConfig
metadata:
  name: main-rules-alert-config
  namespace: monitor
spec:
  route:
    receiver: 'email'
    repeatInterval: 30m
    routes:
    - matchers:
      - name: alertname
        value: HostHighCpuLoad
    - matchers:
      - name: alertname
        value: KubernetesPodCrashLooping
      repeatInterval: 10m
  receivers:
  - name: 'email'
    emailConfigs:
    - to: 'milansenaws@gmail.com'
      from: 'milansenaws@gmail.com'
      smarthost: 'smtp.gmail.com:587'
      authUsername: 'milansenaws@gmail.com'
      authIdentity: 'milansenaws@gmail.com'
      authPassword:
       name: gmail-auth
       key: password
```

<img width="1024" alt="Screenshot 2023-01-03 at 1 07 59 AM" src="https://user-images.githubusercontent.com/105562242/210272499-ec8a7c24-ccb7-4bbe-8cbb-7709e5a0a2f4.png">

**We can not expose our gmail username and password openly hence we will configure secret to store our username and email password as base64 format**
**We will apply these yaml files**

```
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: gmail-auth
  namespace: monitor
data:
  password: QnVOdFkwbyExMjM0
```

<img width="718" alt="Screenshot 2023-01-03 at 1 10 01 AM" src="https://user-images.githubusercontent.com/105562242/210272630-95affbfd-9f92-4827-8afb-3f560c3c8bed.png">

<img width="1235" alt="Screenshot 2023-01-02 at 8 35 03 AM" src="https://user-images.githubusercontent.com/105562242/210272710-5364fdca-8abb-4337-bd24-28b61e836df8.png">

<img width="1205" alt="Screenshot 2023-01-02 at 8 36 38 AM" src="https://user-images.githubusercontent.com/105562242/210272823-f1973694-fef4-4313-94bf-00db4a6dbd2f.png">

**We can see our configuration updated to default configuration on alert manager portal**

<img width="1159" alt="Screenshot 2023-01-02 at 8 40 48 AM" src="https://user-images.githubusercontent.com/105562242/210272840-0b1e216e-9cf3-475b-ba82-ee22b00536e3.png">

**Now we will check if we are getting alart motification to our gmail inbox, to do that we will run again cpu stress pod again**



