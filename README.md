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









