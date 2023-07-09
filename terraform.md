### Example 1
#
##### ./main.tf
```hcl
module "hub_network" {
  source                       = "./modules/virtual_network"
  resource_group_name          = azurerm_resource_group.rg.name
  location                     = var.location
  vnet_name                    = var.hub_vnet_name
  address_space                = var.hub_address_space
  tags                         = var.tags

  subnets = [
    {
      name : "AzureFirewallSubnet"
      address_prefixes : var.hub_firewall_subnet_address_prefix
      enforce_private_link_endpoint_network_policies : true
      enforce_private_link_service_network_policies : false
    },
    {
      name : "AzureBastionSubnet"
      address_prefixes : var.hub_bastion_subnet_address_prefix
      enforce_private_link_endpoint_network_policies : true
      enforce_private_link_service_network_policies : false
    }
  ]
}
```

##### modules/vnet/main.tf
```hcl
resource "azurerm_subnet" "subnet" {
  for_each = { for subnet in var.subnets : subnet.name => subnet }

  name                                           = each.key
  resource_group_name                            = var.resource_group_name
  virtual_network_name                           = azurerm_virtual_network.vnet.name
  address_prefixes                               = each.value.address_prefixes
  enforce_private_link_endpoint_network_policies = each.value.enforce_private_link_endpoint_network_policies
  enforce_private_link_service_network_policies  = each.value.enforce_private_link_service_network_policies
}
```

### Example 2
#
##### ./main.tf
```hcl
module "acr_private_dns_zone" {
  source                       = "./modules/private_dns_zone"
  name                         = "privatelink.azurecr.io"
  resource_group_name          = azurerm_resource_group.rg.name
  virtual_networks_to_link     = {
    (module.hub_network.name) = {
      subscription_id = data.azurerm_client_config.current.subscription_id
      resource_group_name = azurerm_resource_group.rg.name
    }
    (var.aks_network.name) = {
      subscription_id = data.azurerm_client_config.current.subscription_id
      resource_group_name = azurerm_resource_group.rg.name
    }
  }
}
```

##### modules/private_dns_zone/main.tf
```hcl
resource "azurerm_private_dns_zone_virtual_network_link" "link" {
  for_each = var.virtual_networks_to_link

  name                  = "link_to_${lower(basename(each.key))}"
  resource_group_name   = var.resource_group_name
  private_dns_zone_name = azurerm_private_dns_zone.private_dns_zone.name
  virtual_network_id    = "/subscriptions/${each.value.subscription_id}/resourceGroups/${each.value.resource_group_name}/providers/Microsoft.Network/virtualNetworks/${each.key}"

  lifecycle {
    ignore_changes = [
      tags
    ]
  }
}
```

In the provided code snippet, `"link_to_${lower(basename(each.key))}"` is used to dynamically generate the name for the `azurerm_private_dns_zone_virtual_network_link` resource. Let's break it down:

- `each.key`: It represents the key (index or name) of the current iteration in the `for_each` loop. In this case, it refers to the key of the `var.virtual_networks_to_link` map.

- `basename(each.key)`: The `basename` function is used to extract the base name from a given path or string. In this case, it is used to extract the base name from `each.key`.

- `lower(basename(each.key))`: The `lower` function is used to convert the extracted base name to lowercase.

- `"link_to_"`: This is a static string used as a prefix for the generated name.

By combining these elements, the expression `"link_to_${lower(basename(each.key))}"` generates a dynamic name for each `azurerm_private_dns_zone_virtual_network_link` resource. It concatenates the static string `"link_to_"` with the lowercase base name extracted from `each.key`. This helps ensure unique and descriptive names for the virtual network links based on the corresponding network's identifier.

For example, if `each.key` is `"myVirtualNetwork"`, the generated name would be `"link_to_myvirtualnetwork"`.


### Example 3 COALESCE
#
##### ./main.tf

The `coalesce` function in Terraform is used to select the first non-null value from a list of arguments. It returns the first argument that is not null, or null if all arguments are null.

Here's an example of a manifest using the `coalesce` function:

```hcl
variable "oms_agent" {
  description = "Specifies the OMS agent addon configuration."
  type        = object({
    enabled                     = bool           
    log_analytics_workspace_id  = string
  })
  default     = {
    enabled                     = true
    log_analytics_workspace_id  = "workspace123"
  }
}

variable "log_analytics_workspace_id" {
  type    = string
  default = null
}

resource "oms_agent" "example" {
  msi_auth_for_monitoring_enabled = true
  log_analytics_workspace_id      = coalesce(var.oms_agent.log_analytics_workspace_id, var.log_analytics_workspace_id)
}
```

In this example:

- The `oms_agent` resource is created with the provided configuration.
- The `log_analytics_workspace_id` argument of the `oms_agent` resource uses `coalesce` to select the first non-null value between `var.oms_agent.log_analytics_workspace_id` and `var.log_analytics_workspace_id`.
- If `var.oms_agent.log_analytics_workspace_id` is not null, it will be used as the value for `log_analytics_workspace_id`. Otherwise, if `var.log_analytics_workspace_id` is not null, it will be used. If both are null, the value of `log_analytics_workspace_id` will be null.

Here's an example of the input values:

- `var.oms_agent.log_analytics_workspace_id` is set to `"workspace123"`.
- `var.log_analytics_workspace_id` is not set (null).

When applying this configuration, the `log_analytics_workspace_id` value for the `oms_agent` resource will be `"workspace123"`, as it takes the first non-null value from the provided arguments.

The output will look like this:

```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

log_analytics_workspace_id = workspace123
```

As shown in the output, the value of `log_analytics_workspace_id` is `"workspace123"`, which is the non-null value selected by the `coalesce` function.

### Example 4 TRY
#
##### ./main.tf
In Terraform, `try` is a function used to attempt the evaluation of an expression and return a default value if the evaluation results in an error. It helps handle situations where a value may be undefined or cause an error.

Here's an example of a manifest using `try`, `dynamic` block, and `content` block:

```hcl
variable "ingress_application_gateway" {
  type = object({
    gateway_id   = string
    subnet_cidr  = string
    subnet_id    = string
  })
  default = null
}

resource "example_resource" "example" {
  dynamic "ingress_application_gateway" {
    for_each = try(var.ingress_application_gateway.gateway_id, null) == null ? [] : [1]

    content {
      gateway_id   = var.ingress_application_gateway.gateway_id
      subnet_cidr  = var.ingress_application_gateway.subnet_cidr
      subnet_id    = var.ingress_application_gateway.subnet_id
    }
  }
}
```

In this example:

- The `example_resource` resource is created with a dynamic block, `ingress_application_gateway`.
- The dynamic block is conditionally executed based on the evaluation of `try(var.ingress_application_gateway.gateway_id, null) == null`. If `var.ingress_application_gateway.gateway_id` is null or causes an error, the dynamic block will not be executed. If it's not null, the dynamic block will be executed once.
- Inside the dynamic block, the `content` block defines the configuration for each instance of the dynamic block.
- The values for `gateway_id`, `subnet_cidr`, and `subnet_id` are taken from the input variable `var.ingress_application_gateway`.

Here's an example of the input values:

- `var.ingress_application_gateway` is set to the following object:

  ```hcl
  {
    gateway_id   = "gateway123"
    subnet_cidr  = "10.0.0.0/24"
    subnet_id    = "subnet123"
  }
  ```

When applying this configuration, the dynamic block will be executed since `var.ingress_application_gateway.gateway_id` is not null. The `example_resource` resource will be created with the provided configuration.

The output will look like this:

```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

The dynamic block allows for conditional creation of resources based on the availability of the `gateway_id` value. If `var.ingress_application_gateway.gateway_id` is null or not provided, the dynamic block will not execute, and no resources will be created.

### Example 5 DYNAMIC CONTENT
#
##### ./main.tf
The `dynamic` block in Terraform provides a way to generate multiple blocks or nested configurations dynamically based on a collection or map of values. It allows you to create resources or configurations based on varying conditions or multiple instances of similar objects.

A common use case for the `dynamic` block is when you have a variable number of similar resources that need to be created based on a dynamic input, such as a list or map.

Here's a real example to demonstrate how the `dynamic` block works and when to use it:

Let's say you have a list of virtual machines that you want to create in Azure. Each virtual machine has a name, size, and network interface configuration. However, the number of virtual machines can vary and is not fixed. Instead of manually creating multiple resource blocks for each virtual machine, you can use the `dynamic` block to generate the necessary configurations dynamically.

```hcl
variable "virtual_machines" {
  type = list(object({
    name              = string
    size              = string
    network_interface = object({
      subnet_id = string
      ip_address = string
    })
  }))
  default = []
}

resource "azurerm_virtual_machine" "example" {
  dynamic "virtual_machine" {
    for_each = var.virtual_machines

    content {
      name              = virtual_machine.value.name
      vm_size           = virtual_machine.value.size
      network_interface {
        subnet_id    = virtual_machine.value.network_interface.subnet_id
        ip_address   = virtual_machine.value.network_interface.ip_address
      }
    }
  }
}
```

In this example:

- The `variable "virtual_machines"` represents a list of virtual machines, where each virtual machine has a name, size, and network interface configuration.
- The `azurerm_virtual_machine` resource is defined with a `dynamic` block to generate multiple instances of the resource based on the number of elements in `var.virtual_machines`.
- The `for_each` argument in the `dynamic` block iterates over each element of `var.virtual_machines`.
- Inside the `content` block, the values for each virtual machine are dynamically assigned using `virtual_machine.value`.

Here's an example of the input values:

```hcl
variable "virtual_machines" {
  default = [
    {
      name = "vm1"
      size = "Standard_D2s_v3"
      network_interface = {
        subnet_id   = "/subscriptions/12345/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/vnet1/subnets/subnet1"
        ip_address  = "10.0.0.5"
      }
    },
    {
      name = "vm2"
      size = "Standard_D4s_v3"
      network_interface = {
        subnet_id   = "/subscriptions/12345/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/vnet1/subnets/subnet2"
        ip_address  = "10.0.0.10"
      }
    }
  ]
}
```

When applying this configuration, the `azurerm_virtual_machine` resource will be created with two instances based on the provided input. Two virtual machines will be created with the specified names, sizes, and network interface configurations.

The output will look like this:

```
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

The `dynamic` block allows for generating resources dynamically based on the input list of virtual machines. It helps avoid duplicating resource blocks manually and allows for flexible configurations based on the variable input.

Note: The example above is specific to Azure and `azurerm_virtual_machine`, but the `dynamic` block concept applies to other providers and resource types in Terraform as well.

#

The name "virtual_machine" used in the `dynamic "virtual_machine"` statement is arbitrary and can be chosen based on your preference. It is not fixed and can be any valid identifier.

In the code snippet provided:

```hcl
resource "azurerm_virtual_machine" "example" {
  dynamic "vm" {
    for_each = var.virtual_machines

    content {
      name              = vm.value.name
      vm_size           = vm.value.size
      network_interface {
        subnet_id    = vm.value.network_interface.subnet_id
        ip_address   = vm.value.network_interface.ip_address
      }
    }
  }
}
```

The "vm" identifier is used as a block type within the `dynamic` block. It defines the name of the dynamically generated block for each element in the `for_each` expression.

You can choose any meaningful name that helps you identify and reference the dynamically created block within the `dynamic` block's content. It should be unique within the scope of the `dynamic` block and should not conflict with other block types or identifiers used in the same context.

For example, you could choose to name it "instance", "vm", or any other descriptive name that reflects the purpose or meaning of each dynamically generated block.

It is important to note that the chosen name is used to reference the dynamically generated block when accessing its attributes within the `content` block. In the provided code, `vm.value.name` refers to the `name` attribute of each dynamically created "vm" block.

```hcl
content {
  name = vm.value.name
  ...
}
```

Overall, the name used in the `dynamic "virtual_machine"` statement is arbitrary and can be customized based on your requirements and coding conventions.

### Example 6 TOSET
#
##### ./main.tf
Here's an example manifest using the `toset` function and its output:

```hcl
variable "numbers" {
  type    = list(number)
  default = [1, 2, 3, 3, 4, 5]
}

output "unique_numbers" {
  value = toset(var.numbers)
}
```

In this example, we have a variable `numbers` that contains a list of numbers, some of which are duplicated. We want to convert this list into a set to get only the unique numbers.

When applying this configuration, the output `unique_numbers` will display the result of applying the `toset` function to the `var.numbers` list. The duplicate numbers will be removed, and only the unique numbers will be displayed.

The output will look like this:

```
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

unique_numbers = [
  1,
  2,
  3,
  4,
  5
]
```

As you can see, the `toset` function removes the duplicate number `3` from the list, resulting in a set containing only the unique numbers `[1, 2, 3, 4, 5]`.

The `toset` function is useful in scenarios where you want to ensure uniqueness or eliminate duplicates when working with lists or tuples. It's commonly used in dynamic block iterations or when constructing unique sets of values for various purposes in your Terraform configuration.

### EXAMPLE

```hcl
resource "azurerm_monitor_diagnostic_setting" "settings" {
  name                           = var.name
  target_resource_id             = var.target_resource_id
  log_analytics_workspace_id     = var.log_analytics_workspace_id
  log_analytics_destination_type = var.log_analytics_destination_type
  storage_account_id             = var.storage_account_id

  dynamic "log" {
    for_each = toset(logs)
    content {
      category = each.key
      enabled  = true

      retention_policy {
        enabled = var.retention_policy_enabled
        days    = var.retention_policy_days
      }
    }
  }
}
```
In the provided code snippet, the line `for_each = toset(logs)` is used within the `dynamic` block to iterate over a collection of logs and dynamically generate log configurations based on those logs.

The `toset` function is used to convert the `logs` variable into a set, ensuring that each log entry is unique. This step is necessary when working with the `for_each` argument because it expects a collection with unique elements.

Here's a breakdown of how it works:

1. `logs` is expected to be a collection (e.g., list, tuple, or set) that represents the logs you want to configure within the `azurerm_monitor_diagnostic_setting` resource.

2. The `toset` function is called on `logs` to convert it into a set. This ensures that each log entry is unique, removing any duplicates that might be present.

3. The resulting set, which contains unique log entries, is then used in the `for_each` argument of the `dynamic` block. It iterates over each unique log entry to generate individual log configurations.

Inside the `dynamic` block's content, `each.key` represents the current element of the set being iterated over. It is used to set the `category` attribute for each dynamically generated log configuration.

Using the `for_each` and `toset` combination allows you to dynamically generate log configurations based on a collection of logs. It ensures that each log entry is unique and avoids creating duplicate log configurations within the `azurerm_monitor_diagnostic_setting` resource.

Note: The exact behavior and requirements of the code snippet may depend on the context and the specific usage of the `azurerm_monitor_diagnostic_setting` resource.

### Example 7 LOCALS
#
##### ./main.tf
In the provided code snippet:

```hcl
locals {
  module_tag = {
    "module" = basename(abspath(path.module))
  }
  tags = merge(var.tags, local.module_tag)
}
```

The `module_tag` local is a map that includes metadata about the module being executed. It uses two functions, `basename` and `abspath`, to extract and format the module name.

Here's an example to demonstrate its usage:

```hcl
variable "tags" {
  type    = map(string)
  default = {
    "environment" = "production"
    "team"        = "engineering"
  }
}

locals {
  module_tag = {
    "module" = basename(abspath(path.module))
  }
  tags = merge(var.tags, local.module_tag)
}

resource "example_resource" "example" {
  name = "example"
  tags = local.tags
}
```

In this example, we have a `tags` variable defined with default values. The `locals` block contains the `module_tag` and `tags` locals.

The `module_tag` local is a map with a single key-value pair, where the key is "module" and the value is the base name of the module's directory path. It uses the `basename` function to extract the base name and the `abspath` function to get the absolute path of the current module.

The `tags` local is created by merging the `var.tags` map (which represents any user-defined tags passed as input variables) with the `module_tag` local. The `merge` function combines the two maps, with the values from `module_tag` taking precedence in case of any overlapping keys.

When applying this configuration, the output will depend on the module directory path and the values provided for `var.tags`. Let's assume the current module directory is `/path/to/my-module`, and the `var.tags` map is as defined in the example.

The output will look like this:

```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

example_tags = {
  "environment" = "production"
  "module"      = "my-module"
  "team"        = "engineering"
}
```

As you can see, the `tags` local includes the merged values from `var.tags` and `local.module_tag`. The resulting map includes the original tags, along with an additional key "module" that represents the base name of the module directory path.

You can use the `module_tag` local to add metadata or context-specific information to your resources or outputs. It can be helpful when you want to include information about the module's name, location, or any other relevant details in your Terraform configuration. It provides a convenient way to dynamically generate tags or labels based on the module's characteristics.

Using locals allows you to keep your code more readable, modular, and reusable. By defining intermediate values like `module_tag`, you can avoid repetition and easily reference these values throughout your configuration.

### Example 8 PATH.MODULE
#
##### ./main.tf


Let's explain the usage of `path.module` and its variants with an example:

The `path.module` variable in Terraform represents the path to the directory containing the current module. It is a special variable provided by Terraform that you can use within your configurations to reference the current module's location.

Consider the following example directory structure:

```
my-module/
  main.tf
  variables.tf
```

Inside the `main.tf` file, you can use `path.module` to reference the current module's directory path:

```hcl
variable "module_path" {
  type    = string
  default = path.module
}

output "module_path_output" {
  value = var.module_path
}
```

In this example, we have a variable `module_path` that uses `path.module` as its default value. The `output` block then exposes the value of `module_path`.

When applying this configuration, the output will show the module's directory path:

```
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

module_path_output = /path/to/my-module
```

As you can see, the value of `module_path_output` is the absolute path to the current module's directory.

In addition to `path.module`, there are other related variants you can use:

- `path.root`: Represents the root module's directory path. It is useful when working with multi-module configurations or when you want to reference the top-level module's directory.

- `path.cwd`: Represents the current working directory from which Terraform was invoked. It is useful when you want to reference the location from which you are running Terraform.

- `path.root_module`: Represents the directory path of the root module. It is useful when working with multi-module configurations and you want to specifically reference the root module's directory.

The usage of these variables is similar to `path.module`. You can incorporate them into your configurations to reference the respective directory paths.

It's important to note that these variables are specific to Terraform and are evaluated at runtime. They provide a convenient way to access directory paths and help make your configurations more dynamic and flexible.

Remember to use the appropriate variant based on your specific requirements, whether it's referencing the current module, root module, current working directory, or root module directory.

<!-- ### Example 9 
#
##### ./main.tf -->

