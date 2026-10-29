# Branch and pipeline guide

[Back to README](../README.md) · [Infrastructure guide](infrastructure.md)

## Branches inspected

| Branch | Configuration observed |
| --- | --- |
| `main` | Terraform `~> 1.6.3`; no `.github/workflows` file |
| `stage` | Terraform `~> 1.13.4`; [Terraform workflow](https://github.com/MichaelGarrido/iac-vprofile/blob/stage/.github/workflows/terraform.yml) installs `1.6.3` |

The branch comparison was made against `main` commit `45d654e` and `stage` commit `c47cbe6`. Branches can change; inspect the current files before relying on this description.

## What the `stage` workflow does

The workflow is named **Vprofile IAC** and runs in `terraform/` on `ubuntu-latest`:

1. Checks out the repository and installs Terraform `1.6.3`.
2. Configures AWS credentials from repository secrets.
3. Initializes S3 state, overriding the bucket from a secret and using a fixed state key and region.
4. Checks formatting and validates configuration.
5. Writes a plan to `planfile`, then explicitly fails the job if planning failed.
6. Applies that plan automatically only when the Git ref is `refs/heads/main` and planning succeeded.

| Event | Declared filter | Intended effect |
| --- | --- | --- |
| Push | `main` or `stage`, with changes under `terraform/**` | Plan; apply only on `main` |
| Pull request | Target `main`, with changes under `terraform/**` | Plan without the apply step |

The workflow file must exist in the relevant branch/event context to run. The `main` checkout inspected for this documentation does not contain it.

## Required configuration

The workflow references `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `BUCKET_TF_STATE` as GitHub secrets. It uses access keys rather than an OIDC role. No credentials are supplied by this documentation.

Its backend key is fixed to `terraform.tfstate`. There is no branch-specific state key; runs configured with the same bucket and key would address the same state. `AWS_REGION` is fixed in the workflow, while the `EKS_CLUSTER` environment variable is declared but never passed to Terraform.

## Blockers and publishing implications

- On `stage`, the installed Terraform version does not satisfy that branch's required version, so initialization cannot proceed successfully without reconciling the versions.
- No GitHub environment approval, job concurrency group, or explicit state locking appears in the workflow. Additional controls depend on repository and AWS configuration.
- A documentation-only change outside `terraform/**` does not match the current path filter. A change to `.gitignore` also falls outside that filter.
- Copying or merging the workflow into `main` changes the repository's automation capability. A qualifying push to `main` can automatically apply infrastructure once its prerequisites and version issues are resolved.
