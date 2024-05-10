# Step - 1 : Create EKS Management Host in AWS #

1) Launch new Ubuntu VM using AWS Ec2 ( t2.micro )	  
2) Connect to machine and install kubectl using below commands  
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```
3) Install AWS CLI latest version using below commands 
```
sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

4) Install eksctl using below commands
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
# Step - 2 : Create IAM role & attach to EKS Management Host #

1) Create New Role using IAM service ( Select Usecase - ec2 ) 	
2) Add below permissions for the role <br/>
	- IAM - fullaccess <br/>
	- VPC - fullaccess <br/>
	- EC2 - fullaccess  <br/>
	- CloudFomration - fullaccess  <br/>
	- Administrator - acces <br/>
		
3) Enter Role Name (eksroleec2) 
4) Attach created role to EKS Management Host (Select EC2 => Click on Security => Modify IAM Role => attach IAM role we have created) 

# Step - 3 : Create EKS Cluster using eksctl # 
**Syntax:** 

eksctl create cluster --name cluster-name  \
--region region-name \
--node-type instance-type \
--nodes-min 2 \
--nodes-max 2 \ 
--zones <AZ-1>,<AZ-2>

## N. Virgina: <br/>
`
eksctl create cluster --name dataprocluster --region us-east-1 --node-type t2.medium  --zones us-east-1a,us-east-1b
`	
## Mumbai: <br/>
`
eksctl create cluster --name dataprocluster --region ap-south-1 --node-type t2.medium  --zones ap-south-1a,ap-south-1b
`

## Note: Cluster creation will take 5 to 10 mins of time (we have to wait). After cluster created we can check nodes using below command.

`
 kubectl get nodes  
`

## Note: We should be able to see EKS cluster nodes here.**

# We are done with our Setup #
	
## Step - 4 : After your practise, delete Cluster and other resources we have used in AWS Cloud to avoid billing ##

```
eksctl delete cluster --name dataprocluster --region ap-south-1
```



# see if you can get the nodes you created
kubectl get nodes

# Install nano editor in cloudshell. We will need this in the next task
sudo yum install nano -y



Task 4: Create a new POD in EKS for the 2048 game
================================================

# clean up the files in cloudshell (Optional)
rm *.* 

# create the config file in YAML to deploy 2048 game pod into the cluster
nano 2048-pod.yaml

### code starts ###
apiVersion: v1
kind: Pod
metadata:
   name: 2048-pod
   labels:
      app: 2048-ws
spec:
   containers:
   - name: 2048-container
     image: blackicebird/2048
     ports:
       - containerPort: 80

### code ends ###


# apply the config file to create the pod
kubectl apply -f 2048-pod.yaml
#pod/2048-pod created

# view the newly created pod
kubectl get pods


Task 5: Setup Load Balancer Service
===================================
nano mygame-svc.yaml  

### code starts ###

apiVersion: v1
kind: Service
metadata:
   name: mygame-svc
spec:
   selector:
      app: 2048-ws
   ports:
   - protocol: TCP
     port: 80
     targetPort: 80
   type: LoadBalancer

### code ends ###

# apply the config file
kubectl apply -f mygame-svc.yaml

# view details of the modified service
kubectl describe svc mygame-svc

# Access the LoadBalancer Ingress on the kops instance
curl <LoadBalancer_Ingress>:<Port_number>
or
curl a06aa56b81f5741268daca84dca6b4f8-694631959.us-east-1.elb.amazonaws.com:80
(try this from your laptop, not from your cloudshell)

# Go to EC2 console. get the DNS name of ELB and paste the DNS into address bar of the browser
# It will show the 2048 game. You can play. (need to wait for 2-3 minutes for the 
# setup to be complete)


Task 3: Cleanup
---------------
# Clean up all the resources created in the task
kubectl get pods
kubectl delete -f 2048-pod.yaml

kubectl get services
kubectl delete -f mygame-svc.yaml



####################################################################

