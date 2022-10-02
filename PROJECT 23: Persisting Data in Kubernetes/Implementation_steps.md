
#### PERSISTING DATA IN KUBERNETES
----------------------------------------------
#### INTRODUCTION
----------------------------------------------
When a pod dies, any data that is not part of the container image will be lost when the container is restarted because Kubernetes is best at managing stateless applications which means it does not manage data persistence. To ensure data persistent, the PersistentVolume resource is implemented to acheive this.
----------------------------------------------
* If we have deployed K8S clusters with kubeadm then we have to use PersistentVolume and PersistentVolumeClaim.
* For more details please refer this link https://kubernetes.io/docs/concepts/storage/persistent-volumes/
* Here we can see we are running mysql with PersistentVolume and PersistentVolumeClaim configuration to retain data in database

<img width="1102" alt="Screenshot 2022-09-11 at 5 49 08 PM" src="https://user-images.githubusercontent.com/105562242/193438120-795fad8c-a238-4b98-8c8a-911a63137cef.png">

<img width="1390" alt="Screenshot 2022-09-12 at 3 00 16 PM" src="https://user-images.githubusercontent.com/105562242/193438128-80690611-6bab-48f7-8f44-c08b8a1ee56f.png">

* To use EBS from aws we have to run EKS. Here we will cofigure EKS to user EBS to save data. 

<img width="1081" alt="Screenshot 2022-09-14 at 12 48 41 AM" src="https://user-images.githubusercontent.com/105562242/193438182-e8b7ad3d-020a-4bb5-be86-8472be1034be.png">

* When we deploy pods on K8S clusters, first we have to check on wich cluster pods are running and that cluster belongs to which region. We have to create EBS on same region. 

* We have to put the EBS details to pod deployment 

<img width="1214" alt="Screenshot 2022-09-14 at 1 34 24 AM" src="https://user-images.githubusercontent.com/105562242/193438449-59396081-5e70-4084-9201-cd3d3f66de5f.png">

<img width="1474" alt="Screenshot 2022-09-14 at 1 35 52 AM" src="https://user-images.githubusercontent.com/105562242/193438457-dc5bbade-ff21-4a39-bd09-6f5e8eac69a5.png">

<img width="957" alt="Screenshot 2022-09-14 at 1 36 35 AM" src="https://user-images.githubusercontent.com/105562242/193438464-f636f306-4dcc-4e03-b635-77c0bd62e343.png">

* But the problem with this configuration is that when we port forward the service and try to reach the endpoint, we will get a 404 error. This is because mounting a volume on a filesystem that already contains data will automatically erase all the existing data. To solve this issue is by implementing Persistent Volume(PV) and Persistent Volume claims(PVCs) resource.

<img width="1170" alt="Screenshot 2022-09-14 at 1 40 53 AM" src="https://user-images.githubusercontent.com/105562242/193438498-4c749bf2-e0f3-4b19-8d62-77a0d356aff5.png">

<img width="1201" alt="Screenshot 2022-09-14 at 1 40 45 AM" src="https://user-images.githubusercontent.com/105562242/193438485-72efb7f3-e099-44b6-b9d8-224112979315.png">

#### Managing Volumes Dynamically With PV and PVCs
-----------------------------------------------------------
* PVs are resources in the cluster. PVCs are requests for those resources and also act as claim checks to the resource.By default in EKS, there is a default storageClass configured as part of EKS installation which allow us to dynamically create a PV which will create a volume that a Pod will use.

* Verifying that there is a storageClass in the cluster:```$ kubectl get storageclass```
* Creating a manifest file for a PVC, and based on the gp2 storageClass a PV will be dynamically created:

```
apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nginx-volume-claim
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
      storageClassName: gp2
 ```
 * Checking the setup:```$ kubectl get pvc```

<img width="1243" alt="Screenshot 2022-09-14 at 2 02 11 AM" src="https://user-images.githubusercontent.com/105562242/193438617-699814ee-b8aa-4e7a-acef-b3d1713c5089.png">

* The '/tmp/dare' directory will be persisted, and any data written in there will be stored permanetly on the volume, which can be used by another Pod if the current one gets replaced, We will edit our nginx-deployment with PVC.
* Checking the dynamically created PV:```$ kubectl get pv```


<img width="1163" alt="Screenshot 2022-09-14 at 2 05 39 AM" src="https://user-images.githubusercontent.com/105562242/193438699-94f2767c-bbcc-4299-8d7f-9cf28015c838.png">

<img width="1324" alt="Screenshot 2022-09-14 at 2 06 41 AM" src="https://user-images.githubusercontent.com/105562242/193438706-ee501453-1eab-4bc3-87fc-76cc9fca943e.png">

#### Use Of ConfigMap As A Persistent Storage
--------------------------------------------------------
* ConfigMap is an API object used to store non-confidential data in key-value pairs. It is a way to manage configuration files and ensure they are not lost as a result of Pod replacement.
* To demonstrate this, the HTML file that came with Nginx will be used.
* Exec into the container and copying the HTML file:

<img width="1299" alt="Screenshot 2022-09-14 at 2 18 29 AM" src="https://user-images.githubusercontent.com/105562242/193438815-15e54019-21d3-4dd8-befe-26871dabfce3.png">

<img width="1049" alt="Screenshot 2022-09-14 at 2 21 04 AM" src="https://user-images.githubusercontent.com/105562242/193438831-ba470525-9113-4690-8cdc-85ed99d80726.png">

* We will update deployment file to use the configmap in the volumeMounts section

```
cat <<EOF | tee ./nginx-pod-with-cm.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    tier: frontend
spec:
  replicas: 1
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
          - name: config
            mountPath: /usr/share/nginx/html
            readOnly: true
      volumes:
      - name: config
        configMap:
          name: website-index-file
          items:
          - key: index-file
            path: index.html
EOF

```
* Now the index.html file is no longer ephemeral because it is using a configMap that has been mounted onto the filesystem. This is now evident when you exec into the pod and list the /usr/share/nginx/html directory

<img width="1285" alt="Screenshot 2022-09-14 at 2 33 22 AM" src="https://user-images.githubusercontent.com/105562242/193438990-99ef1861-c595-4afa-8517-728871e43692.png">

* To see the configmap created:$ kubectl get configmap
* To see the change in effect, updating the configmap manifest:```$ kubectl edit cm website-index-file```

<img width="989" alt="Screenshot 2022-09-14 at 2 38 36 AM" src="https://user-images.githubusercontent.com/105562242/193439024-f1c68d4a-605a-487e-b9a6-659b4a267f54.png">

<img width="1287" alt="Screenshot 2022-09-14 at 2 40 02 AM" src="https://user-images.githubusercontent.com/105562242/193439038-ce835b81-3626-4717-9c98-86f6b119b33b.png">

<img width="1278" alt="Screenshot 2022-09-14 at 2 40 15 AM" src="https://user-images.githubusercontent.com/105562242/193439045-bc5a4ec9-c905-4fc8-aa74-1fb75619374b.png">
