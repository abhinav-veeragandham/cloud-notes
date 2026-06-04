# AWS IAM — Identity and Access Management

## The one-line mental model
Every AWS action = WHO (identity) doing WHAT (policy) on WHICH resource.
IAM controls all three.

---

## Core concepts

```
IAM User     → represents a person or application
               has long-term credentials (password, access keys)

IAM Group    → collection of users
               attach policies to group, all users inherit them
               users can belong to multiple groups

IAM Role     → temporary permissions for AWS services or users
               no long-term credentials — uses temporary tokens
               EC2 instance role, Lambda role, cross-account access

IAM Policy   → JSON document defining what is allowed or denied
               attached to users, groups, or roles
```

---

## Golden rules — memorize these for interviews

```
1. Never use root account day-to-day
   → create IAM admin user immediately after account creation

2. Always least privilege
   → only grant permissions actually needed, nothing more

3. Enable MFA on every account
   → root account especially, all IAM users ideally

4. Never put access keys in code
   → use IAM roles instead — roles use temporary credentials

5. Use groups to assign permissions
   → never attach policies directly to individual users

6. Rotate access keys regularly
   → disable unused keys immediately
```

---

## Hands-on checklist

### Level 1 — CCP level (you should know these cold)
- [ ] Create an IAM user with console access
- [ ] Create an IAM user with programmatic access (CLI)
- [ ] Create an IAM group and add users to it
- [ ] Attach a managed policy to a group (e.g. AmazonS3ReadOnlyAccess)
- [ ] Enable MFA on an IAM user
- [ ] Generate and download access keys for CLI access
- [ ] Set a password policy for the account
- [ ] View IAM credential report
- [ ] Sign in as IAM user (not root)
- [ ] Test permissions — try an action the user shouldn't be able to do

### Level 2 — SAA level (learn in Weeks 5-8)
- [ ] Create a custom IAM policy from scratch (JSON)
- [ ] Create an IAM role for EC2 instance
- [ ] Attach an IAM role to a running EC2 instance
- [ ] Create a role for Lambda function
- [ ] Set up cross-account access with IAM role
- [ ] Use IAM policy simulator to test permissions
- [ ] Create and test a Service Control Policy (SCP)
- [ ] Set up IAM Identity Center (SSO) basics

### Level 3 — Job level (learn on the job)
- [ ] Permission boundaries
- [ ] Attribute-based access control (ABAC)
- [ ] IAM Access Analyzer
- [ ] Advanced SCP strategies
- [ ] Federation with external identity providers

---

## IAM Policy structure — understand this cold

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

```
Version    → always "2012-10-17" (policy language version)
Statement  → list of permission rules
Effect     → Allow or Deny (Deny always wins)
Action     → what API calls are allowed/denied
Resource   → which AWS resources this applies to
Principal  → who this applies to (used in resource policies)
Condition  → optional conditions (MFA required, IP range, etc.)
```

---

## Policy types — know the difference

```
AWS Managed Policy    → created by AWS, you attach it
                        e.g. AmazonS3FullAccess, AdministratorAccess
                        cannot be edited

Customer Managed      → you create and manage it
                        reusable across multiple identities
                        best practice for custom permissions

Inline Policy         → attached directly to one user/group/role
                        not reusable
                        avoid unless specific reason

Resource Policy       → attached to a resource (S3 bucket, SQS queue)
                        controls who can access that specific resource
                        e.g. S3 bucket policy
```

---

## IAM Roles — the most important concept for cloud engineers

```
Why roles instead of users for AWS services:
- EC2 instance needs to access S3
- BAD:  put access keys in the code (rotates, leaks, hard to manage)
- GOOD: attach IAM role to EC2 — temporary credentials auto-rotate

Common role use cases:
EC2 → S3          attach AmazonS3ReadOnlyAccess role to EC2
Lambda → DynamoDB  attach AmazonDynamoDBFullAccess role to Lambda
ECS task → ECR    attach AmazonECRReadOnlyAccess role to task
Cross-account     Role A in account 1, assumed by user in account 2
```

---

## ARN format — Amazon Resource Name

```
arn:aws:iam::123456789012:user/john
arn:aws:s3:::my-bucket
arn:aws:ec2:us-east-1:123456789012:instance/i-1234567890abcdef0

Format: arn:partition:service:region:account-id:resource
```

---

## Common interview questions

**Q: What is the difference between an IAM user and an IAM role?**
```
User = permanent identity with long-term credentials (password/access keys)
       represents a specific person or application

Role = temporary identity with short-term credentials (auto-expire)
       assumed by services, applications, or users when needed
       best practice for AWS service-to-service access
```

**Q: What happens if a user has both Allow and Deny for the same action?**
```
Deny always wins — explicit Deny overrides any Allow
Default is Deny — everything denied unless explicitly allowed
```

**Q: What is the principle of least privilege?**
```
Grant only the minimum permissions needed to perform a task.
Start with zero permissions, add only what is required.
Review and remove unused permissions regularly.
```

**Q: How do you give an EC2 instance access to S3 without access keys?**
```
Create an IAM role with S3 permissions
Attach the role to the EC2 instance
The instance automatically gets temporary credentials via metadata service
Application uses AWS SDK — credentials handled automatically
```

**Q: What is MFA and why is it important?**
```
Multi-Factor Authentication — requires second factor beyond password
Critical for root account and all privileged users
Types: virtual MFA (Authenticator app), hardware MFA, U2F key
Can enforce MFA via IAM policy condition
```

---

## IAM best practices checklist — for your AWS account right now

```
Root account:
- [ ] Enable MFA on root account
- [ ] Delete root access keys
- [ ] Never use root for daily tasks

IAM setup:
- [ ] Create admin IAM user for daily use
- [ ] Enable MFA on admin user
- [ ] Use groups for permissions, not individual users
- [ ] Apply least privilege to all users

Access keys:
- [ ] Never put access keys in code
- [ ] Never commit access keys to GitHub
- [ ] Rotate access keys every 90 days
- [ ] Delete unused access keys immediately

Monitoring:
- [ ] Enable CloudTrail (logs all API calls)
- [ ] Review IAM credential report monthly
- [ ] Set up billing alarm
```

---

## CLI commands for IAM

```bash
# Configure CLI with access keys
aws configure

# Verify who you are
aws sts get-caller-identity

# List users
aws iam list-users

# Create user
aws iam create-user --user-name john

# Attach policy to user
aws iam attach-user-policy \
  --user-name john \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# List roles
aws iam list-roles

# Get current user's permissions
aws iam get-user
```

---

## IAM → connects to everything else

```
EC2         → instance roles for accessing other services
S3          → bucket policies + IAM policies combined
Lambda      → execution role defines what function can access
ECS/EKS     → task roles for container permissions
Terraform   → needs IAM user or role with right permissions
CI/CD       → GitHub Actions uses OIDC role (no access keys)
CloudTrail  → logs every IAM action for audit
```
