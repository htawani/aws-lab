**IAM Roles, STS, and Temporary Credentials**   
.  
**Goal:** Understand how an AWS service or workload gets temporary access to another AWS service without storing permanent credentials.  
.  
.  
**STS -->** AWS Security Token Service (STS) creates a temporary role session.   
.  
.  
.  
**Assignment: EC2 IAM Role and S3 policy to list and read (Least privilege)**    
  
1\. Create S3 bucket  
2\. Create IAM Policy – S3 list and read  
3\. Create IAM Role – EC2  attach S3 policy  
4\. Create EC2 instance  
5\. Connect EC2 instance and validate # aws s3 ls s3://**bucket-name**/  
6\. Assign IAM Role to the EC2 instance # validate again  

**PRACTICAL:**  
.  
**1. Create S3 bucket**   
Select Region:  
Create bucket  
Bucket type: General purpose  
Bucket namespace  
Bucket name: ht313-demo-iam-s3  
Object ownership: ACLs disabled  
Block public access setting for this bucket: Block all public access  
Bucket versioning: Disabled  
Create bucket  
Upload > Add files > hello.txt > Upload  
.  
.  
<img width="940" height="284" alt="image" src="https://github.com/user-attachments/assets/0e1a0725-8e12-4252-98b4-7aa1f8388682" />

.  
.  
**2. Create IAM Policy**  
Policies  
Create policy  
Select a service: S3  
Actions allowed: List > All and Read > All  
Resources: All > Next  
Policy name: ht313-s3-readonly > Create policy  

**Read Permission:** Policy is EC2 instance will only list out the AWS objects and get the object (download), EC2 instance will not allow to upload any file using this policy  
.  
.  

<img width="940" height="355" alt="image" src="https://github.com/user-attachments/assets/cddc2969-86f3-4500-8c76-858f1b0122ee" />
  
.  
.  
**3. Create IAM Role**  
Role > Create role  
Select trusted entity: AWS service  
Use case: EC2 > Next  
Filter by type: Customer managed  
Select policy (attach the policy): ht313-s3-readonly > Next  
Role name: ht313-ec2-s3-polr > Create role  
.  
**Trust relationships**  
EC2 instance will only assume the role  

## Trust policy is an Authentication the EC2 instance  
## Permission is your Authorization to access the S3 bucket  
.  
.  
<img width="940" height="416" alt="image" src="https://github.com/user-attachments/assets/1f8702ce-7dab-4680-9e37-2a5908d63021" />

.  

<img width="940" height="510" alt="image" src="https://github.com/user-attachments/assets/932e0aa8-cf66-4d39-91d7-3e024cd940d9" />

.  
.  



**4. Create EC2 instance**  
Instances > Launch instance  
Name: demo-ec201  
Application and OS Images: Quick Start Amazon Linux  
Instance type: t3.micro  
Key pair: **key-pair-name**  
Network settings:  
Network: vpc-defalut  
Subnet: No preference-default  
Auto-assig public IP: Enabled  
Firewall (security group): Create security group and Allow SSH traffic from – Anywhere  
Configure storage: 8-gp3 (default) > Launch instance  
.  

5\. **Connect ec2 instance and validate s3 access**    
student@srv-ubuntu:data$ **ssh -i master.pem ec2-user@3.110.212.224**  
\[ec2-user@ip-172-31-36-245 \~]$   
\[ec2-user@ip-172-31-36-245 \~]$ **sudo su -**  
\[root@ip-172-31-36-245 \~]**#**  
\[root@ip-172-31-36-245 \~]# **aws s3 ls s3://ht313-demo-iam-s3**  
Unable to locate credentials. You can configure credentials by running "aws login".  
\[root@ip-172-31-36-245 \~]# 
\[root@ip-172-31-36-245 \~]# **aws sts get-caller-identity**  
Unable to locate credentials. You can configure credentials by running "aws login".  
\[root@ip-172-31-36-245 \~]#  
.  

6\. **Now, assign IAM Role to the ec2 instance and validate s3 read and write access**  
Select instance > Actions > Security > Modify IAM role  
IAM role: attach existing role: ec2-instance-profile-role > Update IAM role  


<img width="940" height="239" alt="image" src="https://github.com/user-attachments/assets/1758e522-555d-45b2-8014-7b20aa28cde3" />

.  
.  
\[root@ip-172-31-36-245 \~]# **aws sts get-caller-identity**  
{
&#x20;   "UserId": "AROATBRPP7LKQQ6643BP5:i-04b28eac3590ab2e7",
&#x20;   "Account": "209479269077",
&#x20;   "Arn": "arn:aws:sts::209479269077:assumed-role/ht313-ec2-s3-polr/i-04b28eac3590ab2e7"
}
\[root@ip-172-31-36-245 \~]#   
\[root@ip-172-31-36-245 \~]# **aws s3 ls**   
2026-07-18 17:31:24 ht313-demo-iam-s3  
\[root@ip-172-31-36-245 \~]# **aws s3 ls s3://ht313-demo-iam-s3**  
2026-07-18 17:32:57         64 hello.txt  
\[root@ip-172-31-36-245 \~]# **cat hello.txt**  
cat: hello.txt: No such file or directory  
\[root@ip-172-31-36-245 \~]#   
\[root@ip-172-31-36-245 \~]# **aws s3 cp s3://ht313-demo-iam-s3/hello.txt .**  
download: s3://ht313-demo-iam-s3/hello.txt to ./hello.txt         
\[root@ip-172-31-36-245 \~]#   
\[root@ip-172-31-36-245 \~]# **cat hello.txt **  
Demo file - Accessing S3 bucket using EC2 instance with IAM Role\[root@ip-172-31-36-245 \~]#   
\[root@ip-172-31-36-245 \~]#   
\[root@ip-172-31-36-245 \~]# **echo "This upload should fail" > upload.txt**  
\[root@ip-172-31-36-245 \~]# **cat upload.txt **  
This upload should fail  
\[root@ip-172-31-36-245 \~]#   
\[root@ip-172-31-36-245 \~]# **aws s3 cp upload.txt s3://ht313-demo-iam-s3/**  
**upload failed:** ./upload.txt to s3://ht313-demo-iam-s3/upload.txt An error occurred **(AccessDenied)** **when calling the PutObject operation:** User: arn:aws:sts::209479269077:**assumed-role/ht313-ec2-s3-polr**/i-04b28eac3590ab2e7 **is not authorized to perform: s3:PutObject on resource:** "arn:aws:s3:::ht313-demo-iam-s3/upload.txt" because **no identity-based policy allows the s3:PutObject action**  
\[root@ip-172-31-36-245 \~]#  





