## Practical: Least privilege - EC2 IAM Role and S3 policy to list and read 

1. Create S3 bucket  
2. Create IAM Policy – S3 list and read  
3. Create IAM Role – EC2 ß attach S3 policy  
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
Bucket name: **s3-list-read-buck**   
Object ownership: ACLs disabled   
Block public access setting for this bucket: Block all public access   
Bucket versioning: Disabled  
Create bucket  

Upload > Add files > hello.txt > Upload
<br>

**2. Create IAM Policy**  
Policies  
Create policy  
Select a service: **S3**  
Actions allowed: List > All **and** Read > All  
Resources: All > Next  
Policy name: **s3-list-read-pol** > Create policy  
<br>
**Read Permission:**  
Policy is ec2 instance will only list out the AWS objects and get the object (download), ec2 instance will not allow to upload any file using this policy
<br>

**3. Create IAM Role**  
Role > Create role  
Select trusted entity: **AWS service**  
Use case: **EC2** > Next  
Filter by type: **Customer managed**  
Select policy (attach the policy): **s3-list-read-pol** > Next   
Role name: **ec2-s3-role** > Create role  
<br>
**Trust relationships**  
Menas, the EC2 instance will only assume the role<br>
--> **Trust policy is an Authentication the EC2 instance**  
--> **Permission is your Authorization to access the S3 bucket**  
<br>
