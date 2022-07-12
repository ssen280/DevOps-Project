
#### STEP 1 – PREPARE NFS SERVER

##### We will spin up a new EC2 instance with RHEL Linux 8 Operating System.
##### Based on our LVM experience from Project 6, We will configure LVM on the Server. Instead of formating the disks as ext4 you will have to format them as xfs
##### We will ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs.

##### We will create mount points on /mnt directory for the logical volumes as follow:
##### Mount lv-apps on /mnt/apps – To be used by webservers
##### Mount lv-logs on /mnt/logs – To be used by webserver logs
##### Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

##### Install NFS server, configure it to start on reboot and make sure it is u and running
```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
<img width="770" alt="Screenshot 2022-07-10 at 7 54 09 PM" src="https://user-images.githubusercontent.com/105562242/178576064-e9de2c6a-2be7-4971-9eec-0e55335b6e91.png">

<img width="1477" alt="Screenshot 2022-07-10 at 7 55 44 PM" src="https://user-images.githubusercontent.com/105562242/178576106-2fd615ae-bd0a-4e1c-b64c-3717e3a02d25.png">

<img width="929" alt="Screenshot 2022-07-10 at 7 58 33 PM" src="https://user-images.githubusercontent.com/105562242/178576150-841b54e7-804f-4dd6-b36e-1f700d85a95e.png">

<img width="818" alt="Screenshot 2022-07-10 at 8 03 24 PM" src="https://user-images.githubusercontent.com/105562242/178576277-51db0dd5-e492-48f3-a63a-804dd8dba6d2.png">

<img width="855" alt="Screenshot 2022-07-10 at 8 04 44 PM" src="https://user-images.githubusercontent.com/105562242/178576317-1d7ac143-130f-4544-9059-451f635ee1b8.png">

<img width="1424" alt="Screenshot 2022-07-10 at 8 08 58 PM" src="https://user-images.githubusercontent.com/105562242/178576373-a4380db8-f970-47c2-ad6b-87804d8b5edb.png">

<img width="1020" alt="Screenshot 2022-07-10 at 8 09 22 PM" src="https://user-images.githubusercontent.com/105562242/178576432-6f1a50c2-4014-4f09-86bf-52889a603a0c.png">

<img width="1338" alt="Screenshot 2022-07-10 at 8 12 26 PM" src="https://user-images.githubusercontent.com/105562242/178576506-28fd5cac-d67e-4d7e-ad13-9ffa5042377e.png">


<img width="1369" alt="Screenshot 2022-07-11 at 8 09 38 PM" src="https://user-images.githubusercontent.com/105562242/178576628-199a3782-92d6-4bfc-9614-4b58bb2e8f1e.png">

<img width="1279" alt="Screenshot 2022-07-11 at 8 15 08 PM" src="https://user-images.githubusercontent.com/105562242/178576683-57897d06-46f0-45a8-bf6d-59638d21e715.png">

<img width="1727" alt="Screenshot 2022-07-11 at 8 24 04 PM" src="https://user-images.githubusercontent.com/105562242/178577271-50acb53f-937f-4e3d-bf29-1fe7a4e01d70.png">

<img width="884" alt="Screenshot 2022-07-13 at 1 01 42 AM" src="https://user-images.githubusercontent.com/105562242/178578580-98f94923-9f0b-4e9b-a3d3-f5aa04534b49.png">

