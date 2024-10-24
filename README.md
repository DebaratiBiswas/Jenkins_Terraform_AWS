# Jenkins_Terraform_AWS
Repo to deploy infra to AWS using terraform. Building and deployment of a Java Web Application in AWS EKS using Ansible. 

1. Created AWS account  
2. Launch EC2 instance for terraform in Ohio region  
3. Terraform-Server1 - Amazon Linux 2 image - t2.micro instance type - terraform-server1-key created new - keep same default vpc - default security group - Launch instance  
4. click on the launched instance - copy public ip  
5. download mobaxterm from https://mobaxterm.mobatek.net/download-home-edition.html so that you can ssh into any server  . Download installer edition and double click to start using it.  Installed in C:\Program Files (x86)\Mobatek\  , pin to taskbar  
6. in mobaxterm - session - ssh - paste public ip address of the instance - 3.17.145.31 - specify username as ec2-user - advanced ssh setting - private key - choose path for the pem key created while launching instance - C:\Users\91887\Desktop\RnD_CICD\AWS\terraform-server1-key.pem  - ok  
7. Was unable to connect hence added 2 new rules in default sg - ssh tcp 22 anyehere ipv4 -  0.0.0.0./0 - http tcp 80 anywhere ipv4 0.0.0.0/0 - save  
8. Check Network ACLs (Advanced)  
Ensure that your VPC's Network ACL (NACL) rules allow inbound and outbound traffic on port 22.  
Go to the VPC dashboard.  
Under Network ACLs, check the rules for the subnet where your EC2 instance resides.  
Ensure there are allow rules for inbound and outbound traffic on port 22 (SSH).  

9. Mobaxterm session established  
10. [ec2-user@ip-172-31-31-122 ~]$       <------ this will be seen in xterm shell
11. [ec2-user@ip-172-31-31-122 ~]$ **sudo yum update  **
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
amzn2-core                                                                                             | 3.6 kB  00:00:00    
No packages marked for update  
[ec2-user@ip-172-31-31-122 ~]$ ** sudo yum install -y yum-utils  **
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
Package yum-utils-1.1.31-46.amzn2.0.1.noarch already installed and latest version  
Nothing to do
[ec2-user@ip-172-31-31-122 ~]$ **sudo yum-config-manager --add-repo http://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo  **
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
adding repo from: http://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo  
grabbing file http://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo to /etc/yum.repos.d/hashicorp.repo  
repo saved to /etc/yum.repos.d/hashicorp.repo
This URL points to the HashiCorp repository specifically for Amazon Linux. By adding this repository, you enable your system to find and install packages published by HashiCorp, such as Terraform, Vault, Consul, and others.
[ec2-user@ip-172-31-31-122 ~]$ **sudo yum -y install terraform**  
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
amzn2-core                                                                                             | 3.6 kB  00:00:00  
hashicorp                                                                                              | 1.5 kB  00:00:00  
hashicorp/x86_64/primary                                                                               | 295 kB  00:00:00  
hashicorp                                                                                                             2117/2117  
Resolving Dependencies  
--> Running transaction check  
    Installed:  
  terraform.x86_64 0:1.9.8-1  
  
Dependency Installed:  
  git.x86_64 0:2.40.1-1.amzn2.0.3       git-core.x86_64 0:2.40.1-1.amzn2.0.3   git-core-doc.noarch 0:2.40.1-1.amzn2.0.3   
  perl-Error.noarch 1:0.17020-2.amzn2   perl-Git.noarch 0:2.40.1-1.amzn2.0.3   perl-TermReadKey.x86_64 0:2.30-20.amzn2.0.2  

Complete!  
Check if terraform is installed   
[ec2-user@ip-172-31-31-122 ~]$ **terraform  **
Usage: terraform [global options] <subcommand> [args]  

The available commands for execution are listed below.  
The primary workflow commands are given first, followed by  
less common or more advanced commands.  

Main commands:  
  init          Prepare your working directory for other commands  
  validate      Check whether the configuration is valid   














