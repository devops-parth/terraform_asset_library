In Terraform, you can perform various types of validations to ensure the correctness of your configuration. Here are some examples of validation techniques along with their corresponding manifest files and potential output:

1. Syntax Validation:
   - Manifest:
     ```hcl
     terraform {
       required_version = ">= 0.12"
     }
     ```
   - Output: When you run Terraform, it checks if the version specified in the `required_version` constraint is met. If the version is not satisfied, Terraform will display an error message.

2. Provider Validation:
   - Manifest:
     ```hcl
     provider "aws" {
       region = "us-west-2"
     }
     ```
   - Output: During the Terraform initialization phase, it verifies if the specified provider plugin is available and compatible with the given configuration. If the provider is not installed or the version is incompatible, Terraform will show an error message.

3. Variable Validation:
   - Manifest:
     ```hcl
     variable "instance_type" {
       type    = string
       default = "t2.micro"
       validation {
         condition = startsWith(var.instance_type, "t2.")
         error_message = "Invalid instance type. Must start with 't2.'"
       }
     }
     ```
   - Output: When you attempt to apply the configuration, Terraform checks if the condition specified in the `validation` block is satisfied for the variable `instance_type`. If the condition is not met, Terraform displays the provided error message.

4. Resource Validation:
   - Manifest:
     ```hcl
     resource "aws_instance" "example" {
       ami           = "ami-0123456789abcdef0"
       instance_type = var.instance_type

       lifecycle {
         prevent_destroy = true
       }
     }
     ```
   - Output: In this example, the `lifecycle` block sets the `prevent_destroy` argument to `true` for the resource. This prevents accidental destruction of the resource when executing `terraform destroy`. Terraform will display a warning message to inform you that the resource cannot be destroyed.

These are just a few examples of validation techniques in Terraform. Depending on your specific requirements, you can implement custom validations using conditions, functions, and other constructs available in the Terraform language. The output will vary depending on the specific validation and whether it passes or fails. When a validation fails, Terraform will typically display an error or warning message explaining the issue.

### Example using CONTAINS

```hcl
variable "firewall_sku_tier" {
  description = "(Required) SKU tier of the Firewall. Possible values are Premium, Standard, and Basic."
  default     = "Standard"
  type        = string

  validation {
    condition = contains(["Premium", "Standard", "Basic" ], var.firewall_sku_tier)
    error_message = "The value of the sku tier property of the firewall is invalid."
  }
}
```