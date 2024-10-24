# Jenkins_Terraform_AWS
Repo to deploy infra to AWS using terraform. Building and deployment of a Java Web Application in AWS EKS using Ansible. 

Ref youtube video : https://www.youtube.com/watch?v=NKUOSc9pCfk&t=12s
github repo https://github.com/DinmaMerciBoi/MyProjectApp

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
11. [ec2-user@ip-172-31-31-122 ~]$ **sudo yum update**
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
amzn2-core                                                                                             | 3.6 kB  00:00:00    
No packages marked for update  
[ec2-user@ip-172-31-31-122 ~]$ **sudo yum install -y yum-utils**
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
Package yum-utils-1.1.31-46.amzn2.0.1.noarch already installed and latest version  
Nothing to do
[ec2-user@ip-172-31-31-122 ~]$ **sudo yum-config-manager --add-repo http://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo**
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
[ec2-user@ip-172-31-31-122 ~]$ **terraform**
Usage: terraform [global options] <subcommand> [args]  

The available commands for execution are listed below.  
The primary workflow commands are given first, followed by  
less common or more advanced commands.  

Main commands:  
  init          Prepare your working directory for other commands  
  validate      Check whether the configuration is valid   
  
12. Set hostname:  [ec2-user@ip-172-31-31-122 ~]$ **sudo hostnamectl set-hostname Terraform-Server**
13.  After changing hostname restart server using [ec2-user@ip-172-31-31-122 ~]$ **sudo init 6** now you will be able to see [ec2-user@terraform-server ~]$

     **Setting up jenkins server using Terraform**
15. Create s3 bucket so that we can store our state files remotely - ohio - project-register-terraform-server name - acls disabled - enable versioning - create
16. Create a folder in server named jenkins so that we can store all terraform scripts related to jenkins in there. [ec2-user@terraform-server ~]$ **mkdir jenkins && cd jenkins**  
17. copy contents of github repo Jenkins_Server/provider.tf to provider.tf inside the jenkins folder of server Same for data.tf main.tf security.tf variables.tf
18. Create IAM role - global - iam - create role - AWS Service - EC2 - next - adminstrator access - next - rolename Terraform-role - create role
19. Modify iam role on terraform-server1 instance to add as Terraform-Role with admin access
20. [ec2-user@terraform-server jenkins]$ **ls**
data.tf  main.tf  provider.tf  security.tf  variables.tf  
21. [ec2-user@terraform-server jenkins]$ **terraform init**  
Initializing the backend...  
Initializing provider plugins... 
- Finding hashicorp/aws versions matching "~> 4.0"...  
- Installing hashicorp/aws v4.67.0...  
- Installed hashicorp/aws v4.67.0 (signed by HashiCorp)   
Terraform has created a lock file .terraform.lock.hcl to record the provider  
selections it made above. Include this file in your version control repository  
so that Terraform can guarantee to make the same selections by default when  
you run "terraform init" in the future.   

Terraform has been successfully initialized!    
22. [ec2-user@terraform-server jenkins]$ **terraform fmt**  
provider.tf  
[ec2-user@terraform-server jenkins]$ **terraform fmt**    
[ec2-user@terraform-server jenkins]$ **terraform validate**  
Success! The configuration is valid.  
23. [ec2-user@terraform-server jenkins]$ **terraform plan**
data.aws_ami.amazonlinux2: Reading...  
data.aws_ami.amazonlinux2: Read complete after 2s [id=ami-0e5be607c4ed9bc92]  

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the  
following symbols:  
  + create  

Terraform will perform the following actions:  

  # aws_instance.JenkinsServer will be created  
  + resource "aws_instance" "JenkinsServer" {  
      + ami                                  = "ami-0e5be607c4ed9bc92"  
      + arn                                  = (known after apply)  
      + associate_public_ip_address          = (known after apply)  
      + availability_zone                    = (known after apply)  
      + cpu_core_count                       = (known after apply)  
      + cpu_threads_per_core                 = (known after apply)  
      + disable_api_stop                     = (known after apply)  
      + disable_api_termination              = (known after apply)  
      + ebs_optimized                        = (known after apply)  
      + get_password_data                    = false  
      + host_id                              = (known after apply)  
      + host_resource_group_arn              = (known after apply)  
      + iam_instance_profile                 = (known after apply)  
      + id                                   = (known after apply)  
      + instance_initiated_shutdown_behavior = (known after apply)  
      + instance_state                       = (known after apply)  
      + instance_type                        = "t2.micro"  
      + ipv6_address_count                   = (known after apply)  
      + ipv6_addresses                       = (known after apply)  
      + key_name                             = "terraform-server1-key"  
      + monitoring                           = (known after apply)  
      + outpost_arn                          = (known after apply)  
      + password_data                        = (known after apply)  
      + placement_group                      = (known after apply)  
      + placement_partition_number           = (known after apply)  
      + primary_network_interface_id         = (known after apply)  
      + private_dns                          = (known after apply)  
      + private_ip                           = (known after apply)  
      + public_dns                           = (known after apply)  
      + public_ip                            = (known after apply)  
      + secondary_private_ips                = (known after apply)  
      + security_groups                      = (known after apply)  
      + source_dest_check                    = true  
      + subnet_id                            = (known after apply)  
      + tags                                 = {  
          + "Name" = "Jenkins-Server"  
        }  
      + tags_all                             = {  
          + "Name" = "Jenkins-Server"  
        }  
      + tenancy                              = (known after apply)  
      + user_data                            = (known after apply)  
      + user_data_base64                     = (known after apply)  
      + user_data_replace_on_change          = false  
      + vpc_security_group_ids               = (known after apply)  

      + capacity_reservation_specification (known after apply)   
  
      + cpu_options (known after apply)
  
      + ebs_block_device (known after apply)

      + enclave_options (known after apply)

      + ephemeral_block_device (known after apply)

      + maintenance_options (known after apply)

      + metadata_options (known after apply)

      + network_interface (known after apply)

      + private_dns_name_options (known after apply)

      + root_block_device (known after apply)
    }

  # aws_security_group.web-traffic will be created  
  + resource "aws_security_group" "web-traffic" {  
      + arn                    = (known after apply)  
      + description            = "Managed by Terraform"  
      + egress                 = [  
          + {
              + cidr_blocks      = [  
                  + "0.0.0.0/0",  
                ]  
              + from_port        = 0  
              + ipv6_cidr_blocks = []  
              + prefix_list_ids  = []   
              + protocol         = "-1"  
              + security_groups  = []  
              + self             = false  
              + to_port          = 0  
                # (1 unchanged attribute hidden)  
            },  
        ]  
      + id                     = (known after apply)  
      + ingress                = [  
          + {
              + cidr_blocks      = [  
                  + "0.0.0.0/0",  
                ]  
              + from_port        = 0  
              + ipv6_cidr_blocks = []  
              + prefix_list_ids  = []  
              + protocol         = "-1"  
              + security_groups  = []  
              + self             = false  
              + to_port          = 0  
                # (1 unchanged attribute hidden)  
            },
        ]
      + name                   = "My_Security_Group1"  
      + name_prefix            = (known after apply)  
      + owner_id               = (known after apply)  
      + revoke_rules_on_delete = false  
      + tags                   = {  
          + "Name" = "My_SG1"    
        }  
      + tags_all               = {  
          + "Name" = "My_SG1"  
        }  
      + vpc_id                 = (known after apply)  
    }  

Plan: 2 to add, 0 to change, 0 to destroy.  
 
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run
"terraform apply" now.  
24. **terraform apply**
Plan: 2 to add, 0 to change, 0 to destroy.  

Do you want to perform these actions?   
  Terraform will perform the actions described above.  
  Only 'yes' will be accepted to approve.  

  Enter a value: yes  

aws_security_group.web-traffic: Creating...  
aws_security_group.web-traffic: Creation complete after 3s [id=sg-0d075053eb9c55b05]  
aws_instance.JenkinsServer: Creating...    
aws_instance.JenkinsServer: Still creating... [10s elapsed]  
aws_instance.JenkinsServer: Creation complete after 12s [id=i-01f1c8606bf1da7e5]  

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.  



 















