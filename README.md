# Automating Creation of IAM Resources Lab

## Introduction

This lab demonstrates how to automate IAM resource creation with AWS CloudFormation using a GitSync workflow. The stack creates IAM groups, policies, and users, and uses AWS Secrets Manager to generate and store a one-time temporary password that is assigned to all lab users.

## Lab Objectives

1. Build a GitSync-enabled CloudFormation template for IAM resources.
2. Assign permissions through IAM groups based on least privilege.
3. Validate access behavior by signing in as each IAM user.
4. Capture evidence (success/failure screenshots) for EC2 and S3 access tests.

## Repository Link

Replace this with your public or private GitHub repository URL before submission:

`https://github.com/<your-username>/<your-repo-name>`

## Repository Structure

```
.
|-- iam-lab-cloudformation.yaml
|-- deployment.yml
|-- README.md
`-- Screenshots/
```

## Task 1 Implementation

### CloudFormation Resources Created

The template in `iam-lab-cloudformation.yaml` creates:

1. `AWS::SecretsManager::Secret` (`UserTempPassword`)
2. `AWS::IAM::Group` (`S3Group`)
3. `AWS::IAM::Group` (`EC2Group`)
4. `AWS::IAM::Policy` (`S3Policy`) attached to `S3Group`
5. `AWS::IAM::Policy` (`EC2Policy`) attached to `EC2Group`
6. `AWS::IAM::User` (`ec2-user1`) with console login profile
7. `AWS::IAM::User` (`ec2-user2`) with console login profile
8. `AWS::IAM::User` (`s3-user`) with console login profile

### Permission Design

1. S3 group permissions:
   - `s3:ListAllMyBuckets`
2. EC2 group permissions:
   - `ec2:DescribeInstances`
   - `ec2:RunInstances`

Both `ec2-user1` and `ec2-user2` are assigned to the EC2 group, so `ec2-user2` is not prevented from launching EC2 instances.

### Password and Console Access Design

1. A temporary password is generated once in Secrets Manager.
2. All users reference the same generated password through a dynamic Secrets Manager reference.
3. Every user has `LoginProfile` configured for AWS Console sign-in.
4. `PasswordResetRequired: true` forces password change at first login.

### Requirement Coverage Matrix

| Lab Requirement                                                | Implementation Status                       |
| -------------------------------------------------------------- | ------------------------------------------- |
| One-time password auto-generated and stored in Secrets Manager | Implemented (`UserTempPassword`)            |
| S3 group can list S3 buckets                                   | Implemented (`S3Policy`)                    |
| EC2 group can list and create EC2 instances                    | Implemented (`EC2Policy`)                   |
| Create users: `ec2-user1`, `ec2-user2`, `s3-user`              | Implemented                                 |
| Assign `ec2-user1` and `ec2-user2` to EC2 group                | Implemented                                 |
| Assign `s3-user` to S3 group                                   | Implemented                                 |
| IAM users use temporary password                               | Implemented (Secrets Manager reference)     |
| IAM users must change password at first login                  | Implemented (`PasswordResetRequired: true`) |
| `ec2-user2` can create EC2 instances                           | Implemented (`ec2:RunInstances`)            |

## GitSync Deployment Procedure

### A. Prepare the Repository

1. Commit and push all files to your GitHub repository.
2. Keep the repository private unless your instructor requires public access.
3. Use branch protection and pull requests if required by your lab security policy.

### B. Configure CloudFormation GitSync

1. Open AWS CloudFormation in the AWS Console.
2. Choose the Git synchronization workflow for stack deployment.
3. Connect your GitHub repository and branch.
4. Select `iam-lab-cloudformation.yaml` as the template path.
5. Ensure IAM capability acknowledgement is enabled (`CAPABILITY_NAMED_IAM`).
6. Create the stack and wait for `CREATE_COMPLETE`.

### C. Deployment Configuration File

`deployment.yml` is configured with:

1. Template path: `./iam-lab-cloudformation.yaml`
2. Required capability: `CAPABILITY_NAMED_IAM`
3. Stack tags for lab tracking (`Lab=IAM-Automation`, `ManagedBy=GitSync`)

## Task 2 Validation and Evidence

### Access Test Plan

Sign in as each user and test both services.

| IAM User    | S3 Access (List Buckets) | EC2 Access (View/Launch)             |
| ----------- | ------------------------ | ------------------------------------ |
| `ec2-user1` | Expected: Fail           | Expected: Success                    |
| `ec2-user2` | Expected: Fail           | Expected: Success (including launch) |
| `s3-user`   | Expected: Success        | Expected: Fail                       |

### Required Screenshots

Capture 2 screenshots per user (6 total):

1. `Screenshots/ec2-user1-s3-result.png`
2. `Screenshots/ec2-user1-ec2-result.png`
3. `Screenshots/ec2-user2-s3-result.png`
4. `Screenshots/ec2-user2-ec2-result.png`
5. `Screenshots/s3-user-s3-result.png`
6. `Screenshots/s3-user-ec2-result.png`

Each screenshot should clearly show:

1. Signed-in IAM username
2. Service page (S3 or EC2)
3. Success or authorization failure message

## Security Best Practices Applied

1. Least privilege permissions are assigned via groups instead of direct user policies.
2. Passwords are not hardcoded; they are generated and stored in Secrets Manager.
3. First-login password reset is enforced for all IAM users.
4. IAM named-resource deployment is explicitly acknowledged with `CAPABILITY_NAMED_IAM`.
5. Git-based infrastructure change tracking is enabled for auditability.

## Deliverables Checklist

1. GitHub repository link added to this README.
2. CloudFormation stack successfully deployed from GitSync.
3. All IAM resources created as specified.
4. 6 screenshots added to `Screenshots/`.
5. Demonstration of IAM behavior prepared for review.

## Rubric Mapping

| Rubric Item                                                   | Weight | Evidence in This Submission                             |
| ------------------------------------------------------------- | -----: | ------------------------------------------------------- |
| CloudFormation template meets requirements and best practice  |    60% | `iam-lab-cloudformation.yaml` + requirement matrix      |
| Successful GitSync implementation with security best practice |    20% | `deployment.yml` + GitSync procedure + security section |
| Clear demonstration of IAM concepts                           |    20% | Validation test plan + screenshot evidence              |

## Cleanup (Optional)

To avoid ongoing charges and resource sprawl after grading:

1. Delete the CloudFormation stack.
2. Confirm IAM users, groups, policies, and secret are removed.
3. Remove screenshots containing sensitive account details before sharing publicly.
