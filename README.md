# aws-k8s-mnist-inference

## Setup
Ensure your ~/.aws configuration is set up with aws_key and aws_id params

## Requirements

- ekstl
- aws cli
- kubectl cli


## Create cluster using the eks cli tool:

```
eksctl create cluster \
--name cloud-ml-caraline-eks-cluster \
--version 1.26 \
--region us-east-1 \
--nodegroup-name linux-nodes-eks-cloud-ml \
--node-type t2.large \
--nodes 2
```

wait 5-10 minutes you will see the nodes as active on the UI

check that they are up from the cli with `kubectl get nodes`

## ROLES
Nodes require permissions to use the EBS volumes. 
Ensure that the EKS worker nodes have the necessary IAM permissions to create and manage EBS volumes. 
The required permissions include actions like ec2:CreateVolume, ec2:AttachVolume, ec2:DescribeVolumes, and others. 
This can be done from the aws UI.

We have the nodes create and manage the volume as this is done through kubectl, so there is an extra step.

Add permissions for the worker nodes to use the EBS CSI driver.

- AmazonEC2FullAccess
- AmazonEC2ContainerRegistryReadOnly
- AmazonEKS_CNI_Policy
- AmazonEKSWorkerNodePolicy
- AmazonSSMManagedInstanceCore
- AmazonEBSCSIDriverPolicy




## Important, Install EBS Driver to the cluster

Verify that the Amazon EBS CSI driver is installed and running in your cluster:

`kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver`

This command should return a list of pods related to the Amazon EBS CSI driver. Ensure that they are running without issues.


### If the driver is not found

One way to install the Amazon EBS CSI driver on your EKS cluster:

```
git clone https://github.com/kubernetes-sigs/aws-ebs-csi-driver.git
cd aws-ebs-csi-driver
git checkout release-1.2
kubectl apply -f deploy/kubernetes/base/csidriver.yaml
```

Install the CSI driver itself:

`kubectl kustomize deploy/kubernetes/overlays/stable | kubectl apply -f -`


#### ERROR ON ONE FILE
Edit deploy/kubernetes/base/pod_disruption_budgets.yaml. Change the first line which gives the apiVersion from policy/v1beta1 to policy/v1. 


Write and Quit.


Then, additionally apply this single file to the cluster

`kubectl apply -f pod_disruption_budgets.yaml`

Verify they are running on the cluster:

`kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver`



## Finally

Apply the pvc and application YAML files:

```
kubectl apply -f aws-pvc.yaml
kubectl apply -f mnist.yaml
```


Get the public endpoint for the load balancer service:

`kubectl get svc my-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'`


Done!
