# Secure Google Cloud Storage (GCS) Bucket Terraform Component

This repository contains an enterprise-ready, secure Google Cloud Storage (GCS) bucket component designed for use in Google Cloud's **Application Design Center (ADC)** and general Terraform deployments.

## Features & Security Hardening

This component is pre-configured with industry-standard security defaults to prevent data leaks and ensure compliance:
- **Uniform Bucket-Level Access**: Enabled by default to simplify IAM policy management and prevent accidental public object exposure.
- **Public Access Prevention**: Enforced by default (`public_access_prevention = "enforced"`), ensuring no public ACLs or IAM policies can be applied to the bucket.
- **Object Versioning**: Enabled by default to protect against accidental deletions or overwrites.
- **Customer-Managed Encryption Keys (CMEK)**: Fully supported via the optional `kms_key_name` parameter.
- **Dynamic Lifecycle Rules**: Supports complex lifecycle management policies (e.g., auto-archiving or deleting old versions after a specified number of days).

---

## Architecture Diagram

```mermaid
graph TD
    User([User / Application]) -->|Reads/Writes| GCS[Secure GCS Bucket]
    GCS -->|Encryption| KMS[Cloud KMS Key (Optional)]
    GCS -->|Lifecycle| LC[Lifecycle Rules]
    GCS -->|Access Control| IAM[Uniform IAM Policies]
```

---

## Usage

### Simple Usage
```hcl
module "secure_bucket" {
  source     = "git::https://github.com/YOUR_ORGANIZATION/YOUR_REPO.git//custom-tf-component"
  project_id = "my-gcp-project"
  name       = "my-unique-secure-bucket"
  location   = "us-central1"
}
```

### Advanced Usage (with KMS Encryption and Lifecycle Rules)
```hcl
module "secure_bucket_advanced" {
  source             = "git::https://github.com/YOUR_ORGANIZATION/YOUR_REPO.git//custom-tf-component"
  project_id         = "my-gcp-project"
  name               = "my-unique-advanced-bucket"
  location           = "us-central1"
  storage_class      = "STANDARD"
  versioning_enabled = true
  kms_key_name       = "projects/my-gcp-project/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key"
  force_destroy      = false

  labels = {
    environment = "production"
    team        = "data-science"
  }

  lifecycle_rules = [
    {
      action = {
        type          = "SetStorageClass"
        storage_class = "NEARLINE"
      }
      condition = {
        age = 30
      }
    },
    {
      action = {
        type = "Delete"
      }
      condition = {
        num_newer_versions = 3
      }
    }
  }
}
```

---

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| `project_id` | The ID of the GCP project where the bucket will be created. | `string` | n/a | yes |
| `name` | The name of the GCS bucket. Must be globally unique. | `string` | n/a | yes |
| `location` | The location of the bucket (e.g., US, EU, us-central1). | `string` | `"US"` | no |
| `storage_class` | The Storage Class of the bucket. | `string` | `"STANDARD"` | no |
| `versioning_enabled` | Set to true to enable versioning. | `bool` | `true` | no |
| `kms_key_name` | Resource name of a Cloud KMS key to encrypt objects. | `string` | `null` | no |
| `force_destroy` | Delete all objects when deleting the bucket. | `bool` | `false` | no |
| `labels` | A mapping of labels to assign to the bucket. | `map(string)` | `{}` | no |
| `lifecycle_rules` | List of lifecycle rules to configure. | `list(object)` | `[]` | no |

## Outputs

| Name | Description |
|------|-------------|
| `bucket_name` | The name of the GCS bucket. |
| `bucket_url` | The `gs://` URL of the GCS bucket. |
| `bucket_self_link` | The URI of the created bucket resource. |
| `bucket_location` | The location of the bucket. |

---

## Integrating with Google Cloud Application Design Center (ADC)

To make this component available in your Application Design Center catalog:

1. **Host the Repository**: Push this directory to your organization's version control system (such as Google Cloud Source Repositories, GitHub Enterprise, or GitLab).
2. **Register in ADC**:
   - Navigate to the **Application Design Center** in the Google Cloud Console.
   - Go to the **Catalog / Components** section.
   - Click **Add Component** or **Register Custom Component**.
   - Provide the Git repository URL, branch/tag, and specify the directory path (`/` or `./`).
   - ADC will parse the `metadata.yaml` and your Terraform variable definitions to make it instantly selectable and configurable in the visual architecture canvas!
