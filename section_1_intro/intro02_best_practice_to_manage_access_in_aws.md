
# Evolution of access management:
- you add permissions to each user individually
- you create IAM roles with a list of permissions and let the users assume the IAM roles.

# Best practice for access management
IAM Identity Center (SSO) + Permission Sets — Detailed Guide

This document explains **Option B (Best Practice): IAM Identity Center (SSO) + Permission Sets** in detail, with step-by-step guidance and SageMaker-specific context.

---

## Big Picture Architecture

```
Human User
↓
Identity Provider (IAM Identity Center or External IdP)
↓
Group
↓
Permission Set
↓
IAM Role (auto-created by AWS)
↓
AWS Account
↓
AWS Services (SageMaker, S3, etc.)
```

Key principle:
> Humans never get IAM Users or long-term credentials.  
> Access is always temporary and role-based.

---

## Step 1: Enable IAM Identity Center

### What it is
IAM Identity Center (formerly AWS SSO) is a centralized service for:
- User and group management
- AWS account access
- Temporary credential issuance (STS)

### How to enable
1. Open **AWS Console → IAM Identity Center**
2. Click **Enable**
3. Choose identity source:
   - Built-in Identity Store (recommended to start)
   - External IdP (Azure AD, Okta, Google, etc.)

### What AWS sets up automatically
- Identity Store
- SSO instance
- Secure STS integration

No IAM Users are created.

---

## Step 2: Create Users

### What an SSO user represents
- A human identity
- No AWS permissions by default
- Authenticates via SSO portal

### How to create users
1. IAM Identity Center → **Users**
2. Create user
3. Provide:
   - Username
   - Email
   - First/Last name
4. User receives invitation email

### Why this is better than IAM Users
- No long-term access keys
- Central MFA and password control
- Easier onboarding and offboarding: just by adding and removing from groups.

---

## Step 3: Create Groups (Job-Based)

### Why groups are critical
Groups make access **repeatable and scalable**.
Permissions are never assigned directly to users.

### Recommended group design
Create groups by **job function**, not by service.

Examples:
- `DataScientists`
- `MLEngineers`
- `SageMakerAdmins`
- `ReadOnlyUsers`

### How to create groups
1. IAM Identity Center → **Groups**
2. Create group
3. Add users

Groups become your main access control mechanism.

---

## Step 4: Create Permission Sets

### What is a Permission Set?
A **Permission Set** is a reusable permission template that AWS uses to automatically create IAM roles in target accounts.

Think of it as:
> A role blueprint

### Components of a Permission Set
- AWS managed policies
- Customer-managed policies
- Inline policies (optional)
- Session duration
- Permissions boundary (optional)

### Example Permission Sets
**Data Scientist**
- AmazonSageMakerFullAccess
- AmazonS3ReadOnlyAccess
- CloudWatchLogsReadOnlyAccess

**ML Engineer**
- SageMaker full access
- ECR access
- S3 read/write

**Admin**
- AdministratorAccess

### How to create a Permission Set
1. IAM Identity Center → **Permission Sets**
2. Create permission set
3. Attach policies
4. Set session duration (e.g., 8 hours)
5. Add description

---

## Step 5: Assign Permission Sets to Groups and Accounts

### How access is granted
You assign:
- Group
- Permission Set
- AWS Account

Example:
```
Group: DataScientists
Permission Set: PS-DataScientist
Account: ML-Dev
```

AWS automatically:
- Creates an IAM role
- Configures trust policy
- Attaches permissions

**>> when you assign permission sets to groups, automatically AWS creates IAM roles**<br>
**>> each time, a user assume a IAM role to do his job with a list of permissions**

Role name example:
```
AWSReservedSSO_PS-DataScientist_abcd1234
```

---

## Step 6: User Login Flow

1. User opens SSO portal URL
2. Authenticates (MFA enforced)
3. Selects:
   - AWS account
   - Role (Permission Set)
4. Receives temporary credentials
5. Accesses AWS Console, CLI, or SDK

No access keys are needed.

---

## Step 7: Using SageMaker with Identity Center

### Access flow
```
SSO User
↓
Permission Set Role
↓
SageMaker Access
↓
SageMaker Domain
↓
User Profile
↓
Execution Role
```

Key separation:
- Permission Set role → what the user can do
- Execution role → what SageMaker workloads can access

### Domain setup
- Choose IAM Identity Center authentication
- Map SSO users/groups to SageMaker user profiles
- Assign execution roles per profile

---

## Step 8: Auditing and Security

### Security advantages
- Temporary credentials only
- Centralized access control
- Full CloudTrail logging
- Instant access revocation

Offboarding is as simple as removing a user from a group.

---

## Step 9: Repeatable Onboarding and Offboarding

### Onboarding
1. Create SSO user
2. Add user to group
3. Done

### Offboarding
1. Remove user from group
2. Access immediately revoked

No IAM cleanup required.

---

## Step 10: Naming Conventions (Recommended)

```
Groups:        DataScientists, MLEngineers
PermissionSets: PS-DataScientist, PS-ML-Engineer
```

Consistent naming makes IAM scalable and understandable.

---

## Step 11: Common Mistakes to Avoid

- Assigning permissions directly to users
- Mixing admin and execution permissions
- Overly long session durations
- One permission set per user

---

## Summary

IAM Identity Center + Permission Sets provide the most secure, scalable, and maintainable way to manage AWS and SageMaker access.  
Permissions are defined once, reused everywhere, and enforced through temporary, auditable roles.