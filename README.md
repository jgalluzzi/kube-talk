# kube-talk
Reference files for Kubernetes Talk

### Slides
https://docs.google.com/presentation/d/1u_qKfUZyHMjLwDkuOHSICypGJOAv4WtMVEnLd17xuJM/

### Virtualization Providers
Needed for Minikube to start/deploy

Docker - https://www.docker.com/products/docker-desktop/
QEMU - https://www.qemu.org/download/#macos
VirtualBox - https://www.virtualbox.org/wiki/Downloads

### Minikube

Download/Install
https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Farm64%2Fstable%2Fbinary+download



## AWS Configuration

### Install CLI
Windows
winget install -e --id Amazon.AWSCLI

MacOS
brew install awscli

### Install helm

brew install helm

### Setup AWS Access Key
I'd suggest creating a separate user for this that isn't your root user. Enable that user with the correct role and create an access key for it.

### Authenticate AWS CLI

`aws configure`

### Create IAM Resources. Reference the file eks-trust-policy.json from git
```
aws iam create-role \
  --role-name EKSClusterRole \
  --assume-role-policy-document file://eks-trust-policy.json
```

```
aws iam attach-role-policy \
  --role-name EKSClusterRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```
```
aws iam create-role \
  --role-name eksNodeRole \
  --assume-role-policy-document file://trust-ec2.json

aws iam attach-role-policy \
  --role-name eksNodeRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy

aws iam attach-role-policy \
  --role-name eksNodeRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

aws iam attach-role-policy \
  --role-name eksNodeRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

aws iam attach-role-policy \
  --role-name eksNodeRole \
  --policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy

```

### Setup VPC/Subnet/Security Groups

```
curl -o vpc.yaml https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-06-10/amazon-eks-vpc-sample.yaml
```

```
aws cloudformation create-stack \
  --stack-name eks-vpc \
  --template-body file://vpc.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

Get stack ouputs to use in Cluster creation
```
aws cloudformation describe-stacks \
  --stack-name eks-vpc \
  --query "Stacks[0].Outputs" \
  --region us-west-1
```
Save Subnet IDs, Security Groups, and VPC for next step

### Create Cluster
Create your EKS cluster. Use your AWS account id for <ACCOUNT_ID>. Ensure your subnets, security groups, and region is correct. 

```
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::<ACCOUNT_ID>:role/EKSClusterRole \
  --resources-vpc-config subnetIds=<SUBNET1_ID>,<SUBNET2_ID>,securityGroupIds=<SG_ID> \
  --region us-west-1
```


#### Node group
Create Node group in cluster
```
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name my-node-group \
  --node-role arn:aws:iam::<ACCOUNT_ID>:role/eksNodeRole \
  --subnets subnet-abc123 subnet-def456 \
  --scaling-config minSize=1,maxSize=2,desiredSize=1 \
  --disk-size 20 \
  --instance-types t3.medium \
  --region us-west-1
```

### Enable kubectl access

```
aws eks update-kubeconfig --name my-cluster --region us-west-1
```

### Deploy Honeypot (helm)
Install the helm chart to deploy the honeypot service/deployment

```
helm install honeypot ./honeypot -f honeypot/values.yaml
```


