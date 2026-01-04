# 🌐 YourDomain Global Infrastructure

This repository manages the foundational "Level 0" resources for the YourDomain ecosystem. 
These resources are environment-agnostic and provide the security, identity, and networking backbone required by all other service stacks.

## 🏗️ Managed Resources

* **Route 53 Hosted Zone**: Public DNS management for `yourdomain.com`.
* **ACM Certificate**: Amazon-issued SSL/TLS certificate with DNS validation for `yourdomain.com` and `www.yourdomain.com`.
* **GitHub Connection**: AWS CodeStar connection for cross-repository CI/CD integration.
* **Git Sync IAM Role**: The `CloudFormationGitSyncRole`, which provides the necessary permissions for AWS to automatically sync infrastructure changes from GitHub.

## 📂 Repository Structure

* `global-infra.yaml`: The CloudFormation template defining the resources.
* `deployment-file.yaml`: The Git Sync configuration file used by CloudFormation to manage the stack.

## 🚀 Deployment & Update Logic

This stack is managed via **AWS CloudFormation Git Sync**.

1.  **Changes**: Any commit pushed to the `main` branch of this repository triggers an automatic CloudFormation stack update.
2.  **Validation**: When adding new domain names, the ACM certificate will enter a `PENDING_VALIDATION` state. You must ensure the DNS CNAME records are present in Route 53 (handled automatically by this template once the hosted zone is live).

## ⚠️ Important Considerations

### 1. Manual Nameserver Handshake
If this stack is recreated, AWS will generate four new **Nameservers (NS records)**. You **must** manually update these at your domain registrar (**Hover**) to point to the new Route 53 zone; otherwise, SSL validation and website traffic will fail.

### 2. Deletion Policy
Resources in this stack (specifically the Hosted Zone and IAM Role) are critical. The `DeletionPolicy` is set to `Retain` on certain resources to prevent accidental loss of the DNS zone during stack updates.

### 3. Cross-Stack Referencing
This repo provides the following **Exports**, which are consumed by the `example-static-infra` repository:
* `GlobalResourcesStack-GitHubConnectionArn`
* `GlobalResources-CertificateArn`
* `GlobalResources-HostedZoneId`

---

## 🛠️ Troubleshooting

| Issue | Cause | Resolution |
| :--- | :--- | :--- |
| **Stack Stuck in `CREATE_IN_PROGRESS`** | ACM Validation | Check ACM Console; ensure "Create Records in Route 53" has been clicked or nameservers are propagated. |
| **Git Sync Permission Error** | IAM Role | Ensure the `CloudFormationGitSyncRole` has the `InfrastructurePermissions` statement with `cloudfront:*` and `route53:*` access. |
| **Export Name Already Exists** | Naming Conflict | Ensure no other stack in the account is attempting to export the same name string. |
