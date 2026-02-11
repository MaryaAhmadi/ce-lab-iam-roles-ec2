# Lab M2.05 - IAM Roles for EC2

**Repository:** [https://github.com/MaryaAhmadi/ce-lab-iam-roles-ec2.git](https://github.com/cloud-engineering-bootcamp/ce-lab-iam-roles-ec2)

**Activity Type:** Individual  
**Estimated Time:** 30-45 minutes

## Learning Objectives

- [ ] Create IAM role for EC2 instances
- [ ] Attach policies to IAM roles
- [ ] Assign IAM role to running EC2 instance
- [ ] Test AWS API access using role
- [ ] Verify security without hardcoded credentials

## Your Task

Create and assign an IAM role that allows your EC2 instance to:
1. Read from specific S3 bucket
2. Write logs to CloudWatch
3. No other permissions (least privilege)

**Success:** Access S3 from EC2 without aws configure

## Step-by-Step Process

###  Create IAM Role
- Name: `ec2-s3-cloudwatch-role`
- Trusted entity: EC2
- Attach policies:
  - **Managed policy:** `CloudWatchAgentServerPolicy`
  - **Custom S3 policy:** `s3-cloudwatch-policy.json`  
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [{
        "Effect": "Allow",
        "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
        "Resource": [
          "arn:aws:s3:::maryam-ahmadi-bootcamp-inventory/*",
          "arn:aws:s3:::maryam-ahmadi-bootcamp-inventory"
        ]
      }]
    }
    ```

###  Attach Role to EC2 Instance
- EC2 Console → Instance → Actions → Security → Modify IAM role  
- Attach `ec2-s3-cloudwatch-role` to instance `rup` (i-0f75c7fd2c3f4f7a9)

###  Test S3 Access from EC2
```bash
ssh -i ~/Downloads/ce-lab-key.pem ec2-user@<EC2_PUBLIC_IP>
aws sts get-caller-identity
aws s3 ls s3://maryam-ahmadi-bootcamp-inventory/
echo "lab test from EC2 role" > test.txt
aws s3 cp test.txt s3://maryam-ahmadi-bootcamp-inventory/



Confirmed S3 access works without aws configure

File uploaded successfully



###  Test CloudWatch Access from EC2

aws logs describe-log-groups
aws logs create-log-group --log-group-name /lab-m2-test
aws logs create-log-stream --log-group-name /lab-m2-test --log-stream-name ec2-test-stream
aws logs put-log-events \
  --log-group-name /lab-m2-test \
  --log-stream-name ec2-test-stream \
  --log-events timestamp=$(date +%s%3N),message="CloudWatch test from EC2 role"



Verified CloudWatchAgentServerPolicy allows logging

Test log sent to /lab-m2-test log group




# Policy Explanation

- S3: Only GetObject, PutObject, ListBucket on the specific bucket
- CloudWatch: Managed policy CloudWatchAgentServerPolicy for log writing
- Principle: Least privilege — no wildcard or extra permissions




# Security Best Practices

- No access keys stored on instance
- Role used for temporary credentials
- Permissions limited to required actions only
- S3 and CloudWatch access scoped to specific resources





# Troubleshooting

- If aws s3 commands fail, ensure the role is attached to EC2
- CloudWatch messages may take a few seconds to appear in Console; verify using CLI output if necessary




