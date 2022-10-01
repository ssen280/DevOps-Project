#### SETTING UP A KUBERNETES CLUSTER FROM GROUND UP (THE HARD WAY)

---------------------------------------------------
#### INTRODUCTION

---------------------------------------------------
In this project, a Kubernetes cluster is manually setup from the scratch without any automated helpers in order to better understand each aspect of spinning up a Kubernetes cluster as each components are manually installed from scratch.

<img width="1123" alt="Screenshot 2022-10-01 at 2 00 12 AM" src="https://user-images.githubusercontent.com/105562242/193351690-ea15b16f-5746-41ae-9801-4c40ab6b9b01.png">

We will create 3 EC2 Instances, and in the end, we will have the following parts of the cluster properly configured:

* One Kubernetes Master
* Two Kubernetes Worker Nodes
* Configured SSL/TLS certificates for Kubernetes components to communicate securely
* Configured Node Network
* Configured Pod Network

The following outlines the steps:

#### STEP 1: Installing Kubectl On The Local Machine(Linux)
---------------------------------------------------------------
* Downloading the binary: $ wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
* Making it executable:$ chmod +x kubectl
* Moving the file to the Bin directory:$ sudo mv kubectl /usr/local/bin/
* Verifying that kubectl version 1.21.0 or higher is installed:$ kubectl version --client

<img width="1539" alt="Screenshot 2022-09-08 at 7 46 11 AM" src="https://user-images.githubusercontent.com/105562242/193353585-11e36b01-0be8-4a1e-8ce6-d5116b783308.png">

#### SIEP 2: Installing CFSSL And CFSSLJSON
------------------------------------------------------------
The cfssl is an open source tool by Cloudflare used to setup a Public Key Infrastructure(PKI) for generating, signing and bundling TLS certificates

Downloading the binary:

```
$ wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
  https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
  
```
* Making it executable: ```$ chmod +x cfssl cfssljson```
* Moving the file to the bin directory: ```$ sudo mv cfssl cfssljson /usr/local/bin/ ```

<img width="1541" alt="Screenshot 2022-09-08 at 7 47 23 AM" src="https://user-images.githubusercontent.com/105562242/193354078-ec3cbfa5-be3c-47a4-b4c6-1a4e1e1afcbc.png">

#### STEP 3: Configuring The Network Infrastructure
-------------------------------------------------------

* Creating a directory named k8s-cluster-from-ground-up and changing directory:``` $ mkdir k8s-cluster-from-ground-up && cd k8s-cluster-from-ground-up ```
* Creating a VPC and storing its ID as a variable:

```
$ VPC_ID=$(aws ec2 create-vpc \
--cidr-block 172.31.0.0/16 \
--output text --query 'Vpc.VpcId'
)
```
* Tagging the VPC so that it is named:

```
$ NAME=k8s-cluster-from-ground-up

$ aws ec2 create-tags \
  --resources ${VPC_ID} \
  --tags Key=Name,Value=${NAME}
```
* Enabling DNS support for the VPC:

```
$ aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-support '{"Value": true}'
```
* Enabling DNS support for hostnames:

```
$ aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-hostnames '{"Value": true}'
```
* Setting the required region:```$ AWS_REGION=us-east-1```
* Configuring DHCP Options Set:

```
$ DHCP_OPTION_SET_ID=$(aws ec2 create-dhcp-options \
  --dhcp-configuration \
    "Key=domain-name,Values=$AWS_REGION.compute.internal" \
    "Key=domain-name-servers,Values=AmazonProvidedDNS" \
  --output text --query 'DhcpOptions.DhcpOptionsId')
```
* Tagging the DHCP Option set:

```
$ aws ec2 create-tags \
  --resources ${DHCP_OPTION_SET_ID} \
  --tags Key=Name,Value=${NAME}
```
* Associating the DHCP Option set with the VPC:
```
$ aws ec2 associate-dhcp-options \
  --dhcp-options-id ${DHCP_OPTION_SET_ID} \
  --vpc-id ${VPC_ID}
```

<img width="1371" alt="Screenshot 2022-09-08 at 7 50 01 AM" src="https://user-images.githubusercontent.com/105562242/193355507-1731af46-2fb9-4a64-9161-7ce8fff13417.png">

* Creating the Subnet:

```
$ SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 172.31.0.0/24 \
  --output text --query 'Subnet.SubnetId')
```
* Tagging the Subnet:

```
$ aws ec2 create-tags \
  --resources ${SUBNET_ID} \
  --tags Key=Name,Value=${NAME}
  
```
* Creating the Internet Gateway and tagging it:
```
$ INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
  --output text --query 'InternetGateway.InternetGatewayId')

$ aws ec2 create-tags \
  --resources ${INTERNET_GATEWAY_ID} \
  --tags Key=Name,Value=${NAME}
```
* Attaching the Internet Gateway to the VPC:
```
$ aws ec2 attach-internet-gateway \
  --internet-gateway-id ${INTERNET_GATEWAY_ID} \
  --vpc-id ${VPC_ID}
```
* Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway:
```
ROUTE_TABLE_ID=$(aws ec2 create-route-table \
  --vpc-id ${VPC_ID} \
  --output text --query 'RouteTable.RouteTableId')
aws ec2 create-tags \
  --resources ${ROUTE_TABLE_ID} \
  --tags Key=Name,Value=${NAME}
aws ec2 associate-route-table \
  --route-table-id ${ROUTE_TABLE_ID} \
  --subnet-id ${SUBNET_ID}
aws ec2 create-route \
  --route-table-id ${ROUTE_TABLE_ID} \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id ${INTERNET_GATEWAY_ID}
  
```
<img width="1386" alt="Screenshot 2022-09-08 at 7 52 47 AM" src="https://user-images.githubusercontent.com/105562242/193356312-60e6f0d1-f45e-4c82-ba61-c400b4356960.png">

* Configuring Security Groups

```
# Create the security group and store its ID in a variable
SECURITY_GROUP_ID=$(aws ec2 create-security-group \
  --group-name ${NAME} \
  --description "Kubernetes cluster security group" \
  --vpc-id ${VPC_ID} \
  --output text --query 'GroupId')

# Create the NAME tag for the security group
aws ec2 create-tags \
  --resources ${SECURITY_GROUP_ID} \
  --tags Key=Name,Value=${NAME}

# Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

# # Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes
aws ec2 authorize-security-group-ingress \
    --group-id ${SECURITY_GROUP_ID} \
    --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'
# Create inbound traffic to allow connections to the Kubernetes API Server listening on port 6443
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 6443 \
  --cidr 0.0.0.0/0

# Create Inbound traffic for SSH from anywhere (Do not do this in production. Limit access ONLY to IPs or CIDR that MUST connect)
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Create ICMP ingress for all types
aws ec2 authorize-security-group-ingress \
  --group-id ${SECURITY_GROUP_ID} \
  --protocol icmp \
  --port -1 \
  --cidr 0.0.0.0/0
  
```
<img width="1130" alt="Screenshot 2022-09-08 at 7 56 45 AM" src="https://user-images.githubusercontent.com/105562242/193356607-b83cb3d7-1f48-434a-b488-0c375ba74764.png">

* Creating a network Load balancer:
```
LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer \
  --name ${NAME} \
  --subnets ${SUBNET_ID} \
  --scheme internet-facing \
  --type network \
  --output text --query 'LoadBalancers[].LoadBalancerArn')
```
* Creating a target group for it
```
TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
  --name ${NAME} \
  --protocol TCP \
  --port 6443 \
  --vpc-id ${VPC_ID} \
  --target-type ip \
  --output text --query 'TargetGroups[].TargetGroupArn')
```
* Registering targets - though there are no real targets but the private IP addresses are selected so that when the nodes become available they will be used as targets:

```
aws elbv2 register-targets \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --targets Id=172.31.0.1{0,1,2}
```
* Creating a listener to listen for requests and forward to the target nodes on TCP port 6443
```
aws elbv2 create-listener \
--load-balancer-arn ${LOAD_BALANCER_ARN} \
--protocol TCP \
--port 6443 \
--default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} \
--output text --query 'Listeners[].ListenerArn'
```
* Retrieving the Kubernetes Public address and storing it

```
KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
--load-balancer-arns ${LOAD_BALANCER_ARN} \
--output text --query 'LoadBalancers[].DNSName')
```

<img width="1354" alt="Screenshot 2022-09-08 at 7 58 14 AM" src="https://user-images.githubusercontent.com/105562242/193357346-50eaae25-3c3a-4989-9468-80a99b6bcacf.png">

#### STEP 4: Creating Compute Resources
-----------------------------------------------

* Retrieving an image ID to create EC2 instances:

```
IMAGE_ID=$(aws ec2 describe-images --owners 099720109477 \
  --filters \
  'Name=root-device-type,Values=ebs' \
  'Name=architecture,Values=x86_64' \
  'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-xenial-16.04-amd64-server-*' \
  | jq -r '.Images|sort_by(.Name)[-1]|.ImageId')
```
* Creating an SSH Key-Pair:

```
$ mkdir -p ssh

$ aws ec2 create-key-pair \
  --key-name ${NAME} \
  --output text --query 'KeyMaterial' \
  > ssh/${NAME}.id_rsa

$ chmod 600 ssh/${NAME}.id_rsa

```
* Creating 3 Master nodes:

```
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name ${NAME} \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.1${i} \
    --user-data "name=master-${i}" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=${NAME}-master-${i}"
done

```
<img width="1335" alt="Screenshot 2022-09-08 at 8 00 30 AM" src="https://user-images.githubusercontent.com/105562242/193358356-8c82fc42-0a3f-4735-979b-6c04be97df62.png">

* Creating 3 worker nodes:

```
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name ${NAME} \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.2${i} \
    --user-data "name=worker-${i}|pod-cidr=172.20.${i}.0/24" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=${NAME}-worker-${i}"
done

```
#### STEP 5: Preparing The Self-Signed Certificate Authority And Generating The TLS Certificates
-------------------------------------------------------------

The PKI Infrastructure is provisioned using cfssl which will have a Certificate Authority which will then generate certificates for all the individual components:

* Creating a directory called ca-authority and cd into it: ```$ mkdir ca-authority && cd ca-authority```
* Generating the CA configuration file, Root Certificate, and Private key:

```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "DAREY.IO DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}

```
* Provisioning Client/Server certificates for all the components that will communicate with the api-server by using the root CA to request more certificates which the different Kubernetes components, i.e. clients and server, will use to have encrypted communication.

* Generating the Certificate Signing Request (CSR), Private Key and the Certificate for the Kubernetes Master Nodes (api-server).

```
{
cat > master-kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
   "hosts": [
   "127.0.0.1",
   "172.31.0.10",
   "172.31.0.11",
   "172.31.0.12",
   "ip-172-31-0-10",
   "ip-172-31-0-11",
   "ip-172-31-0-12",
   "ip-172-31-0-10.${AWS_REGION}.compute.internal",
   "ip-172-31-0-11.${AWS_REGION}.compute.internal",
   "ip-172-31-0-12.${AWS_REGION}.compute.internal",
   "${KUBERNETES_PUBLIC_ADDRESS}",
   "kubernetes",
   "kubernetes.default",
   "kubernetes.default.svc",
   "kubernetes.default.svc.cluster",
   "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "English",
      "O": "Kubernetes",
      "OU": "Saikat",
      "ST": "India"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  master-kubernetes-csr.json | cfssljson -bare master-kubernetes
}

```
* Generating Client Certificate and Private Key for kube-scheduler

```
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-scheduler",
      "OU": "DAREY.IO DEVOPS",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}

```
* Generating Client Certificate and Private Key for kube-proxy

```
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "India",
      "O": "system:node-proxier",
      "OU": "Saikat",
      "ST": "India"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}

```
* Generating Client Certificate and Private Key for kube-controller-manager

```
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "India",
      "O": "system:kube-controller-manager",
      "OU": "NA",
      "ST": "India"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}

```

* Generating Client Certificate and Private Key for kubelet
```
for i in 0 1 2; do
  instance="${NAME}-worker-${i}"
  instance_hostname="ip-172-31-0-2${i}"
  cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance_hostname}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "India",
      "O": "system:nodes",
      "OU": "NA",
      "ST": "India"
    }
  ]
}
EOF

  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')

  internal_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PrivateIpAddress')

  cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=ca-config.json \
    -hostname=${instance_hostname},${external_ip},${internal_ip} \
    -profile=kubernetes \
    ${NAME}-worker-${i}-csr.json | cfssljson -bare ${NAME}-worker-${i}
done

```
* Generating Client Certificate and Private Key for kubernetes admin user

```
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "India",
      "O": "system:masters",
      "OU": "NA",
      "ST": "INDIA"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}

```
 * Generating Client Certificate and Private Key for Token Controller
 
It is a part of the Kubernetes Controller Manager responsible for generating and signing service account tokens which are used by pods or other resources to establish connectivity to the api-server
 
 ```
 {

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "IN",
      "L": "India",
      "O": "Kubernetes",
      "OU": "na",
      "ST": "India"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
}

```
#### STEP 6: Distributing The Client And Server Certificates
------------------------------------------------------

* Sending all the client and server certificates to their respective instances. Starting from worker nodes

```
for i in 0 1 2; do
  instance="${NAME}-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/${NAME}.id_rsa \
    ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
```
<img width="1541" alt="Screenshot 2022-09-08 at 8 13 40 AM" src="https://user-images.githubusercontent.com/105562242/193368809-65863b5e-6fec-4e99-ae1f-35ec9ddb0d10.png">

* For the master nodes:

<img width="1541" alt="Screenshot 2022-09-08 at 8 14 12 AM" src="https://user-images.githubusercontent.com/105562242/193368834-f51b90d2-8921-4416-9c0b-f2e1517467d1.png">

#### STEP 7: Using KUBECTL To Generate Kubernates Configuration Files For Authentication
---------------------------------------------------
* Creating an environment variables for reuse by multiple commands:```$ KUBERNETES_API_SERVER_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')```

* Generating the kubelet kubeconfig file:

```
for i in 0 1 2; do

instance="${NAME}-worker-${i}"
instance_hostname="ip-172-31-0-2${i}"

 # Set the kubernetes cluster in the kubeconfig file
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://$KUBERNETES_API_SERVER_ADDRESS:6443 \
    --kubeconfig=${instance}.kubeconfig

# Set the cluster credentials in the kubeconfig file
  kubectl config set-credentials system:node:${instance_hostname} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

# Set the context in the kubeconfig file
  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:node:${instance_hostname} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done

```
* ls -ltr *.kubeconfig
* Generating the kube-proxy kubeconfig

<img width="1538" alt="Screenshot 2022-09-08 at 8 16 05 AM" src="https://user-images.githubusercontent.com/105562242/193369437-6962d2fd-d8f4-4724-b5c6-717c14a54e6e.png">

```
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}

```
* Generating the Kube-Controller-Manager kubeconfig:

```
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}

```
* Generating the Kube-Scheduler Kubeconfig:

```
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}

```
* Generating the kubeconfig file for the admin user

```
{
  kubectl config set-cluster ${NAME} \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=${NAME} \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}

```
* Distributing the files to thier respective servers using scp and for loop:

<img width="1539" alt="Screenshot 2022-09-08 at 8 39 11 AM" src="https://user-images.githubusercontent.com/105562242/193369949-c2c239c8-954d-40a2-a188-cf1ec91b740c.png">

* For Worker nodes

<img width="1537" alt="Screenshot 2022-09-08 at 8 48 50 AM" src="https://user-images.githubusercontent.com/105562242/193370272-0146d294-ffca-4ef5-a986-9f8da2d849e0.png">

#### STEP 8: Preparing The ETCD Database For Encryption At Rest
-----------------------------------------------------------------
Kubernetes uses etcd (A distributed key value store) to store variety of data which includes the cluster state, application configurations, and secrets but since the data in it is stored as plain text, therefore the etcd is encrypted as follows

* Generating encryption key and encoding it using base64:ETCD_ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
* Creating an encryption-config.yaml file

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
 - resources:
     - secrets
   providers:
     - aescbc:
         keys:
           - name: key1
             secret: ${ETCD_ENCRYPTION_KEY}
     - identity: {}
EOF

```
* Sending the encryption-config.yaml file to the master nodes using scp and a for loop

<img width="1541" alt="Screenshot 2022-09-08 at 8 52 14 AM" src="https://user-images.githubusercontent.com/105562242/193374573-1217fc7e-5160-4824-b769-ed6dea6712f4.png">

* Bootstrapping etcd cluster using tmux to work with multiple terminal sessions simultaneously. Opening 3 panes and ssh into the 3 master nodes and setting the synchronize-panes on

##### For Master node 1

```
master_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=${NAME}-master-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_1_ip}
```

##### For Master node 2
```
master_2_ip=$(
aws ec2 describe-instances \
--filters "Name=tag:Name,Values=${NAME}-master-1" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_2_ip}

```

##### For Master node 3

```
master_3_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=${NAME}-master-2" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${master_3_ip}

```
* Downloading and installing etcd

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.15/etcd-v3.4.15-linux-amd64.tar.gz"
  
```
* Extracting and installing the etcd server and the etcdctl command line utility:

```
{
tar -xvf etcd-v3.4.15-linux-amd64.tar.gz
sudo mv etcd-v3.4.15-linux-amd64/etcd* /usr/local/bin/
}

```
* Configuring the etcd server
```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem master-kubernetes-key.pem master-kubernetes.pem /etc/etcd/
}

```
* The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieving the internal IP address for the current compute instance:

```
export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)
```
* Each etcd member must have a unique name within an etcd cluster. Set the etcd name to node Private IP address so it will uniquely identify the machine:

```
ETCD_NAME=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo ${ETCD_NAME}

```
* Creating the etcd.service systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://172.31.0.10:2380,master-1=https://172.31.0.11:2380,master-2=https://172.31.0.12:2380 \\
  --cert-file=/etc/etcd/master-kubernetes.pem \\
  --key-file=/etc/etcd/master-kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/master-kubernetes.pem \\
  --peer-key-file=/etc/etcd/master-kubernetes-key.pem \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```
* Starting and enabling the etcd Server

```
{
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
}

```

* Verifying the etcd installation

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/master-kubernetes.pem \
  --key=/etc/etcd/master-kubernetes-key.pem
  
```
* Checking the etcd service status:```$ systemctl status etcd ```

<img width="1542" alt="Screenshot 2022-09-08 at 9 14 49 AM" src="https://user-images.githubusercontent.com/105562242/193397602-1dca958f-e6f7-4182-90e7-61e021911879.png">

<img width="1537" alt="Screenshot 2022-09-08 at 9 20 30 AM" src="https://user-images.githubusercontent.com/105562242/193397607-a1295e0c-7cc6-48dc-9457-a5dbca7b05a3.png">

<img width="1537" alt="Screenshot 2022-09-08 at 9 20 30 AM" src="https://user-images.githubusercontent.com/105562242/193397621-feabcd17-26e5-4090-ba4a-e2b23012af26.png">

#### STEP 9: Configuring The Components For The Control Plane On The Master/Controller Nodes. 
#### Please note : We have follow on all three master nodes.
------------------------------------------------------------------------

* Creating the Kubernetes configuration directory:```$ sudo mkdir -p /etc/kubernetes/config```
* Downloading the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-apiserver" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-controller-manager" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-scheduler" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl"

```
* Installing the Kubernetes binaries:

```
{
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}

```

* Configuring the Kubernetes API Server:

```
{
sudo mkdir -p /var/lib/kubernetes/

sudo mv ca.pem ca-key.pem master-kubernetes-key.pem master-kubernetes.pem \
service-account-key.pem service-account.pem \
encryption-config.yaml /var/lib/kubernetes/
}

```



<img width="1537" alt="Screenshot 2022-09-08 at 9 23 09 AM" src="https://user-images.githubusercontent.com/105562242/193397858-baa67530-fd0a-4847-ba27-cb23efceaafd.png">

* The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieving the internal IP address for the current compute instance:```$ export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)```
* Creating the kube-apiserver.service systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/master-kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/master-kubernetes-key.pem\\
  --etcd-servers=https://172.31.0.10:2379,https://172.31.0.11:2379,https://172.31.0.12:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/master-kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --runtime-config='api/all=true' \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${INTERNAL_IP}:6443 \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/master-kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```
* Moving the kube-controller-manager kubeconfig into place:```$ sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/```
* Exporting some variables to retrieve the vpc_cidr which will be required for the bind-address flag:

```
export AWS_METADATA="http://169.254.169.254/latest/meta-data"
export EC2_MAC_ADDRESS=$(curl -s $AWS_METADATA/network/interfaces/macs/ | head -n1 | tr -d '/')
export VPC_CIDR=$(curl -s $AWS_METADATA/network/interfaces/macs/$EC2_MAC_ADDRESS/vpc-ipv4-cidr-block/)
export NAME=k8s-cluster-from-ground-up

```

<img width="1293" alt="Screenshot 2022-09-08 at 9 24 23 AM" src="https://user-images.githubusercontent.com/105562242/193397975-dcd9055d-eca4-4c3e-a18a-48ac23c81852.png">

* Creating the kube-controller-manager.service systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=${VPC_CIDR} \\
  --cluster-name=${NAME} \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```
* Moving the kube-scheduler kubeconfig into place:

```
sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
sudo mkdir -p /etc/kubernetes/config

```
* Creating the kube-scheduler.yaml configuration file:

```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF

```

* Creating the kube-scheduler.service systemd unit file:

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

```
* Starting the Controller Services

```
{
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}

```
* Checking the status of the services

```
{
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler
}

```

<img width="1533" alt="Screenshot 2022-09-08 at 9 25 54 AM" src="https://user-images.githubusercontent.com/105562242/193398130-cb244d28-b4a5-4d3f-a9a2-3bc3d7ec0b97.png">

<img width="1535" alt="Screenshot 2022-09-08 at 9 26 16 AM" src="https://user-images.githubusercontent.com/105562242/193398141-e4f7ec76-6e67-412c-a5d0-2f31408e0235.png">

<img width="1537" alt="Screenshot 2022-09-08 at 9 30 22 AM" src="https://user-images.githubusercontent.com/105562242/193398150-fb024951-4f62-4c68-9694-45b32d1a4329.png">

<img width="1542" alt="Screenshot 2022-09-08 at 9 33 49 AM" src="https://user-images.githubusercontent.com/105562242/193398186-ffe38db0-9b9d-428b-a0c0-bf3c3091dadf.png">

#### STEP 10: Testing that Everything is working fine
#### Please note : We have to perform below steps on all 3 master nodes.
-------------------------------------

* To get the cluster details run:```$ kubectl cluster-info --kubeconfig admin.kubeconfig```
* To get the current namespaces:```$ kubectl get namespaces --kubeconfig admin.kubeconfig```

<img width="1457" alt="Screenshot 2022-09-08 at 9 34 39 AM" src="https://user-images.githubusercontent.com/105562242/193398256-461e899e-28a7-4025-b9d7-1566bb69b5c2.png">

<img width="1204" alt="Screenshot 2022-09-08 at 9 35 28 AM" src="https://user-images.githubusercontent.com/105562242/193398265-3ad54c0b-da38-45ba-b97d-6b6b9235add6.png">

<img width="1264" alt="Screenshot 2022-09-08 at 9 36 49 AM" src="https://user-images.githubusercontent.com/105562242/193398270-37550160-8b69-431f-93d3-23641d6e703a.png">

* To reach the Kubernetes API Server publicly:```$ curl --cacert /var/lib/kubernetes/ca.pem https://$INTERNAL_IP:6443/version```
* To get the status of each component:$ kubectl get componentstatuses --kubeconfig admin.kubeconfig

<img width="1047" alt="Screenshot 2022-09-08 at 9 38 26 AM" src="https://user-images.githubusercontent.com/105562242/193398287-52d96a63-1990-4a4b-af91-03199840595f.png">

<img width="1308" alt="Screenshot 2022-09-08 at 9 39 01 AM" src="https://user-images.githubusercontent.com/105562242/193398296-ba01b847-2170-4030-b41f-7f9977e8fbe9.png">

#### STEP 11: Configuring Role Based Access Control
--------------------------------------------------------
* Configuring Role Based Access Control (RBAC) on one of the controller(master) nodes so that the api-server has necessary authorization for for the kubelet.

* Creating the ClusterRole

```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF

```
<img width="1139" alt="Screenshot 2022-09-08 at 9 39 36 AM" src="https://user-images.githubusercontent.com/105562242/193398391-3de58b64-a179-48c0-bb31-883cecf14abc.png">

* Creating the ClusterRoleBinding to bind the kubernetes user with the role created above

```
cat <<EOF | kubectl --kubeconfig admin.kubeconfig  apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

```
<img width="1071" alt="Screenshot 2022-09-08 at 9 39 50 AM" src="https://user-images.githubusercontent.com/105562242/193398420-280b732e-3773-479a-8326-68aa9c91ef88.png">

* The RBAC permissions is configured to allow the Kubernetes API Server to access the Kubelet API on each worker nodes. Creating the system:kube-apiserver-to-kubelet ClusterRole with permissions to access the Kubelet API and perform most common tasks associated with managing pods on the worker nodes:

```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF

```
<img width="1027" alt="Screenshot 2022-09-08 at 9 40 17 AM" src="https://user-images.githubusercontent.com/105562242/193398458-3465646e-f3ae-4d66-84a2-a5bbab696e78.png">

* Binding the system:kube-apiserver-to-kubelet ClusterRole to the kubernetes user so that API server can authenticate successfully to the kubelets on the worker nodes:

```
cat <<EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF

```
<img width="1015" alt="Screenshot 2022-09-08 at 9 40 37 AM" src="https://user-images.githubusercontent.com/105562242/193398490-5f2c4885-58e7-4e23-952c-3977f25118c9.png">


#### STEP 12: Bootstraping components on the worker nodes
---------------------------------------------------------------

* Opening 3 panes and ssh into the 3 worker nodes and setting the synchronize-panes on.

* For Worker node 1:

```
worker_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=${NAME}-worker-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_1_ip}

```
* For Worker node 2

```
worker_2_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=${NAME}-worker-1" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_2_ip}

```

* For Worker node 3

```
worker_3_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=${NAME}-worker-2" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i k8s-cluster-from-ground-up.id_rsa ubuntu@${worker_3_ip}

```

* Installing OS dependencies:

```
{
  sudo apt-get update
  sudo apt-get -y install socat conntrack ipset
}

```

* Disabling Swap:```$ sudo swapoff -a```
* Downloading and installing binaries of runc, cri-ctl and container runtime (Containerd)

```
 wget https://github.com/opencontainers/runc/releases/download/v1.0.0-rc93/runc.amd64 \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.21.0/crictl-v1.21.0-linux-amd64.tar.gz \
  https://github.com/containerd/containerd/releases/download/v1.4.4/containerd-1.4.4-linux-amd64.tar.gz 
  
```
* Configuring the containerd

```
{
  mkdir containerd
  tar -xvf crictl-v1.21.0-linux-amd64.tar.gz
  tar -xvf containerd-1.4.4-linux-amd64.tar.gz -C containerd
  sudo mv runc.amd64 runc
  chmod +x  crictl runc  
  sudo mv crictl runc /usr/local/bin/
  sudo mv containerd/bin/* /bin/
}

```
* Creating containerd directory:```$ sudo mkdir -p /etc/containerd/```

* Inserting the following in it
```
cat << EOF | sudo tee /etc/containerd/config.toml
[plugins]
  [plugins.cri.containerd]
    snapshotter = "overlayfs"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
EOF

```

<img width="809" alt="Screenshot 2022-09-08 at 9 42 41 AM" src="https://user-images.githubusercontent.com/105562242/193398866-e6ceeee0-9573-4a01-bc8c-819e0f202241.png">


* Creating the containerd.service systemd unit file:

```
cat << EOF | sudo tee /etc/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
EOF

```

* Creating directories to configure kubelet, kube-proxy, cni, and a directory to keep the kubernetes root ca file:

```
sudo mkdir -p \
  /var/lib/kubelet \
  /var/lib/kube-proxy \
  /etc/cni/net.d \
  /opt/cni/bin \
  /var/lib/kubernetes \
  /var/run/kubernetes
  
```
* Downloading Container Network Interface(CNI) plugins available from container networking’s GitHub repo:

```
wget -q --show-progress --https-only --timestamping \
  https://github.com/containernetworking/plugins/releases/download/v0.9.1/cni-plugins-linux-amd64-v0.9.1.tgz
  
```
* Installing CNI into /opt/cni/bin/:$ sudo tar -xvf cni-plugins-linux-amd64-v0.9.1.tgz -C /opt/cni/bin/
* Downloading binaries for kubectl, kube-proxy, and kubelet

```
wget -q --show-progress --https-only --timestamping \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubelet
  
```
* Installing the downloaded binaries:

```
{
  chmod +x  kubectl kube-proxy kubelet  
  sudo mv  kubectl kube-proxy kubelet /usr/local/bin/
}

```
#### STEP 13: Configuring The Worker Nodes Components
------------------------------------------------------------

* Configuring the network
* Getting the POD_CIDR that will be used as part of network configuration:

```
POD_CIDR=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^pod-cidr" | cut -d"=" -f2)
echo "${POD_CIDR}"

```
* Configuring the bridge network:

```
cat > 172-20-bridge.conf <<EOF
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF

```
<img width="1044" alt="Screenshot 2022-09-08 at 9 44 36 AM" src="https://user-images.githubusercontent.com/105562242/193399118-8ac30a44-a26a-4ce7-a341-dbf1e0e22274.png">


* Configuring the loopback network:

```
cat > 99-loopback.conf <<EOF
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
EOF

```
* Moving the files to the network configuration directory:```$ sudo mv 172-20-bridge.conf 99-loopback.conf /etc/cni/net.d/```
* Storing the worker’s name in a variable:

```
NAME=k8s-cluster-from-ground-up
WORKER_NAME=${NAME}-$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)
echo "${WORKER_NAME}"

```
* Moving the certificates and kubeconfig file to their respective configuration directories:

```
sudo mv ${WORKER_NAME}-key.pem ${WORKER_NAME}.pem /var/lib/kubelet/
sudo mv ${WORKER_NAME}.kubeconfig /var/lib/kubelet/kubeconfig
sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
sudo mv ca.pem /var/lib/kubernetes/

```
* Creating the kubelet-config.yaml file.

* Ensuring the needed variables exist:

```
NAME=k8s-cluster-from-ground-up
WORKER_NAME=${NAME}-$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)
echo "${WORKER_NAME}"

```
<img width="1058" alt="Screenshot 2022-09-08 at 9 45 57 AM" src="https://user-images.githubusercontent.com/105562242/193399229-f3910ef6-7731-4a60-8c92-a1d68ee1a6ef.png">

* Creating the kubelet-config.yaml file

```
cat <<EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
resolvConf: "/etc/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${WORKER_NAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${WORKER_NAME}-key.pem"
EOF

```
<img width="1029" alt="Screenshot 2022-09-08 at 9 46 12 AM" src="https://user-images.githubusercontent.com/105562242/193399276-72532736-9a0f-492f-9129-ce18ce2cb23e.png">

* Configuring the kubelet systemd service:

```
cat <<EOF | sudo tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service
[Service]
ExecStart=/usr/local/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --cluster-domain=cluster.local \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF

```


<img width="1291" alt="Screenshot 2022-09-08 at 9 47 13 AM" src="https://user-images.githubusercontent.com/105562242/193399310-808ed03d-8051-4e8f-967d-ef7baa68a2a4.png">


* Creating the kube-proxy.yaml file:

```
cat <<EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "172.31.0.0/16"
EOF

```

<img width="1291" alt="Screenshot 2022-09-08 at 9 47 13 AM" src="https://user-images.githubusercontent.com/105562242/193399348-b2826ff1-4911-443e-aeb2-4c7171aadfb0.png">


