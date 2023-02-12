Prerequisite: Install required client tools. You will need to install the following tools:
1.	awscli – to be used to manage your AWS services
2.	kubectl – this command line utility will be your main control tool to manage your K8s cluster.
3.	cfssl – a Cloudflare open source toolkit for everything TLS/SSL 
4.	cfssljson – a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.
	
### Install and configure AWSCLI on virtual machine (Linux)
1.	Configure AWSCLI to access all AWS services required. You need a user with programmatic access keys configured in AWS Identity and Access Management (IAM)
2.	Generate your access keys and save them safely.
3.	Install AWSCLI on your VM. Click on this link to install AWSCLI.(https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
4.	Next configure AWSCLI on your VM with the following steps:
```
type `aws configure` 
enter the access key ID you save
enter the secret access key
enter your desired region
enter `json` for your default output format.
```
5. Type `aws ec2 describe-vpcs` to test AWSCLI.
6. It should give you a list of available VPCs in the region you selected of the account.

### Install Kubectl on Linux machine
Kubectl is a tool that allows you easily interact with Kubernetes to deploy applications, inspect and manage cluster resources, view logs and perform many more administrative operations instead of using **curl** everytime you need to send commands to the Kubernetes cluster API. To install Kubectl:
1. Download the binary `wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl`
2. Make it executable `chmod +x kubectl`
3. Move to the Bin directory `sudo mv kubectl /usr/local/bin/`
4. Verify that kubectl version 1.21.0 or higher is installed: `kubectl version --client`

### Install CFSSL and CFSSLJSON
1. Download the binary: 
```
wget -q --show-progress --https-only --timestamping \
https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssl \
https://storage.googleapis.com/kubernetes-the-hard-way/cfssl/1.4.1/linux/cfssljson
```
2. Make it executeable: `chmod +x cfssl cfssljson`
3. Move it to binary directory: `sudo mv cfssl cfssljson /usr/local/bin/`

##PROVISION AWS CLOUD RESOURCES FOR KUBERNETES CLUSTER

Our Kubernetes cluster will be needing some AWS resouces to run the **Control plane** and the **Worker nodes** This step can be easily automated with Terraform but for the purposes of learning, we will be doing it manually with AWSCLI. SOme of the resources we will be provisioning includes: VPC, SUbnet, EC2, Security groups, Loadbalancer etc. Lets begin.

### Configure Network Infrastructure
**Virtual Private Cloud – VPC**
1. Create a directory named **k8s-cluster-from-ground-up**: `mkdir k8s-cluster-from-ground-up`
2. Create a VPC and store the ID as a variable:
```
VPC_ID=$(aws ec2 create-vpc \
--cidr-block 172.31.0.0/16 \
--output text --query 'Vpc.VpcId'
)
```
3. Tag the VPC so that it is named k8s-cluster-from-ground-up:
```
NAME=k8s-cluster-from-ground-up

aws ec2 create-tags \
  --resources ${VPC_ID} \
  --tags Key=Name,Value=${NAME}
```
**Domain Name System – DNS**

1. Enable DNS support for your VPC:
```
aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-support '{"Value": true}'
```
2. Enable DNS support for hostnames:
```
aws ec2 modify-vpc-attribute \
--vpc-id ${VPC_ID} \
--enable-dns-hostnames '{"Value": true}'
```
**Set AWS Region**

1. Set the required region: `AWS_REGION=us-east-2`
Subnet

**Create the Subnet and tag it:**
```
SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id ${VPC_ID} \
  --cidr-block 172.31.0.0/24 \
  --output text --query 'Subnet.SubnetId')
aws ec2 create-tags \
  --resources ${SUBNET_ID} \
  --tags Key=Name,Value=${NAME}
Internet Gateway – IGW
```

**Create the Internet Gateway and attach it to the VPC:**
```
INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway \
  --output text --query 'InternetGateway.InternetGatewayId')
aws ec2 create-tags \
  --resources ${INTERNET_GATEWAY_ID} \
  --tags Key=Name,Value=${NAME}
aws ec2 attach-internet-gateway \
  --internet-gateway-id ${INTERNET_GATEWAY_ID} \
  --vpc-id ${VPC_ID}
```
**Route tables**

1. Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway:
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
Your output should look like this:
```
Output:

{
    "AssociationId": "rtbassoc-07a8877e92504def7",
    "AssociationState": {
        "State": "associated"
    }
}
{
    "Return": true
}
```
**Security Groups**
1. To configure security groups, create seecurity group and tag it: 
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
```
2. Configure inbound traffic for all communication within the subnet:
```
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
**Network Load Balancer**
1. Create a network Load balancer:
```
LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer \
--name ${NAME} \
--subnets ${SUBNET_ID} \
--scheme internet-facing \
--type network \
--output text --query 'LoadBalancers[].LoadBalancerArn')
```
**Tagret Group**
1. Create a target group: (For now it will be unhealthy because there are no real targets yet.)
```
TARGET_GROUP_ARN=$(aws elbv2 create-target-group \
  --name ${NAME} \
  --protocol TCP \
  --port 6443 \
  --vpc-id ${VPC_ID} \
  --target-type ip \
  --output text --query 'TargetGroups[].TargetGroupArn')
```
**Register targets**
Just like above, no real targets. You will just put the IP addresses so that, when the nodes become available, they will be used as targets
```
aws elbv2 register-targets \
  --target-group-arn ${TARGET_GROUP_ARN} \
  --targets Id=172.31.0.1{0,1,2}
```
**Create Listener**
1. Create a listener to listen for requests and forward to the target nodes on TCP port 6443
```
aws elbv2 create-listener \
--load-balancer-arn ${LOAD_BALANCER_ARN} \
--protocol TCP \
--port 6443 \
--default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} \
--output text --query 'Listeners[].ListenerArn'
```
**K8s Public Address**
1. Get the Kubernetes Public address
```
KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers \
--load-balancer-arns ${LOAD_BALANCER_ARN} \
--output text --query 'LoadBalancers[].DNSName')
```
