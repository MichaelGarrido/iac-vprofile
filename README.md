# VProfile infrastructure on AWS

Terraform configuration for the VPC and Amazon EKS cluster used in a VProfile learning project. The repository describes infrastructure; application source and Kubernetes workload manifests are maintained separately in [vprofile-action](https://github.com/MichaelGarrido/vprofile-action).

This is a fork of [hkhcoder/iac-vprofile](https://github.com/hkhcoder/iac-vprofile). The upstream project supplies the infrastructure foundation. This fork preserves that attribution and documents its current configuration and branch differences.

## What the code defines

| Component | Configuration on `main` |
| --- | --- |
| Network | One VPC; three public and three private subnets across three availability zones |
| Outbound access | One shared NAT gateway |
| Kubernetes | EKS version `1.27`, with a public API endpoint |
| Worker nodes | Two managed node groups in private subnets, using `t3.small` instances and `AL2_x86_64` |
| Capacity | Three desired nodes in total; group limits are 1–3 and 1–2 |
| State | S3 backend; the bucket must already exist |
| Default region | `us-east-2` |

The configuration is a lab starting point. Review version pins and cloud availability before provisioning.

## Repository map

- [`terraform/vpc.tf`](terraform/vpc.tf): network, subnet tags, and NAT configuration.
- [`terraform/eks-cluster.tf`](terraform/eks-cluster.tf): EKS version and node groups.
- [`terraform/terraform.tf`](terraform/terraform.tf): Terraform/provider requirements and S3 backend.
- [`terraform/variables.tf`](terraform/variables.tf): region and cluster name inputs.
- [`terraform/outputs.tf`](terraform/outputs.tf): cluster connection information.
- [Infrastructure guide](docs/infrastructure.md): architecture, preparation, validation, and operational gaps.
- [Branch and pipeline guide](docs/pipeline.md): the workflow found on `stage` and its current blockers.

## Start with validation

Clone this fork, then run commands from its `terraform` directory:

```sh
git clone https://github.com/MichaelGarrido/iac-vprofile.git
cd iac-vprofile/terraform
terraform version
terraform init -backend=false
terraform fmt -check
terraform validate
```

On `main`, the Terraform requirement is `~> 1.6.3` (1.6.3 or a later 1.6 patch). Initialization downloads the declared modules and providers; `-backend=false` avoids initializing the S3 backend. Planning against the intended AWS account is a separate step.

For a real plan, first select the AWS account, configure credentials, and choose a state bucket and key you own. Follow the [infrastructure guide](docs/infrastructure.md) before connecting to remote state or applying changes. Infrastructure creates billable cloud resources.

## Pipeline status

`main` has no checked-in GitHub Actions workflow. The remote `stage` branch contains a Terraform workflow, but it installs Terraform `1.6.3` while that branch requires `~> 1.13.4`. Its apply step is conditional on a push to `main`. See the [pipeline guide](docs/pipeline.md) for triggers, dependencies, and publishing implications.
