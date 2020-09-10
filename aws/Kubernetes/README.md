# Setup Kuberntes using terraform script
### Prerequisite
#### terraform version 0.11.14
Download terraform using this link 
https://releases.hashicorp.com/terraform/

### Running Terraform 
```
terraform init
terraform apply
terraform plan
```


Amazon EKS Clusters
An Amazon EKS cluster consists of two primary components:

• Amazon EKS control plane
• Amazon EKS worker nodes registered with the control plane
Worker Nodes
Worker machines which are part of the Kubernetes cluster are called Worker nodes. Amazon EKS worker nodes are created in your AWS account, and they establish a connection to your cluster’s control plane instance running in AWS managed the account, via endpoint of the cluster API server and a certificate file created for each cluster. Amazon EKS worker nodes are registered with the control plane
Cluster VPC Considerations
To launch & configure an Amazon EKS cluster, you need to specify the Amazon VPC subnets required for your cluster to be hosted in. To maintain high availability, Amazon EKS requires the subnets from at least two Availability Zones
Cluster Security Group Considerations:
The security group of worker nodes and the security group that handles the communication of the control plane with the worker nodes have been constructed in a way to avoid communication via the privileged ports in the worker nodes. In case your applications require additional inbound or outbound access from the control plane or worker nodes, you should include these rules to the security groups associated with your cluster.
Let's start working on it!!
Prerequisite:
AWS Account Access
Terraform v0.11.14
Download GitHub clone file to create VPC, EKS setup and ec2 access server using terraform script
git clone https://github.com/tensult/terraform.git
This terraform script will create IAM roles, VPC, EKS, and worker node, it will also create kubernetes server to configure kubectl on EKS.
Note: This terraform will also take workstation IP, so you don't have to create a Kubernetes server separately.
cd aws/Kubernetes
terraform init
terraform plan
terraform apply
Note: Here you can also change the address range for VPC and subnet in an array of a number of availability zone in the regions.
eg VPC CIDR: 10.0.0.0/16
eg subnets [“10.0.0.0/20”, “10.0.16.0/20”, “10.0.32.0/20”]
After launching terraform
Install and configure kubectl for EKS on Linux machine(Amazon Linux)
Download and install kubectl
curl -o kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/kubectl
2. Verify the downloaded binary with the SHA-256 sum for your binary.
curl -o kubectl.sha256 https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/kubectl.sha256
3. Check the SHA-256 sum for your downloaded binary.
openssl sha1 -sha256 kubectl
4. Apply to execute permissions to the binary.
chmod +x ./kubectl
5. Copy the binary to a folder in your PATH
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$HOME/bin:$PATH
6. Add the $HOME/bin path to your shell initialization file
echo ‘export PATH=$HOME/bin:$PATH’ >> ~/.bashrc
Installing aws-iam-authenticator
Download the Amazon EKS-vended aws-iam-authenticator binary from Amazon S3
curl -o aws-iam-authenticator https://amazon-eks.s3-us-west-2.amazonaws.com/1.13.7/2019-06-11/bin/linux/amd64/aws-iam-authenticator
2. Apply execute permissions to the binary
chmod +x ./aws-iam-authenticator
3. Copy the binary to a folder in your $PATH
mkdir -p $HOME/bin && cp ./aws-iam-authenticator $HOME/bin/aws-iam-authenticator && export PATH=$HOME/bin:$PATH
4. Add $HOME/bin to your PATH environment variable
echo ‘export PATH=$HOME/bin:$PATH’ >> ~/.bashrc
Configure kubectl for Amazon EKS
Before creating kubeconfig file use aws configure. Here you can also create a profile and add it to kubeconfig file
aws eks --region region update-kubeconfig -- name cluster_name -- profile profile-name
Test your configuration
kubectl get svc
Output:
NAME           TYPE      CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   1m
To enable worker nodes to join your cluster
Download, edit, and apply the AWS IAM Authenticator configuration map
curl -o aws-auth-cm.yaml https://amazon-eks.s3-us-west-2.amazonaws.com/cloudformation/2019-02-11/aws-auth-cm.yaml
open the aws-auth-cm.yaml file any editor. Replace the ARN of instance role (not instance profile)
Note: do not change any other line in this file
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: <ARN of instance role (not instance profile)>
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
Apply the configuration. This command may take a few minutes to finish
kubectl apply -f aws-auth-cm.yaml
Watch the status of your nodes and wait for them to reach the Ready status
kubectl get nodes — watch
Deploy the Nginx container to the Cluster
Kubernetes Cluster is now ready, it’s time to deploy the Nginx container.
On the Master Node, run the following command to create an Nginx deployment:
kubectl create deployment nginx --image=nginx
Output:
deployment.apps/nginx created
You can list out the deployments with the following command:
kubectl get pods
Output :
NAME   READY   UP-TO-DATE AVAILABLE   
nginx   0/1     1            0          
After, we will need to make the Nginx container available to the network with this command:
kubectl create service nodeport nginx --tcp=80:80
Now, list out all the services by running the following command:
kubectl get svc
You should see the Nginx service with assigned port 30784:
NAME        TYPE         CLUSTER-IP    EXTERNAL-IP    PORT(S)      
nginx       NodePort    10.102.166.47   <none>        80:30784/TCP  
kubernetes  ClusterIP   10.100.0.1      <none>        443/TCP
Finally, just open your web browser and type the URL http://<workernode-private-ip>:30784 (Worker Node IP: The port number from your output). You should see the default Nginx Welcome page
Image for post
