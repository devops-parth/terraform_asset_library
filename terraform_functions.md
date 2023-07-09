## basename

Here's an example to demonstrate the use of `basename` function when virtual network names contain paths or extensions:

```hcl
variable "virtual_networks_to_link" {
  type = map(object({
    subscription_id      = string
    resource_group_name  = string
  }))
  default = {
    "virtual_networks/myVirtualNetwork1" = {
      subscription_id     = "12345"
      resource_group_name = "myResourceGroup1"
    }
    "virtual_networks/myVirtualNetwork2" = {
      subscription_id     = "67890"
      resource_group_name = "myResourceGroup2"
    }
  }
}

resource "azurerm_private_dns_zone_virtual_network_link" "link" {
  for_each = var.virtual_networks_to_link

  name                  = "link_to_${lower(basename(each.key))}"
  resource_group_name   = var.resource_group_name
  private_dns_zone_name = "myPrivateDnsZone"
  virtual_network_id    = "/subscriptions/${each.value.subscription_id}/resourceGroups/${each.value.resource_group_name}/providers/Microsoft.Network/virtualNetworks/${each.key}"
}

output "link_names" {
  value = [azurerm_private_dns_zone_virtual_network_link.link[key].name for key in var.virtual_networks_to_link]
}
```

In this example:

- The `virtual_networks_to_link` variable is defined as a map with keys representing virtual network names containing paths (`virtual_networks/` prefix).

- The `azurerm_private_dns_zone_virtual_network_link` resource is created using the `for_each` loop over the `virtual_networks_to_link` map.

- The `name` attribute of the `azurerm_private_dns_zone_virtual_network_link` resource uses `"link_to_${lower(basename(each.key))}"`. The `basename` function is used to extract only the base name from the virtual network names, removing the path prefix.

After applying the configuration and retrieving the output, the `link_names` output will provide a list of the generated names for the virtual network links:

```
link_names = [
  "link_to_myvirtualnetwork1",
  "link_to_myvirtualnetwork2"
]
```

In this case, the use of `basename` ensures that only the base names (`myVirtualNetwork1` and `myVirtualNetwork2`) are used in the generated names, excluding the path prefix (`virtual_networks/`).

## Lookup

Here's the manifest along with an explanation of the `lookup` function and an example output:

```hcl
source_image_reference {
  sku     = lookup(var.os_disk_image, "sku", null)
  version = lookup(var.os_disk_image, "version", null)
}
```

In this example, the `lookup` function is used to retrieve values from the `var.os_disk_image` map variable.

The `lookup` function has the following syntax: `lookup(map, key, default)`. It searches for the value associated with the given `key` in the `map` and returns it. If the `key` is not found, it returns the `default` value.

In the code snippet, the `lookup` function is used to retrieve the values for the `sku` and `version` attributes in the `source_image_reference` block. If the corresponding key is found in the `var.os_disk_image` map, it will use the value associated with that key. Otherwise, it will default to `null`.

For example, let's say the `var.os_disk_image` map is defined as follows:

```hcl
variable "os_disk_image" {
  type    = map
  default = {
    sku     = "Ubuntu"
    version = "18.04"
  }
}
```

If the `var.os_disk_image` map is set with the above values, the `source_image_reference` block will be populated as follows:

```hcl
source_image_reference {
  sku     = "Ubuntu"
  version = "18.04"
}
```

On the other hand, if the `var.os_disk_image` map is not provided or is empty, the `source_image_reference` block will be populated with `null` values:

```hcl
source_image_reference {
  sku     = null
  version = null
}
```

This allows you to have flexible configuration by providing default values in the `var.os_disk_image` map, and using the `lookup` function to retrieve those values or fallback to `null` if the map is not present or the key is not found.

The output will depend on the values defined in `var.os_disk_image` and how they are used in your Terraform configuration.