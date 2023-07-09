In Terraform, there are several variable types available for defining input variables. Here are some commonly used variable types along with example manifests and default values:

1. String Variable:
   A string variable represents a sequence of characters.

   Example:
   ```hcl
   variable "environment" {
     type    = string
     default = "development"
   }
   ```

   In this example, the variable "environment" is defined as a string type with a default value of "development".

2. Number Variable:
   A number variable represents numeric values.

   Example:
   ```hcl
   variable "instance_count" {
     type    = number
     default = 2
   }
   ```

   In this example, the variable "instance_count" is defined as a number type with a default value of 2.

3. Boolean Variable:
   A boolean variable represents true or false values.

   Example:
   ```hcl
   variable "enable_feature" {
     type    = bool
     default = true
   }
   ```

   In this example, the variable "enable_feature" is defined as a boolean type with a default value of true.

4. List Variable:
   A list variable represents an ordered sequence of values.

   Example:
   ```hcl
   variable "regions" {
     type    = list(string)
     default = ["us-west-1", "eu-west-1", "ap-southeast-2"]
   }
   ```

   In this example, the variable "regions" is defined as a list of strings with a default value containing three regions.

5. Map Variable:
   A map variable represents a collection of key-value pairs.

   Example:
   ```hcl
   variable "tags" {
     type    = map(string)
     default = {
       Name        = "my-instance"
       Environment = "production"
     }
   }
   ```

   In this example, the variable "tags" is defined as a map of strings with default key-value pairs for resource tagging.

6. Object Variable:
   An object variable represents a complex data structure composed of multiple attributes.

   Example:
   ```hcl
   variable "instance" {
     type = object({
       name     = string
       cpu      = number
       memory   = number
       enabled  = bool
     })
     default = {
       name    = "my-instance"
       cpu     = 2
       memory  = 8
       enabled = true
     }
   }
   ```

   In this example, the variable "instance" is defined as an object type with attributes for name, CPU, memory, and enabled status.

These are just a few examples of variable types in Terraform. The choice of variable type depends on the nature of the data you need to pass into your Terraform configuration.

You can use these variable types to define input variables in your manifests, and provide default values to set initial values for those variables. Default values are optional and can be overridden when running Terraform commands or using variable files.

In terraform `any` is used as the value type in the `map` type definition for the variable "tags". 

`any` is a special type in Terraform that represents a value of any type. It allows flexibility in defining a map where the value type can be of any type. This means that the values in the "tags" map can be of different types, such as strings, numbers, booleans, or even complex objects.

Here's an example demonstrating the usage of `map(any)` with different value types:

```hcl
variable "tags" {
  type = map(any)
  default = {
    Name        = "Example"
    Environment = "Production"
    Count       = 10
    Enabled     = true
  }
}

output "tag_values" {
  value = var.tags
}
```

In this example, the "tags" variable is defined as a map where the values can be of any type. The default value includes key-value pairs with different value types, such as strings, numbers, and booleans.

When you run Terraform with this configuration, you will see the output of the "tag_values" variable, which will display the map with the assigned values:

```
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

tag_values = {
  "Name" = "Example"
  "Environment" = "Production"
  "Count" = 10
  "Enabled" = true
}
```

Using `any` in the `map` type definition allows you to have flexibility in defining maps that can hold values of different types. This can be useful when dealing with dynamic data structures or when the specific value type is not known in advance.