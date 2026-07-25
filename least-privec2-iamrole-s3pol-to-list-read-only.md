## Practical: Least privilege - EC2 IAM Role and S3 policy to list and read 

1. Create S3 bucket  
2. Create IAM Policy – **S3 list and read**  
3. Create IAM Role – **EC2 ß attach S3 policy**  
4. Create EC2 instance  
5. Connect EC2 instance and validate  
   `aws s3 ls s3://bucket-name/`  
7. Assign IAM Role to the EC2 instance
   --> validate again
<br>

**1. Create S3 bucket**  
Select Region:  
Create bucket >  
Bucket type: **General purpose**  
Bucket namespace:   
Bucket name: **s3-list-read-bkt**   
Object ownership: ACLs disabled   
Block public access setting for this bucket: Block all public access   
Bucket versioning: Disabled  
Create bucket  

Upload > Add files > hello.txt > Upload  
<br>
<img width="979" height="380" alt="image" src="https://github.com/user-attachments/assets/786d8e6c-3f70-4e17-b791-e1fdbe7e388a" />
<br>
<br>
<br>
**2. Create IAM Policy**  
Policies  
Create policy  
Select a service: **S3**  
Actions allowed: List > All **and** Read > All  
Resources: All > Next  
Policy name: **s3-list-read-pol** > Create policy  
<br>
<br>
<img width="941" height="432" alt="image" src="https://github.com/user-attachments/assets/413a685b-1efa-49a7-b08d-e2891b7817ec" />


<br>
<br>

**Read Permission:**  
Policy is ec2 instance will only list out the AWS objects and get the object (download), ec2 instance will not allow to upload any file using this policy
<br>

**3. Create IAM Role**  
Role > Create role  
Select trusted entity: **AWS service**  
Service or Use case: **EC2** > Next  
Filter by type: Select **Customer managed**  
Select policy: **s3-list-read-pol** > Next   
Role name: **ec2-s3-role** > Create role  
<br>
<br>
<img width="917" height="347" alt="image" src="https://github.com/user-attachments/assets/1e58b69e-1df9-4cc7-8ba1-bd5956ef69ea" /><br>
### Trust Relationships
Menas, the EC2 instance will only assume the role<br>
--> **Trust policy is an Authentication the EC2 instance**  
--> **Permission is your Authorization to access the S3 bucket**  
<br>
**4. Create EC2 instance**  
Instances > Launch instance  
Name: **ec2-01**  
Application and OS Images: Quick Start **Amazon Linux**  
Instance type: **t3.micro**  
Key pair: Select key pair - **key.pem**  
**Network settings:**  
Network: **vpc-defalut**  
Subnet: **No preference-default**  
Auto-assig public IP: **Enabled**  
Firewall (security group): Create security group and Allow SSH traffic from – **Anywhere**  
Configure storage: 8-gp3 (default) > Launch instance  
<br>
**5. Connect EC2 Instance**  
[ec2-user@ip-172-31-2-218 ~]$ `sudo su -`  
[root@ip-172-31-2-218 ~]# `aws sts get-caller-identity`  

Unable to locate credentials. You can configure credentials by running "aws login".  
[root@ip-172-31-2-218 ~]# 
[root@ip-172-31-2-218 ~]# `aws s3 ls s3://s3-list-read-bkt`  

Unable to locate credentials. You can configure credentials by running "aws login".  
[root@ip-172-31-2-218 ~]#  

Now, we will attach the **IAM Role to the EC2 instance** --> **_to access S3 bucket_** and it should get the temporary credentials using **STS**   

EC2 >
Select instance > Actions >  
Security > Modify IAM role >   
IAM role: Attach existing role: **ec2-s3-role** > Update IAM role  

**NOTE** The Role is attached to the AWS service (EC2) not to the **user**  

Now, validate the list and read:  
<br>
[root@ip-172-31-2-218 ~]# `aws sts get-caller-identity`  
{  
    "UserId": "AROATBRPP7LKTGEGCAWK5:i-0a8befba44d4e028c",  
    "Account": "209479269077",  
    "Arn": "arn:aws:sts::209479269077:assumed-role/ec2-s3-role/i-0a8befba44d4e028c"  
}  
[root@ip-172-31-2-218 ~]#   
[root@ip-172-31-2-218 ~]# `aws s3 ls s3://s3-list-read-bkt`  
2026-07-25 09:09:38         57 hello.txt  
[root@ip-172-31-2-218 ~]#   
[root@ip-172-31-2-218 ~]# `aws s3 cp s3://s3-list-read-bkt/hello.txt .`  
download: s3://s3-list-read-bkt/hello.txt to ./hello.txt          
[root@ip-172-31-2-218 ~]# `ls`  
hello.txt  
[root@ip-172-31-2-218 ~]# `cat hello.txt`   
This is demo file  
List the S3 bucket  
Read the S3 bucket  
[root@ip-172-31-2-218 ~]#   
<br>
Now, create a sample file and try uploading to the s3 bucket and validate it  
<br>
[root@ip-172-31-2-218 ~]# `echo "This is sample upload file to s3 bucket with no write permission" > upload.txt`  
[root@ip-172-31-2-218 ~]# `ls`   
hello.txt  upload.txt  
[root@ip-172-31-2-218 ~]# `cat upload.txt`   
This is sample upload file to s3 bucket with no write permission  
[root@ip-172-31-2-218 ~]#   
[root@ip-172-31-2-218 ~]# `aws s3 cp upload.txt  s3://s3-list-read-bkt/`  
**upload failed:**__ ./upload.txt to s3://s3-list-read-bkt/upload.txt An error occurred **(AccessDenied) when calling the PutObject** operation: User: arn:aws:sts::209479269077:assumed-role/ec2-s3-role/i-0a8befba44d4e028c **is not authorized to perform: s3:PutObject** on resource: "arn:aws:s3:::s3-list-read-bkt/upload.txt" because no identity-based policy allows the s3:PutObject action
[root@ip-172-31-2-218 ~]#  
<br>






















