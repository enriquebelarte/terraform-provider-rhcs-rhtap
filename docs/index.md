---
# generated by https://github.com/hashicorp/terraform-plugin-docs
page_title: "Red Hat Cloud Services (rhcs) Provider"
subcategory: ""
description: |-

---
<a href="https://redhat.com">
    <img src="https://raw.githubusercontent.com/terraform-redhat/terraform-provider-rhcs/main/.github/Logo_Red_Hat.png" alt="RedHat logo" title="RedHat" align="right" max-width="60px" />
</a>

# Red Hat Cloud Services Provider

> Please note that this Terraform provider and its modules are open source and will continue to iterate features, gradually maturing this code. If you encounter any issues, please report them in this repo.

## Introduction

The Red Hat Cloud Services provider allows Terraform to manage Red Hat OpenShift Service on AWS (ROSA) clusters, machine pools, and an identity provider.

### Red Hat OpenShift Cluster Manager

Red Hat OpenShift Cluster Manager is a managed service where you can install, modify, operate, and upgrade your Red Hat OpenShift clusters. This service allows you to work with all of your organization’s clusters from a single dashboard.

More information can be found [here](https://access.redhat.com/documentation/en-us/red_hat_openshift_service_on_aws/4/html/red_hat_openshift_cluster_manager/ocm-overview).

### ROSA
Red Hat OpenShift Service on AWS (ROSA) is a fully-managed, turnkey application platform that allows you to focus on delivering value to your customers by building and deploying applications.

ROSA provides seamless integration with a wide range of AWS compute, database, analytics, machine learning, networking, mobile, and other services to further accelerate the building and delivering of differentiating experiences to your customers.

You subscribe to the service directly from your AWS account. After the clusters are created, you can operate your clusters with the OpenShift web console or through the OpenShift Cluster Manager. The ROSA service also uses OpenShift APIs and command-line interface (CLI) tools. These tools provide a standardized OpenShift experience to use your existing skills and tools knowledge.

More information can be found [here](https://access.redhat.com/documentation/en-us/red_hat_openshift_service_on_aws/4/html/introduction_to_rosa/rosa-understanding).

### AWS STS

AWS Secure Token Service (STS) is a global web service that provides short-term credentials for Identity Access Management (IAM) or federated users. ROSA with STS is the recommended credential mode for ROSA clusters. You can use AWS STS with ROSA to allocate temporary, limited-privilege credentials for component-specific IAM roles. The service enables cluster components to make AWS API calls using secure cloud resource management practices.

You can use the ROSA CLI to create the IAM role, policy, and identity provider resources that are required for ROSA clusters that use STS.

AWS STS aligns with principles of least privilege and secure practices in cloud service resource management. The ROSA CLI manages the STS credentials that are assigned for unique tasks and takes action upon AWS resources as part of OpenShift functionality. A limitation of using STS is that roles must be created for each ROSA cluster.

### ROSA STS mode

To deploy a ROSA cluster that uses the STS, you must create the following AWS IAM resources:

* Specific account-wide IAM roles and policies that provide the STS permissions required for ROSA support, installation, control plane, and compute functionality. This includes account-wide Operator policies.
* Cluster-specific Operator IAM roles that permit the ROSA cluster Operators to carry out core OpenShift functionality.
* An OpenID Connect (OIDC) provider that the cluster Operators use to authenticate.

### Terraform

Terraform is an infrastructure-as-a-code tool that is primarily used by DevOps teams. The tool allows you to define resources in human-readable configuration files that you can version, reuse, and share. Terraform uses a declarative language model, which allows you to describe the intended goal rather than focusing on the processes.

Terraform creates and manages resources through application programming interfaces (APIs) by using "providers".

For more information, see the [Terraform documentation](https://developer.hashicorp.com/terraform).

## Limitations of the Red Hat Cloud Services Terraform provider

The following items are limitations with the current release of the Red Hat Cloud Services Terraform provider:

* The latest version is not backward compatible with version 1.0.1.
* When creating a machine pool, you need to specify your replica count. You must define either the `replicas= "<count>"` variable or provide values for the following variables to build the machine pool:
   * `min_replicas = "<count>"`
   * `max_replicas="<count>"`
   * `autoscaling_enabled=true`
* The htpasswd identity provider does not support creating the identity provider with multiple users or adding additional users to the existing identity provider.
* The S3 bucket that is created as part of the OIDC configuration must be created in the same region as your OIDC provider.
* The Terraform provider does not support auto-generated `operator_role_prefix`. You must provide your `operator_role_prefix` when creating the account roles.
* Resource created by terraform provider and deleted not by the terraform provider might cause to issue, the terraform provider wouldn't be able to recreate the resource

## Prerequisites

To use the Red Hat Cloud Services provider inside your Terraform configuration you must meet the following:

* Completed [the ROSA getting started](https://console.redhat.com/openshift/create/rosa/getstarted) requirements

  You must complete some AWS account and local configurations to create and managed ROSA clusters.

* An offline [OCM token](https://console.redhat.com/openshift/token/rosa)

  This token is generated through the Red Hat Hybrid Cloud Console. The purpose of this token is to verify that you have access and permission to create and upgrade clusters. This token is unique to your account and should not be shared.

* [GoLang version 1.20 or newer](https://go.dev/doc/install)

  To build components with Terraform, you must have the latest version of Go installed and usable on your system.

* [Terraform version 1.4.6 or newer](https://developer.hashicorp.com/terraform/downloads)

  You need to have Terraform configured for your local system. The Terraform website contains installation options for MacOS, Windows, and Linux.

* **Optional**: A [configured `*.tfvars` file](docs/terraform-vars.md).

  A `*.tfvars` file is a definition sheet for all of your variables. This method allows you to reference a file that is safely stored due to the sensitive nature of some variable values. You can create multiple `*.tfvars` files with different variable values.

* [ROSA account roles](https://access.redhat.com/documentation/en-us/red_hat_openshift_service_on_aws/4/html/introduction_to_rosa/rosa-sts-about-iam-resources)

  You need to have created the AWS account-wide roles. The specific account-wide IAM roles and policies provide the STS permissions required for ROSA support, installation, control plane, and compute functionality. This includes account-wide Operator policies.

  To create the account roles using Terraform, see the [Account Roles Terraform example](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_account_roles/).

* AWS Permissions

  The following excerpt lists the minimum AWS permissions required to run Terraform.

    ```
    {
	    "Version": "2012-10-17",
        "Statement": [
            {
                "Sid": "VisualEditor0",
                "Effect": "Allow",
                "Action": [
                    "iam:GetPolicyVersion",
                    "iam:DeletePolicyVersion",
                    "iam:CreatePolicyVersion",
                    "iam:UpdateAssumeRolePolicy",
                    "secretsmanager:DescribeSecret",
                    "iam:ListRoleTags",
                    "secretsmanager:PutSecretValue",
                    "secretsmanager:CreateSecret",
                    "iam:TagRole",
                    "secretsmanager:DeleteSecret",
                    "iam:UpdateOpenIDConnectProviderThumbprint",
                    "iam:DeletePolicy",
                    "iam:CreateRole",
                    "iam:AttachRolePolicy",
                    "iam:ListInstanceProfilesForRole",
                    "secretsmanager:GetSecretValue",
                    "iam:DetachRolePolicy",
                    "iam:ListAttachedRolePolicies",
                    "iam:ListPolicyTags",
                    "iam:ListRolePolicies",
                    "iam:DeleteOpenIDConnectProvider",
                    "iam:DeleteInstanceProfile",
                    "iam:GetRole",
                    "iam:GetPolicy",
                    "iam:ListEntitiesForPolicy",
                    "iam:DeleteRole",
                    "iam:TagPolicy",
                    "iam:CreateOpenIDConnectProvider",
                    "iam:CreatePolicy",
                    "secretsmanager:GetResourcePolicy",
                    "iam:ListPolicyVersions",
                    "iam:UpdateRole",
                    "iam:GetOpenIDConnectProvider",
                    "iam:TagOpenIDConnectProvider",
                    "secretsmanager:TagResource",
                    "sts:AssumeRoleWithWebIdentity",
                    "iam:ListRoles"
                ],
            "Resource": [
                    "arn:aws:secretsmanager:*:<ACCOUNT_ID>:secret:*",
                    "arn:aws:iam::<ACCOUNT_ID>:instance-profile/*",
                    "arn:aws:iam::<ACCOUNT_ID>:role/*",
                    "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/*",
                    "arn:aws:iam::<ACCOUNT_ID>:policy/*"
                ]
            },
            {
                "Sid": "VisualEditor1",
                "Effect": "Allow",
                "Action": [
                   "s3:*"
                ],
                "Resource": "*"
            }
        ]
    }
    ```

## Authentication and configuration

The login to your Red Hat account is done by private offline access token which can be generate at https://console.redhat.com/openshift/token/rosa

Configuration for the RHCS Provider can be derived from several sources, which are applied in the following order:

1. Parameters in the provider configuration
1. Environment Variables

## Provider Configuration

!> **Warning:** Hard-coded credentials are not recommended in any Terraform
configuration and risks secret leakage should this file ever be committed to a
public version control system.

The Red Hat Cloud Services token can be provided by adding a `token` to the `rhcs` provider block.

Usage:

```terraform
provider "rhcs" {
  token = "my-token"
}
```

### Environment Variables

The Red Hat Cloud Services token can be provided by using the `RHCS_TOKEN` environment variables.

For example:

```terraform
provider "rhcs" {}
```

```console
% export RHCS_TOKEN="my-token"
```

## Terraform examples

The example Terraform files are all considered in development and should not be used for production environments:

### Before creating a cluster
* **Required**: [Account Roles Terraform](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_account_roles/)

### Creating a ROSA cluster
* [Create a ROSA cluster that uses STS and has a managed OIDC configuration](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_rosa_sts_cluster/oidc_configuration/cluster_with_managed_oidc_config/)
* [Create a ROSA cluster that uses STS and has an unmanaged OIDC configuration](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_rosa_sts_cluster/oidc_configuration/cluster_with_unmanaged_oidc_config/)

### After creating a cluster
* [Machine pools](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_machine_pool/)
* Supported identity providers:
    * [Github](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_identity_provider/github/)
    * [Gitlab](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_identity_provider/gitlab/)
    * [Google](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_identity_provider/google/)
    * [HTPasswd](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_identity_provider/google/)
    * [LDAP](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_identity_provider/ldap/)
* [Upgrading or updating your cluster](upgrading-cluster.md)
### Create your Operator IAM roles prefix name

The Operator IAM roles will be created per cluster with the [rosa-sts](https://registry.terraform.io/modules/terraform-redhat/rosa-sts) module.