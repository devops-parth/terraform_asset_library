Various conditional statements commonly used in Terraform, along with examples of manifests and their corresponding outputs.

1. Ternary Conditional Operator (`condition ? true_value : false_value`):
   This conditional statement evaluates a condition and returns one of two values based on the result.

   Example:
   ```hcl
   variable "environment" {
     type    = string
     default = "production"
   }

   output "greeting" {
     value = var.environment == "production" ? "Hello!" : "Hi!"
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   greeting = Hello!
   ```

   In this example, the output value of `greeting` is determined by the condition `var.environment == "production"`. If the condition is true, the output is set to `"Hello!"`, otherwise, it's set to `"Hi!"`.

2. Null Coalescing Operator (`value != "" ? value : null`):
   This conditional statement checks if a value is not empty, and if it is not, returns the value; otherwise, it returns `null`.

   Example:
   ```hcl
   variable "optional_value" {
     type    = string
     default = ""
   }

   output "value_output" {
     value = var.optional_value != "" ? var.optional_value : null
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   value_output = null
   ```

   In this example, the output value of `value_output` is determined by the condition `var.optional_value != ""`. If `var.optional_value` is not an empty string, it is assigned to the output; otherwise, `null` is assigned.

3. Conditional Expression (`condition ? true_value : null`):
   This conditional statement evaluates a condition and returns the true value if the condition is true; otherwise, it returns `null`.

   Example:
   ```hcl
   variable "flag" {
     type    = bool
     default = true
   }

   output "result" {
     value = var.flag ? "Condition is true!" : null
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   result = Condition is true!
   ```

   In this example, if the condition `var.flag` is true, the output is set to `"Condition is true!"`; otherwise, it's set to `null`.

These are just a few examples of conditional statements in Terraform. The choice of conditional statement depends on the specific condition and desired behavior in your configuration.

By utilizing these conditional statements, you can dynamically control the values assigned to variables, resources, outputs, and other parts of your Terraform configuration based on different conditions.

## True or False

Here's an explanation along with a manifest example for the line "var.public_ip ? 1 : 0" in the given code:

```hcl
resource "azurerm_public_ip" "public_ip" {
  name                = "${var.name}PublicIp"
  location            = var.location
  resource_group_name = var.resource_group_name
  allocation_method   = "Dynamic"
  domain_name_label   = lower(var.domain_name_label)
  count               = var.public_ip ? 1 : 0
  tags                = var.tags

  lifecycle {
    ignore_changes = [
        tags
    ]
  }
}
```

In this example, the "azurerm_public_ip" resource creates a public IP address. The `count` argument is used to determine whether to create the public IP address resource based on the value of the variable `var.public_ip`.

Here's an explanation of the line `count = var.public_ip ? 1 : 0`:
- If `var.public_ip` is true (evaluates to a non-empty or non-null value), the count will be set to 1. This means that one resource will be created.
- If `var.public_ip` is false (evaluates to an empty or null value), the count will be set to 0. This means that no resource will be created.

This approach allows you to conditionally create the resource based on the value of the `var.public_ip` variable. If the variable is true, a single public IP address will be created; otherwise, no public IP address resource will be provisioned.

When you run `terraform apply`, the Terraform configuration will be applied. If `var.public_ip` is true, you will see the output indicating that the resource was created:

```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

If `var.public_ip` is false, the output will indicate that no resources were created:

```
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```

Please note that the output may vary depending on your actual configuration and variable values.

## Conditional

In the given code snippet:

```hcl
storage_account_uri = var.boot_diagnostics_storage_account == "" ? null : var.boot_diagnostics_storage_account
```

This line assigns a value to the `storage_account_uri` variable based on the condition using the ternary operator.

Here's the breakdown of the expression:

- If the value of the `var.boot_diagnostics_storage_account` variable is an empty string (`""`), the condition `var.boot_diagnostics_storage_account == ""` evaluates to `true`.
- In this case, the expression after the `?` is evaluated, which assigns the value `null` to the `storage_account_uri` variable.
- If the value of the `var.boot_diagnostics_storage_account` variable is not an empty string, the condition `var.boot_diagnostics_storage_account == ""` evaluates to `false`.
- In this case, the expression after the `:` is evaluated, which assigns the value of `var.boot_diagnostics_storage_account` to the `storage_account_uri` variable.

In summary, this line of code sets the `storage_account_uri` variable to either the value of `var.boot_diagnostics_storage_account` if it is not an empty string, or `null` if it is an empty string. This allows you to conditionally assign a value based on the presence of a non-empty storage account value.

The output value of `storage_account_uri` will depend on the value assigned to `var.boot_diagnostics_storage_account`. If `var.boot_diagnostics_storage_account` is an empty string, the output value will be `null`. Otherwise, it will be the value of `var.boot_diagnostics_storage_account`.