## IAM — the most important AWS service
Every AWS action = who (identity) doing what (policy) on which resource

Identity types:
- IAM User     → person or application
- IAM Group    → collection of users
- IAM Role     → temporary permissions for services/users
- IAM Policy   → JSON document defining permissions

Golden rules:
- Never use root account day-to-day
- Always least privilege — only permissions needed
- Enable MFA on every account
- Never put access keys in code — use roles instead
