## Practical: Least privilege - EC2 IAM Role and S3 policy to list and read 

1. Create S3 bucket  
2. Create IAM Policy – S3 list and read  
3. Create IAM Role – EC2 ß attach S3 policy  
4. Create EC2 instance  
5. Connect EC2 instance and validate  
   `aws s3 ls s3://bucket-name/`  
7. Assign IAM Role to the EC2 instance
   --> validate again

