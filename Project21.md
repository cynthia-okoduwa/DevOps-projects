Prerequisite: Install required client tools. You will need to install the following tools:
1.	awscli – to be used to manage your AWS services
2.	kubectl – this command line utility will be your main control tool to manage your K8s cluster.
3.	cfssl – a Cloudflare open source toolkit for everything TLS/SSL 
4.	cfssljson – a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.

Install and configure AWSCLI on virtual machine (Linux)
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
