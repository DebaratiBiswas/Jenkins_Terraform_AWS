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
EC2 AND SECURITY GROUP ARE CREATED HERE  
**JENKINS SERVER**  
26. login to jenkins server using the public ip and same pem key. See for the details in EC2 Jenkins-Server newly launched  
Install Jenkins  
Login as root user 
[ec2-user@ip-172-31-30-93 ~]$ **sudo su**  
[root@ip-172-31-30-93 ec2-user]#   **sudo yum update -y**  
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
amzn2-core                                                                                             | 3.6 kB  00:00:00  
No packages marked for update  
[root@ip-172-31-30-93 ec2-user]# **wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo**  
--2024-10-25 05:29:02--  https://pkg.jenkins.io/redhat-stable/jenkins.repo  
Resolving pkg.jenkins.io (pkg.jenkins.io)... 146.75.78.133, 2a04:4e42:83::645  
Connecting to pkg.jenkins.io (pkg.jenkins.io)|146.75.78.133|:443... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: 85  
Saving to: ‘/etc/yum.repos.d/jenkins.repo’  

100%[====================================================================================>] 85          --.-K/s   in 0s  

2024-10-25 05:29:02 (7.86 MB/s) - ‘/etc/yum.repos.d/jenkins.repo’ saved [85/85]  
[root@ip-172-31-30-93 ec2-user]# **rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key**  
[root@ip-172-31-30-93 ec2-user]# **yum upgrade**  
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd  
amzn2-core                                                                                             | 3.6 kB  00:00:00
jenkins                                                                                                | 2.9 kB  00:00:00
jenkins/primary_db                                                                                     |  51 kB  00:00:00
No packages marked for update
[root@ip-172-31-30-93 ec2-user]# **amazon-linux-extras install epel**
This installs the EPEL repository, which allows you to access and install many more software packages that are not available by default in the standard Amazon Linux repositories.  
These include development tools, libraries, and various utilities.  
Installing epel-release
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
Cleaning repos: amzn2-core amzn2extra-docker amzn2extra-epel jenkins
14 metadata files removed
6 sqlite files removed
0 metadata files removed
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
amzn2-core                                                                                             | 3.6 kB  00:00:00
amzn2extra-docker                                                                                      | 2.9 kB  00:00:00
amzn2extra-epel                                                                                        | 3.0 kB  00:00:00
jenkins                                                                                                | 2.9 kB  00:00:00
(1/8): amzn2-core/2/x86_64/group_gz                                                                    | 2.7 kB  00:00:00
(2/8): amzn2-core/2/x86_64/updateinfo                                                                  | 983 kB  00:00:00
(3/8): amzn2extra-epel/2/x86_64/primary_db                                                             | 1.8 kB  00:00:00
(4/8): amzn2extra-docker/2/x86_64/updateinfo                                                           |  20 kB  00:00:00
(5/8): amzn2extra-docker/2/x86_64/primary_db                                                           | 114 kB  00:00:00
(6/8): amzn2extra-epel/2/x86_64/updateinfo                                                             |   76 B  00:00:00
(7/8): jenkins/primary_db                                                                              |  51 kB  00:00:00
(8/8): amzn2-core/2/x86_64/primary_db                                                                  |  71 MB  00:00:01
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-11 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================================
 Package                         Arch                      Version                   Repository                          Size
==============================================================================================================================
Installing:
 epel-release                    noarch                    7-11                      amzn2extra-epel                     15 k

Transaction Summary
==============================================================================================================================
Install  1 Package

Total download size: 15 k
Installed size: 24 k
Is this ok [y/d/N]: y
Downloading packages:
epel-release-7-11.noarch.rpm                                                                           |  15 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-11.noarch                                                                                   1/1
  Verifying  : epel-release-7-11.noarch                                                                                   1/1

Installed:
  epel-release.noarch 0:7-11

Complete!
  2  httpd_modules            available    [ =1.0  =stable ]  
  3  memcached1.5             available    \
        [ =1.5.1  =1.5.16  =1.5.17 ]  
  9  R3.4                     available    [ =3.4.3  =stable ]
 10  rust1                    available    \  
        [ =1.22.1  =1.26.0  =1.26.1  =1.27.2  =1.31.0  =1.38.0
          =stable ]  
 18  libreoffice              available    \  
        [ =5.0.6.2_15  =5.3.6.1  =stable ]  
 19  gimp                     available    [ =2.8.22 ]  
 20 †docker=latest            enabled      \  
        [ =17.12.1  =18.03.1  =18.06.1  =18.09.9  =stable ]  
 21  mate-desktop1.x          available    \   
        [ =1.19.0  =1.20.0  =stable ]
 22  GraphicsMagick1.3        available    \  
        [ =1.3.29  =1.3.32  =1.3.34  =stable ]
 24  epel=latest              enabled      [ =7.11  =stable ]  
 25  testing                  available    [ =1.0  =stable ]  
 26  ecs                      available    [ =stable ]  
 27 †corretto8                available    \  
        [ =1.8.0_192  =1.8.0_202  =1.8.0_212  =1.8.0_222  =1.8.0_232
          =1.8.0_242  =stable ]
 32  lustre2.10               available    \  
        [ =2.10.5  =2.10.8  =stable ]
 34  lynis                    available    [ =stable ]  
 36  BCC                      available    [ =0.x  =stable ]  
 37  mono                     available    [ =5.x  =stable ]  
 38  nginx1                   available    [ =stable ]  
 40  mock                     available    [ =stable ]  
 43  livepatch                available    [ =stable ]  
 45  haproxy2                 available    [ =stable ]  
 46  collectd                 available    [ =stable ]  
 47  aws-nitro-enclaves-cli   available    [ =stable ]  
 48  R4                       available    [ =stable ]  
 49  kernel-5.4               available    [ =stable ]  
 50  selinux-ng               available    [ =stable ]  
 52  tomcat9                  available    [ =stable ]  
 53  unbound1.13              available    [ =stable ]
 54 †mariadb10.5              available    [ =stable ]  
 55  kernel-5.10              available    [ =stable ]  
 56  redis6                   available    [ =stable ]  
 58 †postgresql12             available    [ =stable ]  
 59 †postgresql13             available    [ =stable ]  
 60  mock2                    available    [ =stable ]
 61  dnsmasq2.85              available    [ =stable ]  
 62  kernel-5.15              available    [ =stable ]  
 63 †postgresql14             available    [ =stable ]  
 64  firefox                  available    [ =stable ]  
 65  lustre                   available    [ =stable ]  
 66 †php8.1                   available    [ =stable ]  
 67  awscli1                  available    [ =stable ]
 68 †php8.2                   available    [ =stable ]  
 69  dnsmasq                  available    [ =stable ]  
 70  unbound1.17              available    [ =stable ]  
 72  collectd-python3         available    [ =stable ]  
† Note on end-of-support. Use 'info' subcommand.  

Install jdk11 for Jenkins  
[root@ip-172-31-30-93 ec2-user]# **amazon-linux-extras install java-openjdk11 -y**  
Topic java-openjdk11 has end-of-support date of 2024-09-30  
Installing java-11-openjdk  
[root@ip-172-31-30-93 ec2-user]# **yum install java-11-amazon-corretto -y**  
This is the JDK (Java Development Kit) package for Java 11 provided by Amazon.
Amazon Corretto is a no-cost, multiplatform, production-ready distribution of OpenJDK. It’s designed for use in cloud and enterprise applications and is regularly updated with security patches.  
Installed:  
  java-11-amazon-corretto.x86_64 1:11.0.24+8-1.amzn2  

Dependency Installed:  
  dejavu-sans-mono-fonts.noarch 0:2.33-6.amzn2                            dejavu-serif-fonts.noarch 0:2.33-6.amzn2  
  java-11-amazon-corretto-headless.x86_64 1:11.0.24+8-1.amzn2             libXinerama.x86_64 0:1.1.3-2.1.amzn2.0.2  
  libXrandr.x86_64 0:1.5.1-2.amzn2.0.3                                    libXt.x86_64 0:1.1.5-3.amzn2.0.2  
Complete!  
[root@ip-172-31-30-93 ec2-user]# **yum install jenkins -y**   
Running transaction  
  Installing : jenkins-2.462.3-1.1.noarch                                                                                 1/1  
  Verifying  : jenkins-2.462.3-1.1. noarch                                                                                 1/1  

Installed:    
  jenkins.noarch 0:2.462.3-1.1    
Complete!  
[root@ip-172-31-30-93 ec2-user]# **systemctl enable jenkins**  
Created symlink from /etc/systemd/system/multi-user.target.wants/jenkins.service to /usr/lib/systemd/system/jenkins.service.  
[root@ip-172-31-30-93 ec2-user]# **systemctl start jenkins**  
[root@ip-172-31-30-93 ec2-user]# **systemctl status jenkins**  
● jenkins.service - Jenkins Continuous Integration Server  
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; vendor preset: disabled)  
   Active: active (running) since Fri 2024-10-25 06:01:54 UTC; 10s ago  
 Main PID: 9289 (java)  
   CGroup: /system.slice/jenkins.service  
           └─9289 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=%C/jenkins/war --httpPo...  

Oct 25 06:01:46 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: 8319639b95014bd6a0fd8f9a8090add0  
Oct 25 06:01:46 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: This may also be found at: /var/lib/jenkins/se...ord  
Oct 25 06:01:46 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: **********************************************...***  
Oct 25 06:01:46 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: **********************************************...***  
Oct 25 06:01:46 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: **********************************************...***  
Oct 25 06:01:54 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: 2024-10-25 06:01:54.019+0000 [id=31]        IN...ion  
Oct 25 06:01:54 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: 2024-10-25 06:01:54.052+0000 [id=23]        IN...ing  
Oct 25 06:01:54 ip-172-31-30-93.us-east-2.compute.internal systemd[1]: Started Jenkins Continuous Integration Server.  
Oct 25 06:01:54 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: 2024-10-25 06:01:54.425+0000 [id=46]        IN...ler  
Oct 25 06:01:54 ip-172-31-30-93.us-east-2.compute.internal jenkins[9289]: 2024-10-25 06:01:54.426+0000 [id=46]        IN... #1  
Hint: Some lines were ellipsized, use -l to show in full.  
[root@ip-172-31-30-93 ec2-user]# **hostnamectl set-hostname jenkins-server**  
[root@ip-172-31-30-93 ec2-user]# **init 6**  
No packages needed for security; 2 packages available   
Run "sudo yum update" to apply all updates.   
[ec2-user@jenkins-server ~]$ **sudo su**
[root@jenkins-server ec2-user]# **sudo yum update**
[root@jenkins-server ec2-user]# **java -version**  
openjdk version "11.0.24" 2024-07-16 LTS  
OpenJDK Runtime Environment Corretto-11.0.24.8.1 (build 11.0.24+8-LTS)  
OpenJDK 64-Bit Server VM Corretto-11.0.24.8.1 (build 11.0.24+8-LTS, mixed mode)  

[root@jenkins-server ec2-user]# **javac --version**  
javac 11.0.24  
check for sctive status of jenkins
[root@jenkins-server ec2-user]# **systemctl status jenkins**  
● jenkins.service - Jenkins Continuous Integration Server  
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; vendor preset: disabled)  
   **Active**: active (running) since Fri 2024-10-25 06:11:54 UTC; 3min 42s ago  
 Main PID: 2852 (java)  
   CGroup: /system.slice/jenkins.service  
           └─2852 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=%C/jenkins/war --httpPo...  
[root@jenkins-server ec2-user]# **systemctl start jenkins**
27. INSTALL AND CONFIGURE MAVEN IN JENKINS SERVER   
https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz
[root@jenkins-server ec2-user]# **cd /opt**  
[root@jenkins-server opt]# **wget https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz**  
--2024-10-25 06:22:46--  https://dlcdn.apache.org/maven/maven-3/3.9.5/binaries/apache-maven-3.9.5-bin.tar.gz  
Resolving dlcdn.apache.org (dlcdn.apache.org)... 151.101.2.132, 2a04:4e42::644  
Connecting to dlcdn.apache.org (dlcdn.apache.org)|151.101.2.132|:443... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: 9359994 (8.9M) [application/x-gzip]  
Saving to: ‘apache-maven-3.9.5-bin.tar.gz’  

100%[====================================================================================>] 9,359,994   --.-K/s   in 0.08s  

2024-10-25 06:22:47 (105 MB/s) - ‘apache-maven-3.9.5-bin.tar.gz’ saved [9359994/9359994]   

[root@jenkins-server opt]# **ls**
apache-maven-3.9.5-bin.tar.gz  aws  rh
[root@jenkins-server opt]# **tar -xzvf apache-maven-3.9.5-bin.tar.gz**  
[root@jenkins-server opt]# **ls**  
apache-maven-3.9.5  apache-maven-3.9.5-bin.tar.gz  aws  rh  
[root@jenkins-server opt]# rm apache-maven-3.9.5-bin.tar.gz  
rm: remove regular file ‘apache-maven-3.9.5-bin.tar.gz’? y  
[root@jenkins-server opt]# **ls**  
apache-maven-3.9.5  aws  rh  
[root@jenkins-server opt]# **mv apache-maven-3.9.5 maven**  
[root@jenkins-server opt]# **ls**  
aws  maven  rh  
[root@jenkins-server opt]#   
[root@jenkins-server opt]# **cd maven**  
[root@jenkins-server maven]# **ls**    
bin  boot  conf  lib  LICENSE  NOTICE  README.txt  
[root@jenkins-server maven]# **cd bin**  
[root@jenkins-server bin]# ls  
m2.conf  mvn  mvn.cmd  mvnDebug  mvnDebug.cmd  mvnyjp  
[root@jenkins-server bin]# **./mvn -v**  
Apache Maven 3.9.5 (57804ffe001d7215b5e7bcb531cf83df38f93546)  
Maven home: /opt/maven  
Java version: 11.0.24, vendor: Amazon.com Inc., runtime: /usr/lib/jvm/java-11-amazon-corretto.x86_64  
Default locale: en_US, platform encoding: UTF-8  
OS name: "linux", version: "4.14.352-268.569.amzn2.x86_64", arch: "amd64", family: "unix" 


TO EXECUTE MAVEN FROM ANYWHERE NOT ONLY bin    
change in ./bash_profile the PATH   
[root@jenkins-server bin]# cd ..  
[root@jenkins-server maven]# cd ~  
[root@jenkins-server ~]# ls -a  
.  ..  .bash_history  .bash_logout  .bash_profile  .bashrc  .cshrc  .ssh  .tcshrc  
[root@jenkins-server ~]# vi .bash_profile   
[root@jenkins-server ~]# **find / -name java-11***  
/etc/alternatives/java-11-amazon-corretto   
/etc/alternatives/java-11   
/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64      <---- TAKE tHIS>
/usr/lib/jvm/java-11-amazon-corretto.x86_64   
/usr/lib/jvm/java-11-amazon-corretto   
/usr/lib/jvm/java-11   
/usr/lib/jvm/java-11-openjdk   
/usr/share/doc/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64   
/usr/share/icons/hicolor/48x48/apps/java-11-openjdk.png  
/usr/share/icons/hicolor/16x16/apps/java-11-openjdk.png   
/usr/share/icons/hicolor/24x24/apps/java-11-openjdk.png   
/usr/share/icons/hicolor/32x32/apps/java-11-openjdk.png     
[root@jenkins-server ~]# **vi .bash_profile**  
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi
# Maven home setup
M2_HOME=/opt/maven
M2=/opt/maven/bin
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64
# User specific environment and startup programs

PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2

export PATH[root@jenkins-server ~]# **echo $PATH**  
/sbin:/bin:/usr/sbin:/usr/bin  
[root@jenkins-server ~]# **source .bash_profile**  
[root@jenkins-server ~]# **echo $PATH**  
/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64:/opt/maven:/opt/maven/bin
[root@jenkins-server ~]# 

[root@jenkins-server ~]# **mvn -v**  
Apache Maven 3.9.5 (57804ffe001d7215b5e7bcb531cf83df38f93546)    
Maven home: /opt/maven  
Java version: 11.0.24, vendor: Amazon.com Inc., runtime: /usr/lib/jvm/java-11-amazon-corretto.x86_64  
Default locale: en_US, platform encoding: UTF-8  
OS name: "linux", version: "4.14.352-268.569.amzn2.x86_64", arch: "amd64", family: "unix"  

CONFIGURE JENKINS INTERFACE AND MAVEN INTEGRATION     
28. gO TO THE PUBLIC IP OF Jenkis-Server ec2 instance add port http://3.12.196.76:8080/  MUST BE HTTP
get initial password from /var/lib/jenkins/secrets/initialAdminPassword  
[root@jenkins-server ~]# **cat /var/lib/jenkins/secrets/initialAdminPassword**  
8319639b95014bd6a0fd8f9a8090add0     
jenkins creds same as local. setup       
install recommended plugins      
manage jenkins - plugins - install maven integration plugin   
[root@jenkins-server ~]# **echo $JAVA_HOME**  
/usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64  
manage jenkins - tools - add jdk - java11 - /usr/lib/jvm/java-11-openjdk-11.0.23.0.9-2.amzn2.0.1.x86_64   
add maven - maven - /opt/maven   
manage jenkins - plugins - installed plugin - github search - disable GitHub Branch Source Plugin - enable gihub plugin - restart   
[root@jenkins-server ~]# **yum install git -y**    
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd     
amzn2-core                                        
Installed:       
  git.x86_64 0:2.40.1-1.amzn2.0.3       

Dependency Installed:          
  git-core.x86_64 0:2.40.1-1.amzn2.0.3   git-core-doc.noarch 0:2.40.1-1.amzn2.0.3      perl-Error.noarch 1:0.17020-2.amzn2             
  perl-Git.noarch 0:2.40.1-1.amzn2.0.3   perl-TermReadKey.x86_64 0:2.30-20.amzn2.0.2        

Complete!             
29. CREATE A TEST1 JOB as maven project  
configure -> git scm -> url of git https://github.com/DebaratiBiswas/CICD_mvn_java.git -> ./main -> pom.xml in root with clean install as cmd -> apply -> save -> build now   

30. Deploy ANSIBLE SERVER through Terraform server  
mkdir ansible and paste all files of Ansible_Server  
[ec2-user@terraform-server ~]$ **cd ansible**  
[ec2-user@terraform-server ansible]$ ls  
data.tf  main.tf  provider.tf  security.tf  variables.tf    
data.tf     
It looks like you're defining a data source to fetch the most recent Amazon Linux 2 AMI using Terraform.  
Data Source:   

data "aws_ami" "amazonlinux2" defines a data source to look up an Amazon Machine Image (AMI).  
most_recent = true ensures that the most recent AMI matching the filters will be selected.  
Filters:  

The first filter matches AMIs that are owned by Amazon.  
The second filter uses a wildcard to match the AMI name pattern for Amazon Linux 2.   

main.tf  
file defines an AWS EC2 instance resource for an Ansible server. References the fetched AMI, instance type, key name, and the security group.  

provider.tf  
required_version: Specifies the minimum version of Terraform required to run this configuration. The ~> operator means any version greater than or equal to 1.0.0 but less than 2.0.0.

required_providers: Lists the providers needed for this configuration.

aws: Specifies that the AWS provider from HashiCorp should be used.
source: Indicates the source of the provider.
version: Defines the version constraint for the AWS provider. Here, it means any version greater than or equal to 4.0.0 but less than 5.0.0. Specifying a version is good practice to ensure stability and compatibility, especially in production environments.
backend "s3": Configures the backend for storing the Terraform state file.

bucket: The S3 bucket where the state file will be stored.
key: The path within the S3 bucket for the state file. This allows you to organize state files in a specific structure.
region: The AWS region where the S3 bucket is located. This ensures that Terraform interacts with the correct bucket.  
provider "aws": Configures the AWS provider with specific settings.
region: Sets the AWS region where resources will be created. In this case, it is set to us-east-2.  

security.tf - for setting security group with ingress egress rules  
variables.tf - define variables to be used by tf  
[ec2-user@terraform-server ansible]$ **terraform init**  
Initializing the backend...  
╷
│ Error: Backend configuration changed  
│
│ A change in the backend configuration has been detected, which may require migrating existing state.  
│
│ If you wish to attempt automatic migration of the state, use "terraform init -migrate-state".  
│ If you wish to store the current configuration with no changes to the state, use "terraform init   -reconfigure".    

**The error is because there are lock file present which shows terraform was initialized before. So we will delete it.**  
[ec2-user@terraform-server ansible]$ **ls -la**  
total 24  
drwxrwxr-x 3 ec2-user ec2-user  139 Oct 29 06:34 .  
drwx------ 6 ec2-user ec2-user  161 Oct 29 06:34 ..  
-rw-rw-r-- 1 ec2-user ec2-user  202 Oct 28 06:09 data.tf  
-rw-rw-r-- 1 ec2-user ec2-user  295 Oct 29 05:53 main.tf  
-rw-rw-r-- 1 ec2-user ec2-user  368 Oct 29 05:58 provider.tf  
-rw-rw-r-- 1 ec2-user ec2-user  395 Oct 29 06:30 security.tf  
drwxr-xr-x 3 ec2-user ec2-user   48 Oct 28 06:09 .terraform    
-rw-r--r-- 1 ec2-user ec2-user 1406 Oct 28 06:09 .terraform.lock.hcl   
-rw-rw-r-- 1 ec2-user ec2-user  518 Oct 28 06:09 variables.tf   
[ec2-user@terraform-server ansible]$ rm -rf .terraform .terraform.lock.hcl   
[ec2-user@terraform-server ansible]$ terraform init   
Initializing the backend...    

Successfully configured the backend "s3"! Terraform will automatically    
use this backend unless the backend configuration changes.    
Initializing provider plugins...   
- Finding hashicorp/aws versions matching "~> 4.0"...   
- Installing hashicorp/aws v4.67.0...     
- Installed hashicorp/aws v4.67.0 (signed by HashiCorp)     
Terraform has created a lock file .terraform.lock.hcl to record the provider    
selections it made above. Include this file in your version control repository    
so that Terraform can guarantee to make the same selections by default when     
you run "terraform init" in the future.   

Terraform has been successfully initialized!    

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands  
shoul[ec2-user@terraform-server ansible]$ **terraform fmt**   
[ec2-user@terraform-server ansible]$ **terraform validate**  
Success! The configuration is valid.   

[ec2-user@terraform-server ansible]$ terraform plan  
data.aws_ami.amazonlinux2: Reading...        
data.aws_ami.amazonlinux2: Read complete after 1s [id=ami-0e5be607c4ed9bc92]       

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create
d now work.    

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.      
Plan: 2 to add, 0 to change, 0 to destroy.  
[ec2-user@terraform-server ansible]$ **terraform apply  --auto-approve**   
data.aws_ami.amazonlinux2: Reading...
data.aws_ami.amazonlinux2: Read complete after 1s [id=ami-0e5be607c4ed9bc92]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:  

  # aws_instance.AnsibleServer will be created      
  + resource "aws_instance" "AnsibleServer" {  .......    


Plan: 2 to add, 0 to change, 0 to destroy.   
aws_security_group.web-traffic: Creating...       
aws_security_group.web-traffic: Creation complete after 2s [id=sg-027d5ded60d353430]  
aws_instance.AnsibleServer: Creating...    
aws_instance.AnsibleServer: Still creating... [10s elapsed]    
aws_instance.AnsibleServer: Creation complete after 12s [id=i-01f6b87799b84784a]   
    
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.     

31. Install and Configure Ansible  
[ec2-user@ip-172-31-16-224 ~]$ **sudo hostnamectl set-hostname ansible-server**
[root@ansible-server ec2-user]# **sudo su -**
Last login: Tue Oct 29 06:49:47 UTC 2024 on pts/0  
[root@ansible-server ~]#  
[root@ansible-server ~]# **useradd ansadmin**
[root@ansible-server ~]# **passwd ansadmin**
Changing password for user ansadmin.  
New password:    ## password
BAD PASSWORD: The password fails the dictionary check - it is based on a dictionary word  
Retype new password:  
passwd: all authentication tokens updated successfully.  
ADD USER TO SUDO   
[root@ansible-server ~]# **visudo**   
ansadmin ALL=(ALL)       NOPASSWD: ALL   
[root@ansible-server ~]# **cd /etc/ssh**   
[root@ansible-server ssh]# **ls -a**   
.   moduli      sshd_config         ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub
..  ssh_config  ssh_host_ecdsa_key  ssh_host_ed25519_key    ssh_host_rsa_key   
[root@ansible-server ssh]# **vi sshd_config**   

SET PASSWORD AUTHENTICATION TO YES  
PasswordAuthentication yes  
GET INTO ANSADMIN and GENERATE KEY SO IT COULKD MANAGE AKLL SEVER WITH THE USER  
[root@ansible-server ssh]# **service sshd reload**
Redirecting to /bin/systemctl reload sshd.service
[root@ansible-server ssh]# **su ansadmin**
[ansadmin@ansible-server ssh]$ **cd ~**  
[ansadmin@ansible-server ~]$ **ssh-keygen**  
Generating public/private rsa key pair.  
Enter file in which to save the key (/home/ansadmin/.ssh/id_rsa):  
Created directory '/home/ansadmin/.ssh'.  
Enter passphrase (empty for no passphrase):  
Enter same passphrase again:  
Your identification has been saved in /home/ansadmin/.ssh/id_rsa.  
Your public key has been saved in /home/ansadmin/.ssh/id_rsa.pub.    
The key fingerprint is:    
SHA256:wj9Yf8m5wjKmpJLRIlJPVzbDVqoglfYNq3IisYRwsI4 ansadmin@ansible-server   
The key's randomart image is:   
+---[RSA 2048]----+
|..  .. . ..      |
|....o . B.       |
|+..... B.o       |
|=....o+..        |
|E= + o+ S        |
|= = =  = . . o   |
|.o B  o o.. =    |
|  o  o  +.o. .   |
|   .. .o o ..    |
+----[SHA256]-----+
[ansadmin@ansible-server ~]$   
[ansadmin@ansible-server ~]$ **ls .ssh/**   
id_rsa  id_rsa.pub   
THESE ARE PRIVATE AND PUBLIC KEYS  
32. INSTALL ANSIBLE  
[ansadmin@ansible-server ~]$ **sudo su**    
[root@ansible-server ansadmin]# **cd ~**   
[root@ansible-server ~]# **amazon-linux-extras install ansible2**    
[root@ansible-server ~]# **ansible --version**   
ansible 2.9.23   
  config file = /etc/ansible/ansible.cfg   
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']     
  ansible python module location = /usr/lib/python2.7/site-packages/ansible    
  executable location = /bin/ansible              
  python version = 2.7.18 (default, Dec 18 2023, 22:08:43) [GCC 7.3.1 20180712 (Red Hat 7.3.1-17)]   

33. INTEGRATE ANSIBLE WITH JENKINS  
open jenkins - manage jenkins - plugins - available plugins - publish over ssh - install   
restart - login 
manage jenkins - system - add ssh servers - name Ansible-Server - copy public ip of ansible server ec2 instance as hostname - username ansadmin - advanced pwd as password - apply - TEST CONFIG - save   
34. 
INSTALL DOCKER IN ANSIBLE SERVER   
[ansadmin@ansible-server ec2-user]$ **cd /opt**  
[ansadmin@ansible-server opt]$ **sudo mkdir docker**   
[ansadmin@ansible-server opt]$ ls
aws  docker  rh
[ansadmin@ansible-server opt]$ ll
total 0
drwxr-xr-x 4 root root 33 Oct 14 20:16 aws
**drwxr-xr-x 2 root root  6 Oct 29 07:40 docker**  
drwxr-xr-x 2 root root  6 Aug 16  2018 rh   
[ansadmin@ansible-server opt]$ **sudo chown ansadmin:ansadmin docker**  
[ansadmin@ansible-server opt]$ ll  
total 0   
drwxr-xr-x 4 root     root     33 Oct 14 20:16 aws     
**drwxr-xr-x 2 ansadmin ansadmin  6 Oct 29 07:40 docker**    
drwxr-xr-x 2 root     root      6 Aug 16  2018 rh    
go to test1 job - configure -  postbuild action - send build artifact over ssh - transfer set -  source fles - target/*.jar - remove prefix - target - Remote directory - apply save  
Build the job - check for the jar files in the ansible-server

[ansadmin@ansible-server opt]$ **date**  
Tue Oct 29 07:58:38 UTC 2024  
[ansadmin@ansible-server opt]$ **ls docker**  
hello-world-maven-1.0.0.jar  original-hello-world-maven-1.0.0.jar   
[ansadmin@ansible-server opt]$ **ll docker**  
total 1152   
-rw-rw-r-- 1 ansadmin ansadmin 587733 Oct 29 07:56 hello-world-maven-1.0.0.jar     
-rw-rw-r-- 1 ansadmin ansadmin 587733 Oct 29 07:56 original-hello-world-maven-1.0.0.jar   

DO THIS ACTIVITY WITH WEBAPP  
hence create a new jenkins job for maven build with git repo https://github.com/DebaratiBiswas/maven_webapp  

create maven-webapp job as maven project - configure -> git scm -> url of git https://github.com/DebaratiBiswas/maven_webapp.git -> ./main -> pom.xml in root with clean install as cmd -> apply -> save -> build now  

 configure -  postbuild action - send build artifact over ssh - transfer set -  source fles - target/*.war - remove prefix - target - Remote directory //opt//docker - apply - test - save  
Build the job - check for the jar files in the ansible-server
[ec2-user@ansible-server ~]$ **date**   
Wed Oct 30 06:20:24 UTC 2024  
[ec2-user@ansible-server ~]$ **cd /opt/docker**  
[ec2-user@ansible-server docker]$ **ll**  
total 1156  
-rw-rw-r-- 1 ansadmin ansadmin 587733 Oct 29 07:56 hello-world-maven-1.0.0.jar  
**-rw-rw-r-- 1 ansadmin ansadmin   2098 Oct 30 06:20 java-hello-world.war**  
-rw-rw-r-- 1 ansadmin ansadmin 587733 Oct 29 07:56 original-hello-world-maven-1.0.0.jar   
CONFIGURE DOCKER IN ANSIBLE SERVER  
35. [ec2-user@ansible-server docker]$ **sudo yum install docker**  

Installed:  
  docker.x86_64 0:25.0.6-1.amzn2.0.2  

Dependency Installed:   
  containerd.x86_64 0:1.7.22-1.amzn2.0.2        libcgroup.x86_64 0:0.41-21.amzn2        pigz.x86_64 0:2.3.4-1.amzn2.0.1
  runc.x86_64 0:1.1.14-1.amzn2   

Complete!   

[ec2-user@ansible-server docker]$ **id ansadmin**   
uid=1001(ansadmin) gid=1001(ansadmin) groups=1001(ansadmin)
[ec2-user@ansible-server docker]$ **sudo usermod -aG docker ansadmin**  
[ec2-user@ansible-server docker]$ **id ansadmin**  
uid=1001(ansadmin) gid=1001(ansadmin) groups=1001(ansadmin),992(docker)  
So, the command sudo usermod -aG docker ansadmin means:  

Add the user ansadmin to the docker group while keeping their existing group memberships intact. This allows ansadmin to run Docker commands without needing elevated permissions (i.e., without using sudo).  
[ec2-user@ansible-server docker]$ **sudo systemctl start docker**  
[ec2-user@ansible-server docker]$ **docker ps**  
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.44/containers/json": dial unix /var/run/docker.sock: connect: permission denied   
[ec2-user@ansible-server docker]$ **sudo init 6**  
[ec2-user@ansible-server ~]$ **sudo systemctl start docker**  
[ec2-user@ansible-server ~]$ **sudo systemctl status docker**  
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: **active** (running) since Wed 2024-10-30 06:34:14 UTC; 10s ago
     Docs: https://docs.docker.com   

36. Create dockerfile in ansible server  
[ec2-user@ansible-server ~]$ **sudo su - ansadmin**      
Last login: Tue Oct 29 07:39:18 UTC 2024 on pts/1   
Last failed login: Tue Oct 29 21:49:27 UTC 2024 from 202.4.115.172 on ssh:notty   
There was 1 failed login attempt since the last successful login.   
[ansadmin@ansible-server ~]$ **cd /opt/docker**   
[ansadmin@ansible-server docker]$ ls   
hello-world-maven-1.0.0.jar  java-hello-world.war  original-hello-world-maven-1.0.0.jar   
[ansadmin@ansible-server docker]$ **vi Dockerfile**  
[ansadmin@ansible-server docker]$ **cat Dockerfile**    
FROM tomcat:latest   
RUN cp -R /usr/local/tomcat/webapp.dist/* /usr/local/tomcat/webapps  
COPY ./*.war /usr/local/tomcat/webapps/java-hello-world.war   
[ansadmin@ansible-server docker]$ **ls**  
Dockerfile  hello-world-maven-1.0.0.jar  java-hello-world.war  original-hello-world-maven-1.0.0.jar  

37. CREATE ANSIBLE PLAYBOOK FOR DOCKER TASKS  
[ansadmin@ansible-server docker]$ **ifconfig**  
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 9001  
        inet 172.31.16.224  netmask 255.255.240.0  broadcast 172.31.31.255   <---check with private ip ansible server  
        inet6 fe80::4aa:86ff:feb2:e5eb  prefixlen 64  scopeid 0x20<link>  
        ether 06:aa:86:b2:e5:eb  txqueuelen 1000  (Ethernet)  
        RX packets 12287  bytes 80627772 (76.8 MiB)  
        RX errors 0  dropped 0  overruns 0  frame 0  
        TX packets 3055  bytes 252039 (246.1 KiB)  
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536  
        inet 127.0.0.1  netmask 255.0.0.0  
        inet6 ::1  prefixlen 128  scopeid 0x10<host>  
        loop  txqueuelen 1000  (Local Loopback)  
        RX packets 0  bytes 0 (0.0 B)  
        RX errors 0  dropped 0  overruns 0  frame 0   
        TX packets 0  bytes 0 (0.0 B)   
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0   
1:35 
CREATE ANSIBLE HOST FILE  
[ansadmin@ansible-server docker]$ **sudo vi /etc/ansible/hosts**  
[ansadmin@ansible-server docker]$ **cat /etc/ansible/hosts**
# This is the default ansible 'hosts' file.  
#  
# It should live in /etc/ansible/hosts    
[ansible]  
172.31.16.224   <------------ Private ip of the ansible server  
Copy the ssh key   
[ansadmin@ansible-server docker]$ ssh-copy-id 172.31.16.224   
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ansadmin/.ssh/id_rsa.pub"   
The authenticity of host '172.31.16.224 (172.31.16.224)' can't be established.   
ECDSA key fingerprint is SHA256:Zn83C/cRv2QqyiFn4WWKbTgxFjagHdJaaDHUyW/ojzA.   
ECDSA key fingerprint is MD5:d9:01:d9:83:ee:e3:9e:f1:12:82:ff:9e:19:47:e9:ac.   
Are you sure you want to continue connecting (yes/no)? yes    
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed         
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys      
ansadmin@172.31.16.224's password:         <-- password>

Number of key(s) added: 1       

Now try logging into the machine, with:   "ssh '172.31.16.224'"   
and check to make sure that only the key(s) you wanted were added.     
MANIFEST FILE TO CREATE DOCKER IMAGE FROM APPLICATION  
login to docker hub using gmail and password  
[ansadmin@ansible-server docker]$ **ls**  
Dockerfile  hello-world-maven-1.0.0.jar  java-hello-world.war  original-hello-world-maven-1.0.0.jar
[ansadmin@ansible-server docker]$ **vi java-hello-world-ci.yml**     
[ansadmin@ansible-server docker]$ **cat java-hello-world-ci.yml**  
- hosts: ansible  
  
  tasks:  
  - name: create docker image  
    command: docker build -t java-hello-word:latest .    
    args:      
      chdir: /opt/docker       
       
  - name: create tag to push image to dockerhub         
    command: docker tag java-hello-word:latest debsss/java-hello-word:latest            
                 
  - name: push to docker hub          
    command: docker push debsss/java-hello-word:latest           
check playbook for errors  
[ansadmin@ansible-server docker]$ **ansible-playbook java-hello-world-ci.yml --check**  

PLAY [ansible] ***************************************************************************************************************        

TASK [Gathering Facts] *******************************************************************************************************    
[WARNING]: Platform linux on host 172.31.16.224 is using the discovered Python interpreter at /usr/bin/python, but future
installation of another Python interpreter could change this. See
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.     
ok: [172.31.16.224]       

TASK [create docker image] ***************************************************************************************************    
skipping: [172.31.16.224]    

TASK [create tag to push image to dockerhub] *********************************************************************************     
skipping: [172.31.16.224]         

TASK [push to docker hub] ****************************************************************************************************       
skipping: [172.31.16.224]  

PLAY RECAP*******************************************************************************************************************    
172.31.16.224              : ok=1    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0        

[ansadmin@ansible-server docker]$ **docker login -u debsss**   
Password:     
WARNING! Your password will be stored unencrypted in /home/ansadmin/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store      

Login Succeeded       

GOT ERROR WHILE RUNNING LAYBOOK . FOUND docker disabled. HENCE STARTED docker service and executed the play  

[ansadmin@ansible-server docker]$ **ansible-playbook java-hello-world-ci.yml**     

PLAY [ansible] ***************************************************************************************************************        

TASK [Gathering Facts] *******************************************************************************************************
[WARNING]: Platform linux on host 172.31.16.224 is using the discovered Python interpreter at /usr/bin/python, but future        
installation of another Python interpreter could change this. See
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.         
ok: [172.31.16.224]      

TASK [create docker image] ***************************************************************************************************     
fatal: [172.31.16.224]: FAILED! => {"changed": true, "cmd": ["docker", "build", "-t", "java-hello-word:latest", "."], "delta": "0:00:00.135632", "end": "2024-11-04 08:18:21.643520", "msg": "non-zero return code", "rc": 1, "start": "2024-11-04 08:18:21.507888", "stderr": "ERROR: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?", "stderr_lines": ["ERROR: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"], "stdout": "", "stdout_lines": []}      

PLAY RECAP *******************************************************************************************************************      
172.31.16.224              : ok=1    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0         

[ansadmin@ansible-server docker]$ **sudo systemctl status docker**       
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://docs.docker.com   

[ansadmin@ansible-server docker]$ **sudo systemctl start docker**      
[ansadmin@ansible-server docker]$ **sudo systemctl status docker**    
● docker.service - Docker Application Container Engine        
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)    
   Active: **active** (running) since Mon 2024-11-04 08:19:16 UTC; 8s ago    
     Docs: https://docs.docker.com        
  Process: 6889 ExecStartPre=/usr/libexec/docker/docker-setup-runtimes.sh (code=exited, status=0/SUCCESS)
  Process: 6877 ExecStartPre=/bin/mkdir -p /run/docker (code=exited, status=0/SUCCESS)  
 Main PID: 6896 (dockerd)
    Tasks: 7
   Memory: 110.6M
   CGroup: /system.slice/docker.service
           └─6896 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --default-ulimit nofile=32768:6553...   

Nov 04 08:19:06 ansible-server systemd[1]: Starting Docker Application Container Engine...    
Nov 04 08:19:13 ansible-server dockerd[6896]: time="2024-11-04T08:19:13.673028907Z" level=info msg="Starting up"
Nov 04 08:19:15 ansible-server dockerd[6896]: time="2024-11-04T08:19:15.163231479Z" level=info msg="[graphdriver] usin...lay2"
Nov 04 08:19:15 ansible-server dockerd[6896]: time="2024-11-04T08:19:15.179329359Z" level=info msg="Loading containers...art."
Nov 04 08:19:16 ansible-server dockerd[6896]: time="2024-11-04T08:19:16.122876219Z" level=info msg="Default bridge (do...ress"
Nov 04 08:19:16 ansible-server dockerd[6896]: time="2024-11-04T08:19:16.185281828Z" level=info msg="Loading containers: done."
Nov 04 08:19:16 ansible-server dockerd[6896]: time="2024-11-04T08:19:16.298709068Z" level=info msg="Docker daemon" com...5.0.6
Nov 04 08:19:16 ansible-server dockerd[6896]: time="2024-11-04T08:19:16.319698581Z" level=info msg="Daemon has complet...tion"
Nov 04 08:19:16 ansible-server dockerd[6896]: time="2024-11-04T08:19:16.521578861Z" level=info msg="API listen on /run...sock"    
Nov 04 08:19:16 ansible-server systemd[1]: Started Docker Application Container Engine.      
Hint: Some lines were ellipsized, use -l to show in full.        
[ansadmin@ansible-server docker]$ **sudo usermod -aG docker ansadmin**    
[ansadmin@ansible-server docker]$  
CHANGED Dockerfile  
[ansadmin@ansible-server docker]$ **cat Dockerfile**     
FROM tomcat:latest    
COPY ./*.war /usr/local/tomcat/webapps/java-hello-world.war   
[ansadmin@ansible-server docker]$ **ansible-playbook java-hello-world-ci.yml**  

PLAY [ansible] ***************************************************************************************************************       

TASK [Gathering Facts] *******************************************************************************************************
[WARNING]: Platform linux on host 172.31.16.224 is using the discovered Python interpreter at /usr/bin/python, but future
installation of another Python interpreter could change this. See
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html for more information.      
ok: [172.31.16.224]    

TASK [create docker image] ***************************************************************************************************
changed: [172.31.16.224]      

TASK [create tag to push image to dockerhub] *********************************************************************************
changed: [172.31.16.224]      
  
TASK [push to docker hub] ****************************************************************************************************
changed: [172.31.16.224]       

PLAY RECAP *******************************************************************************************************************    
172.31.16.224              : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0     

39. CREATE CONTINOUS INTEGRATION JOB  
Same config of maven project job with 
create maven_webapp_ci job as maven project - configure -> git scm -> url of git https://github.com/DebaratiBiswas/maven_webapp.git -> ./main -> pom.xml in root with clean install as cmd  configure -  postbuild action - send build artifact over ssh - transfer set -  source fles - target/*.war - remove prefix - target - Remote directory //opt//docker -> exec command in post build action as ansible-playbook /opt/docker/java-hello-world-ci.yml     
execute and check dockerhub for latest pushed image  
1:48

40. PROVISION EKS SERVER WITH TERRAFORM  
Here is the output with double spaces and an extra line break after each line:

```bash
[ec2-user@terraform-server ~]$ ls  

ansible  jenkins  

[ec2-user@terraform-server ~]$ cp -r ansible eks-server  

[ec2-user@terraform-server ~]$ ls  

ansible  eks-server  jenkins  

[ec2-user@terraform-server ~]$ cd eks-server/  

[ec2-user@terraform-server eks-server]$ ls  

data.tf  main.tf  provider.tf  security.tf  variables.tf  

[ec2-user@terraform-server eks-server]$ ls -a  

.  ..  data.tf  main.tf  provider.tf  security.tf  .terraform  .terraform.lock.hcl  variables.tf  

[ec2-user@terraform-server eks-server]$ rm -rf .terraform .terraform.lock.hcl  

[ec2-user@terraform-server eks-server]$ ls -a  

.  ..  data.tf  main.tf  provider.tf  security.tf  variables.tf  

[ec2-user@terraform-server eks-server]$ vi main.tf  

[ec2-user@terraform-server eks-server]$ cat main.tf  

resource "aws_instance" "EKSServer" {  

  ami                    = data.aws_ami.amazonlinux2.id  

  instance_type          = var.my_instance_type  

  key_name               = var.my_key  

  vpc_security_group_ids = [aws_security_group.web-traffic.id]  

  tags = {  

    "Name" = "EKS-Server"  

  }  

}  

[ec2-user@terraform-server eks-server]$ vi provider.tf  

[ec2-user@terraform-server eks-server]$ cat provider.tf  

terraform {  

  required_version = "~> 1.0"  

  required_providers {  

    aws = {  

      source  = "hashicorp/aws"  

      version = "~> 4.0" # Optional but recommended in production  

    }  

  }  

  backend "s3" {  

    bucket = "project-register-terraform-server"  

    key    = "eks-server/terraform.tfstate"  

    region = "us-east-2"  

  }  

}  

provider "aws" {  

  region = "us-east-2"  

}  

[ec2-user@terraform-server eks-server]$ vi security.tf  

[ec2-user@terraform-server eks-server]$ cat security.tf  

# Create Security Group - SSH Traffic and other ports  

resource "aws_security_group" "web-traffic" {  

  name = "My_Security_Group3"  

  ingress {  

    from_port   = 0  

    to_port     = 0  

    protocol    = "-1"  

    cidr_blocks = ["0.0.0.0/0"]  

  }  

  egress {  

    from_port   = 0  

    to_port     = 0  

    protocol    = "-1"  

    cidr_blocks = ["0.0.0.0/0"]  

  }  

  tags = {  

    "Name" = "My_SG3"  

  }  

}  

[ec2-user@terraform-server eks-server]$ vi variables.tf  

[ec2-user@terraform-server eks-server]$ cat variables.tf  

variable "region" {  

  type    = string  

  default = "us-east-2"  

}  

variable "my_instance_type" {  

  type    = string  

  default = "t2.micro"  

}  

variable "my_key" {  

  description = "AWS EC2 Key pair that needs to be associated with EC2 Instance"  

  type        = string  

  default     = "terraform-server1-key"  

}  

variable "ingressrules" {  

  type    = list(number)  

  default = [22, 80, 443, 8080, 8090, 9000, 8081, 2479]  

}  

variable "egressrules" {  

  type    = list(number)  

  default = [25, 80, 443, 8080, 8090, 3306, 53]  

}  

41. Here is the output with double spaces and an extra line break after each line:

```bash
[ec2-user@terraform-server eks-server]$ terraform init  

Initializing the backend...  

Successfully configured the backend "s3"! Terraform will automatically  
use this backend unless the backend configuration changes.  

Initializing provider plugins...  

- Finding hashicorp/aws versions matching "~> 4.0"...  

- Installing hashicorp/aws v4.67.0...  

- Installed hashicorp/aws v4.67.0 (signed by HashiCorp)  

Terraform has created a lock file .terraform.lock.hcl to record the provider  
selections it made above. Include this file in your version control repository  
so that Terraform can guarantee to make the same selections by default when  
you run "terraform init" in the future.  

Terraform has been successfully initialized!  

You may now begin working with Terraform. Try running "terraform plan" to see  
any changes that are required for your infrastructure. All Terraform commands  
should now work.  

If you ever set or change modules or backend configuration for Terraform,  
rerun this command to reinitialize your working directory. If you forget, other  
commands will detect it and remind you to do so if necessary.  

[ec2-user@terraform-server eks-server]$ terraform fmt  

[ec2-user@terraform-server eks-server]$ terraform validate  

Success! The configuration is valid.  

[ec2-user@terraform-server eks-server]$ terraform plan  

data.aws_ami.amazonlinux2: Reading...  

data.aws_ami.amazonlinux2: Read complete after 1s [id=ami-0fa15516db7b1dc62]  

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the  
following symbols:  
  + create  

Terraform will perform the following actions:  

  # aws_instance.EKSServer will be created  
  + resource "aws_instance" "EKSServer" {  
      + ami                                  = "ami-0fa15516db7b1dc62"  
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
          + "Name" = "EKS-Server"  
        }  
      + tags_all                             = {  
          + "Name" = "EKS-Server"  
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
      + name                   = "My_Security_Group3"  
      + name_prefix            = (known after apply)  
      + owner_id               = (known after apply)  
      + revoke_rules_on_delete = false  
      + tags                   = {  
          + "Name" = "My_SG3"  
        }  
      + tags_all               = {  
          + "Name" = "My_SG3"  
        }  
      + vpc_id                 = (known after apply)  
    }  

Plan: 2 to add, 0 to change, 0 to destroy.  

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────  

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run  
"terraform apply" now.  

[ec2-user@terraform-server eks-server]$ terraform apply  

data.aws_ami.amazonlinux2: Reading...  

data.aws_ami.amazonlinux2: Read complete after 1s [id=ami-0fa15516db7b1dc62]  

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the  
following symbols:  
  + create  

Terraform will perform the following actions:  

  # aws_instance.EKSServer will be created  
  + resource "aws_instance" "EKSServer" {  
      + ami                                  = "ami-0fa15516db7b1dc62"  
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
      + security_groups                      =

 (known after apply)  
      + source_dest_check                    = true  
      + subnet_id                            = (known after apply)  
      + tags                                 = {  
          + "Name" = "EKS-Server"  
        }  
      + tags_all                             = {  
          + "Name" = "EKS-Server"  
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
      + name                   = "My_Security_Group3"  
      + name_prefix            = (known after apply)  
      + owner_id               = (known after apply)  
      + revoke_rules_on_delete = false  
      + tags                   = {  
          + "Name" = "My_SG3"  
        }  
      + tags_all               = {  
          + "Name" = "My_SG3"  
        }  
      + vpc_id                 = (known after apply)  
    }  

Plan: 2 to add, 0 to change, 0 to destroy.  

Do you want to perform these actions?  
  Terraform will perform the actions described above.  
  Only 'yes' will be accepted to approve.  

  Enter a value: yes  

aws_security_group.web-traffic: Creating...  

aws_security_group.web-traffic: Creation complete after 2s [id=sg-0feed4719b49d2832]  

aws_instance.EKSServer: Creating...  

aws_instance.EKSServer: Still creating... [10s elapsed]  

aws_instance.EKSServer: Creation complete after 12s [id=i-0d5bd172cf0964e63]  

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.  

[ec2-user@terraform-server eks-server]$  

42. ssh to eks -server using ip address   
Change hostname from ip-172-31-23-205.us-east-2.compute.internal to EKS-Server  
[ec2-user@ip-172-31-23-205 ~]$ sudo vi /etc/hostname  
[ec2-user@ip-172-31-23-205 ~]$ cat /etc/hostname  
EKS-Server  
[ec2-user@ip-172-31-23-205 ~]$ sudo init 6  <---- reboot restart command   
43. PROVISION EKS CLUSTER USING EKSCTL   
 

**sudo su** 	Switches to root user without a login shell	Retains the current user's environment (working directory, environment variables, etc.)   
**sudo su -** 	Switches to root user with a login shell (- simulates a full login)	Resets the environment to that of root (changes working directory to /root, sources root's login configurations)  
INstall kubectl  
give exec permission  
[root@EKS-Server ~]# curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl   
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 46.9M  100 46.9M    0     0  10.7M      0  0:00:04  0:00:04 --:--:-- 10.7M  
[root@EKS-Server ~]# chmod +x ./kubectl  
[root@EKS-Server ~]# ls    
kubectl  
[root@EKS-Server ~]# ll  
total 48092  
-rwxr-xr-x 1 root root 49246208 Nov  7 09:48 kubectl  
[root@EKS-Server ~]# mv kubectl /bin   
[root@EKS-Server ~]# ls /bin | grep kubectl    
kubectl      
[root@EKS-Server ~]# kubectl version         
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.      
Client Version: version.Info{Major:"1", Minor:"27+", GitVersion:"v1.27.1-eks-2f008fe", GitCommit:"abfec7d7e55d56346a5259c9379dea9f56ba2926", GitTreeState:"clean", BuildDate:"2023-04-14T20:43:13Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
Kustomize Version: v5.0.1    
The connection to the server localhost:8080 was refused - did you specify the right host or port?    
INSTALL EKSCTL  
[root@EKS-Server ~]# **curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp**       
[root@EKS-Server ~]# **cd /tmp**      
[root@EKS-Server tmp]# **sudo mv /tmp/eksctl /bin**    
[root@EKS-Server tmp]# **eksctl version**          
0.194.0      
[root@EKS-Server tmp]#    


















