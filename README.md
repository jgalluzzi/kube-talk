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

Setup AWS Access Key
I'd suggest creating a separate user for this that isn't your root user. Enable that user with the correct role and create an access key for it.

Authenticate AWS CLI

`aws configure`

Setup VPC/Subnet/Security Groups

`curl -o vpc.yaml https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-06-10/amazon-eks-vpc-sample.yaml`

```
aws cloudformation create-stack \
  --stack-name eks-vpc \
  --template-body file://vpc.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```