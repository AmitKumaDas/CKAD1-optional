# CKAD1-optional



Kubernetes Essentials
=============================================================
Lab 1: Kubernetes Operations(kops) on AWS
=============================================================

##Task 1: Launching an EC2 Instance
#create an EC2 Instance
Ubuntu Server22.04 LTS (HVM), SSD Volume type
t2.micro

##Task 2: Create an IAM role
#create an IAM role for your instance as kops-admin-role with AdministratorAccess policy

##Now, attach the IAM kops-admin-role to your EC2 instance kops by following these steps:
1. First select EC2 instance kops.
2. Next click on Action > Security > Modify IAM role
3. Search   for kops-admin role, select it and click Update IAM role

##Task 3: Setting up a Kubernetes Cluster

sudo hostnamectl set-hostname kops

bash

wget https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/kops-v1.25.sh

. ./kops-v1.25.sh

kops validate cluster

=============================================================
Lab 2: Services in Kubernetes
=============================================================
---------------------------------------------------------------
# Task 1 Create a pod using below yaml
---------------------------------------------------------------

vi httpd-pod.yaml


apiVersion: v1
kind: Pod
metadata:
  name: httpd-pod
  labels:
    env: prod 
    type: front-end
    app: httpd-ws
spec:
  containers:
  - name: httpd-container
    image: httpd
    ports:
       - containerPort: 80


# Apply the pod definition yaml

kubectl create -f httpd-pod.yaml


# Check the newly created Pod

kubectl get pods

kubectl get pods -o wide

# Describe Pod using below command

kubectl describe pod httpd-pod


----------------------------------------------------------------------
#Task 2  Setup ClusterIP service
----------------------------------------------------------------------

# Create  a ClusterIP service using below YAML

vi httpd-svc.yaml


apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP


 #Apply above definition using below to create a ClusterIP service

kubectl apply -f httpd-svc.yaml

 # Describe the service and verify it has populated the endpoints with IP address matching Pod label

kubectl get svc

kubectl describe svc httpd-svc

  # Get EndPoint of the service

kubectl get ep  

 # Get External IPs of the machines in the cluster using below.

kubectl get nodes -owide | awk '{print $7}'

  #SSH to one of the machines and rerun the command in the previous task

ssh -t ubuntu@<Node_IP> curl <Cluster_IP>:<Service_Port>


------------------------------------------------------------------------------
#Task 3  Setup NodePort Service
------------------------------------------------------------------------------

# Modify the service created in the previous task to type NodePort

vi httpd-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: NodePort


 #  Apply the changes using below command

kubectl apply -f httpd-svc.yaml


 # View details of the modified service

kubectl describe svc httpd-svc

 # Validate connectivity using External IP on NodePort using below or via browser

curl <EXTERNAL-IP>:NodePort

 # Get External IPs of the machines in the cluster. SSH to one of the machines and rerun the command in the previous task

kubectl get nodes -o wide | awk '{print $7}'

ssh -t ubuntu@<Node_IP> curl <Cluster_IP>:<Service_Port>
------------------------------------------------------------------------------------
#Task 4  Setup LoadBalancer Service
------------------------------------------------------------------------------------

 # Modify the service created in the previous task to type LoadBalancer 

vi httpd-svc.yaml

apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  selector:
    env: prod
    type: front-end
    app: httpd-ws
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer


 #  Apply the changes using below command

kubectl apply -f httpd-svc.yaml


 # Verify that a new service of type LoadBalancer has been created

kubectl get svc

kubectl describe svc httpd-svc

 
 # Access the LoadBalancer on the kops instance or via browser

curl <LoadBalancer_DNS>



-------------------------------------------------------------------------------
#Task 5 Delete and recreate httpd Pod
-------------------------------------------------------------------------------
# Delete the existing httpd-pod using below

kubectl delete -f httpd-pod.yaml

# View the service details and notice that the Endpoints field is empty

kubectl describe svc httpd-svc

# Recreate the httpd Pod and view service details Verify that the endpoints is updated with new Pod IP

kubectl apply -f httpd-pod.yaml

kubectl describe svc httpd-svc



--------------------------------------------------------------------------------
#Task 6 Cleanup the resources using below command
----------------------------------------------------------------------------------

kubectl delete -f httpd-pod.yaml
kubectl delete -f httpd-svc.yaml

=============================================================
Lab 3: Deployment
=============================================================

----------------------------------------------------------------------
Task 1: Write a Deployment yaml and Apply it
----------------------------------------------------------------------
#Create a dep-nginx.yaml using content given below

vi dep-nginx.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dep
  labels:
    app: nginx-dep
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx-ctr
        image: nginx:1.12.2
        ports:
        - containerPort: 80



#Apply the Deployment yaml created in the previous step

kubectl apply -f dep-nginx.yaml


#View the objects created by Kubernetes, Deployment and Replica Set 

kubectl get deployments
kubectl get rs


#Access one of the Pods and view nginx version

kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash


# nginx -v
# exit


-------------------------------------------------------------------------
 Task 2: Update the Deployment with a Newer Image
-------------------------------------------------------------------------

#Update the nginx image in Pod using below

kubectl set image deployment/nginx-dep nginx-ctr=nginx:1.11


#Describe the deployment and see that the old pods are replaced with newer ones

kubectl describe deployments


#Access one of the Pods and view nginx version

kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash

# nginx -v
# exit



-----------------------------------------------------------------------------
Task 3: Rollback of Deployment 
-----------------------------------------------------------------------------

#View the history of Deployments

kubectl rollout history deployment/nginx-dep


#Rollback the Deployment done in the previous task

kubectl rollout undo deployment/nginx-dep --to-revision=1

kubectl get rs


#Access one of the Pods and view nginx version

kubectl get pods
kubectl exec -it <pod_name> -- /bin/bash

# nginx -v
# exit



------------------------------------------------------------------------------
Task 4: Scaling of Deployments
------------------------------------------------------------------------------

#View the number of Pod replicas created by the Deployment

kubectl get deployments
kubectl get pods


#Scale up the deployment to have 8 Pod replicas

kubectl scale deployment nginx-dep --replicas=8



#Check the Pods and deployment to and verify that the number of Pod replicas are 8

kubectl get deployments
kubectl get pods


#Scale down the deployments to 2 Pod replicas

kubectl scale deployment nginx-dep --replicas=2


#Check the Pods and deployment to and verify that the number of Pod replicas are down to 2

kubectl get deployments
kubectl get pods


-----------------------------------------------------------------------------
#Task 6 Cleanup the resources using below command
-----------------------------------------------------------------------------
kubectl delete -f dep-nginx.yaml

=============================================================
Lab 4: DaemonSet in Kubernetes
=============================================================

#Create a DaemonSet using below yaml

vi ds-pod.yaml
----------------------------------------------
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-ds
  labels:
    app: fluent-ds
spec:
  selector:
    matchLabels:
      app: fluentd-app
  template:
    metadata:
       labels:
           app: fluentd-app
    spec:
      containers:
      - name: fluentd-ctr
        image: fluent/fluentd:v1.16

-----------------------------------------------

#Apply the yaml definition to create a fluent-ds DaemonSet

kubectl apply -f ds-pod.yaml

#Check the available daemonsets in kubernetes cluster

kubectl get ds fluent-ds


#Verify that pods for fluent are created one for each node using DaemonSet

kubectl get pods -o wide

#Cleanup the DaemeonSet using below 

kubectl delete -f ds-pod.yaml

#Verify the pods to find (all) the fluentd pods being deleted from each of the nodes

kubectl get pods


=============================================================
Lab 5: Persistent Volume in Kubernetes
=============================================================

----------------------------------------------------------------------------
# Task 1  Get Node Label and Create Custom Index.html on Node
----------------------------------------------------------------------------

# View worker nodes and their labels

kubectl get nodes --show-labels | grep kubernetes.io/hostname


# make a note of the kubernetes.io/hostname label of one of the nodes

kubectl get node -o wide ( run this to get the <node_public_IP )

# ssh to one of the nodes using below

 ssh -t ubuntu@<node_public_IP> 



# Switch to root and run the following commands. A directory with custom index.html is created for PersistentVolume mount 

sudo su
mkdir /pvdir
echo Hello World! > /pvdir/index.html


--------------------------------------------------------------------------------
# Task 2 - Create a Local Persistent Volume
--------------------------------------------------------------------------------

vi pv-volume.yaml


kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-volume
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/pvdir"



kubectl apply -f pv-volume.yaml

kubectl get pv

kubectl describe pv pv-volume

------------------------------------------------------------------------------------
# Task 3  - Create a PV Claim
------------------------------------------------------------------------------------
vi pv-claim.yaml


kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pv-claim
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi

kubectl apply -f pv-claim.yaml
----------------------------------------------------------------------------------------
# Task 4  - Create nginx Pod with NodeSelector
----------------------------------------------------------------------------------------
vi pv-pod.yaml


kind: Pod
apiVersion: v1
metadata:
  name: pv-pod
spec:
  volumes:
    - name: pv-storage
      persistentVolumeClaim:
        claimName: pv-claim
  containers:
     - name: pv-container
       image: nginx
       ports:
          - containerPort: 80
            name: "http-server"
       volumeMounts:
          - mountPath: "/usr/share/nginx/html"
            name: pv-storage
  nodeSelector:
    kubernetes.io/hostname: ip-172-20-33-138.ap-south-1.compute.internal

# Apply the Pod yaml created in the previous step

kubectl apply -f pv-pod.yaml

# View Pod details and see that is created on the required node

kubectl get pods -o wide

# Access shell on a container running in your Pod

kubectl exec -it pv-pod -- /bin/bash

# Run the following commands in the container to verify PersistentVolume

 apt-get update
 apt-get install curl -y
 curl localhost
 exit

# delete the resources created in this lab.
kubectl delete -f pv-pod.yaml
kubectl delete -f pv-claim.yaml
kubectl delete -f pv-volume.yaml
=============================================================
Lab 6: StatefulSet Implementation
=============================================================

----------------------------------------------------------------------
#Task1 - Create Stateful Set
----------------------------------------------------------------------

#Download file with yaml defintion for an nginx Stateful Set 

wget https://s3.ap-south-1.amazonaws.com/files.cloudthat.training/devops/kubernetes-essentials/nginx-sts.yaml


#Create a Stateful Set by applying the yaml

kubectl apply -f nginx-sts.yaml


#Create a headless service

kubectl get service nginx-svc


#Validate the stateful set creation

kubectl get statefulset nginx-sts

kubectl delete pod --all

kubectl get pod

kubectl delete pod web-0

kubectl get pod


----------------------------------------------------------------------------
# Task 2  - Scaling a Stateful Set
----------------------------------------------------------------------------

# Scale the Stateful Set to 5 replicas using below.

kubectl scale sts nginx-sts --replicas=5


# Verify the pods getting created in ordinal way

kubectl get pods -w -l app=nginx-sts

# Verify the PV Claim getting created in ordinal fashion
kubectl get pvc -l app=nginx-sts

# Edit the stateful Set yam and reduce replicas to 3 
kubectl edit sts nginx-sts

# Notice that the controller deletes the pods one at a time. It waits for one to completely shut down before going to next

kubectl get pods -w -l app=nginx-sts


# Verify statefulSet’s PersistentVolumeClaims and verify that are not deleted on scaling down. 

kubectl get pvc -l app=nginx-sts


-------------------------------------------------------------------------------
# Task 4 - Delete a Stateful Set - Cleanup
--------------------------------------------------------------------------------

kubectl delete -f nginx-sts.yaml

# List all the PV and PVC’s that has been allocated to Statefulset pods and delete them as below.

kubectl get pvc

kubectl delete pvc --all



*****************Delete Cluster*********************************
. ./delete-kops.sh   

or

kops get cluster
kops delete cluster --name <cluster-naame> --state s3://<cluster-naame> --yes
