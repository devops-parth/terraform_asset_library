## Templating

Terraform provides a built-in template engine called `templatefile` that allows you to generate dynamic content based on variables and expressions. The `templatefile` function evaluates a template file and replaces placeholders or expressions with the provided values. Let's explore some examples of using the Terraform template engine in both template files and Terraform manifests:

1. Using variables in template files:
```
# template.tpl
Hello, ${var.name}!
```
```hcl
# main.tf
data "template_file" "example" {
  template = file("template.tpl")
  vars = {
    name = "John"
  }
}

output "example_output" {
  value = data.template_file.example.rendered
}
```
In this example, the `template.tpl` file contains a placeholder `${var.name}`. In the `main.tf` file, the `template_file` data source is used to render the template file, and the `vars` argument is used to pass variable values. The output will be `Hello, John!`.

2. Using expressions and conditionals in template files:
```
# template.tpl
${if var.environment == "production"}This is a production environment.${else}This is a development environment.${endif}
```
```hcl
# main.tf
data "template_file" "example" {
  template = file("template.tpl")
  vars = {
    environment = "development"
  }
}

output "example_output" {
  value = data.template_file.example.rendered
}
```
In this example, the template file includes an `if` condition to check the value of `var.environment`. Based on the condition, it generates different content. The output will be `This is a development environment.`.

3. Using loops in template files:
```
# template.tpl
The numbers are:
${~ for num in var.numbers ~}${num},
${~ endfor ~}
```
```hcl
# main.tf
data "template_file" "example" {
  template = file("template.tpl")
  vars = {
    numbers = [1, 2, 3, 4, 5]
  }
}

output "example_output" {
  value = data.template_file.example.rendered
}
```
In this example, the template file includes a loop that iterates over the `numbers` variable and generates content for each element. The output will be:
```
The numbers are:
1,
2,
3,
4,
5,
```

4. Using template functions:
```
# template.tpl
The uppercase value is: ${upper(var.value)}
```
```hcl
# main.tf
data "template_file" "example" {
  template = file("template.tpl")
  vars = {
    value = "hello"
  }
}

output "example_output" {
  value = data.template_file.example.rendered
}
```
In this example, the `upper` template function is used to convert the value of `var.value` to uppercase. The output will be `The uppercase value is: HELLO`.

These examples demonstrate how you can use the Terraform template engine in both template files and Terraform manifests to generate dynamic content based on variables, expressions, conditionals, and loops. The `templatefile` function is flexible and can be adapted to various use cases to generate configuration files, scripts, or any other text-based content needed in your infrastructure provisioning process.

## Templating with JSON formatting 
> I.e. without "," in the end of JSON loop

Certainly! To ensure proper JSON formatting without a comma at the end of the loop, you can use conditional logic and the `length` function to check if an element is the last one in the loop. Here's an example:

```hcl
# template.tpl
{
  "numbers": [
    ${~ for num in var.numbers ~}
      ${num}${if num != var.numbers[length(var.numbers) - 1]},${endif}
    ${~ endfor ~}
  ]
}
```

In this example, the loop iterates over the `var.numbers` list and outputs each element followed by a comma, except for the last element. The `if` condition checks if the current `num` is not equal to the last element in the list (`var.numbers[length(var.numbers) - 1]`). If it's not the last element, a comma is appended; otherwise, no comma is added.

For instance, if `var.numbers` is `[1, 2, 3]`, the rendered template will be:

```json
{
  "numbers": [
    1,
    2,
    3
  ]
}
```

This approach ensures proper JSON formatting by omitting the trailing comma after the last element in the loop.

Let's break down the line `${num}${if num != var.numbers[length(var.numbers) - 1]},${endif}` in more detail:

- `${num}`: This inserts the value of the `num` variable into the template at this position. It represents the current element in the loop iteration.

- `${if num != var.numbers[length(var.numbers) - 1]},${endif}`: This is a conditional statement that checks if the current element `num` is not equal to the last element of the `var.numbers` list.

  - `num != var.numbers[length(var.numbers) - 1]`: This condition compares the current element `num` with the last element of the list. It checks if they are not equal.

  - `${if ... },${endif}`: This is a conditional construct in the template syntax. If the condition within the `if` statement is true, it includes the comma `,` before the `endif`. Otherwise, it excludes the comma.

    - `,`: This is the comma that will be added if the condition is true, indicating that there are more elements in the list.

By combining these components, the line `${num}${if num != var.numbers[length(var.numbers) - 1]},${endif}` ensures that each element `num` in the loop is followed by a comma `,` if it is not the last element in the `var.numbers` list. This helps in generating a well-formatted JSON array without a trailing comma after the last element.

## Template Functions

Some commonly used template functions in Terraform along with a brief description and examples:

1. `length`: Returns the length of a list, string, or map.
   Example: `${length(var.my_list)}` returns the length of the `my_list` variable.

2. `upper`: Converts a string to uppercase.
   Example: `${upper("hello")}` returns "HELLO".

3. `lower`: Converts a string to lowercase.
   Example: `${lower("WORLD")}` returns "world".

4. `join`: Concatenates elements of a list into a single string using a separator.
   Example: `${join(", ", var.my_list)}` joins the elements of `my_list` with a comma and space separator.

5. `slice`: Returns a subsequence of a list, string, or map.
   Example: `${slice(var.my_list, 0, 2)}` returns the first two elements of `my_list`.

6. `replace`: Replaces occurrences of a substring in a string with another substring.
   Example: `${replace("Hello, World!", "World", "Terraform")}` replaces "World" with "Terraform" in the string.

7. `jsonencode`: Encodes a value as a JSON string.
   Example: `${jsonencode(var.my_map)}` encodes `my_map` as a JSON string.

8. `regexall`: Extracts all occurrences of a regular expression in a string.
   Example: `${regexall("([A-Za-z]+)", "Hello, World!")}` extracts all words from the string.

9. `timestamp`: Converts a string representation of a timestamp to a Unix timestamp.
   Example: `${timestamp("2022-01-01T12:00:00Z")}` converts the given timestamp to Unix timestamp.

10. `format`: Formats a string with placeholders based on given values.
    Example: `${format("Hello, %s!", "John")}` formats the string by replacing `%s` with "John".

These are just a few examples of commonly used template functions in Terraform. Terraform's template engine offers a wide range of functions that you can utilize to manipulate values, format strings, perform calculations, and more, providing flexibility in generating dynamic content within your templates. For more details and a comprehensive list of available functions, you can refer to the Terraform documentation.

### Example
Certainly! Here's an example that combines a template file, a Terraform manifest file, and the final output using some of the template functions we discussed:

1. Template File (`template.tpl`):
```hcl
{
  "name": "${upper(var.name)}",
  "description": "${var.description}",
  "values": [${join(", ", var.values)}],
  "formattedValues": "${join(", ", [format("'%s'", value) for value in var.values])}"
}
```

2. Terraform Manifest File (`main.tf`):
```hcl
variable "name" {
  type    = string
  default = "example"
}

variable "description" {
  type    = string
  default = "Example template file"
}

variable "values" {
  type    = list(any)
  default = ["value1", "value2", "value3"]
}

data "template_file" "example" {
  template = file("template.tpl")
  vars = {
    name        = var.name
    description = var.description
    values      = var.values
  }
}

output "example_output" {
  value = data.template_file.example.rendered
}
```

3. Final Output:
After running `terraform apply` and retrieving the output, the final output will be:
```json
{
  "name": "EXAMPLE",
  "description": "Example template file",
  "values": ["value1", "value2", "value3"],
  "formattedValues": "'value1', 'value2', 'value3'"
}
```

In this example, the template file `template.tpl` includes the usage of template functions such as `upper`, `join`, and `format`. The Terraform manifest file `main.tf` defines variables for `name`, `description`, and `values`, and passes them to the `template_file` data source.

The final output demonstrates the rendered template with dynamic values. The `name` variable is converted to uppercase using the `upper` function. The `values` variable is included as a JSON array and also formatted as a comma-separated list of values enclosed in single quotes.

By running `terraform apply` and inspecting the `example_output` output, you will see the final output containing the rendered template with the applied template functions.

## Template Function EXHAUSTIVE LIST WITH EXAMPLE

Certainly! Here are some commonly used template functions in Terraform, along with example manifests and their corresponding outputs:

1. `templatefile`:
   The `templatefile` function renders a template file using variables and generates a resulting string.

   Example:
   `template.tpl`:
   ```hcl
   Hello, ${var.name}!
   ```

   `main.tf`:
   ```hcl
   variable "name" {
     type    = string
     default = "John Doe"
   }

   output "greeting" {
     value = templatefile("template.tpl", {
       name = var.name
     })
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   greeting = Hello, John Doe!
   ```

2. `jsonencode`:
   The `jsonencode` function converts a value to its JSON representation as a string.

   Example:
   ```hcl
   variable "config" {
     type = map(string)
     default = {
       key1 = "value1"
       key2 = "value2"
     }
   }

   output "json_config" {
     value = jsonencode(var.config)
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   json_config = {"key1":"value1","key2":"value2"}
   ```

3. `file`:
   The `file` function reads the contents of a file and returns it as a string.

   Example:
   `content.txt`:
   ```
   This is some content.
   ```

   `main.tf`:
   ```hcl
   output "file_content" {
     value = file("content.txt")
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   file_content = This is some content.
   ```

4. `base64encode`:
   The `base64encode` function encodes a string as a base64 representation.

   Example:
   ```hcl
   variable "secret" {
     type    = string
     default = "password123"
   }

   output "encoded_secret" {
     value = base64encode(var.secret)
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   encoded_secret = cGFzc3dvcmQxMjM=
   ```

5. `basename`:
   The `basename` function extracts the base name of a file or directory path.

   Example:
   ```hcl
   variable "file_path" {
     type    = string
     default = "/path/to/file.txt"
   }

   output "base_name" {
     value = basename(var.file_path)
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   base_name = file.txt
   ```

6. `dirname`:
   The `dirname` function extracts the directory path from a file or directory path.

   Example:
   ```hcl
   variable "file_path" {
     type    = string
     default = "/path/to/file.txt"
   }

   output "dir_path" {
     value = dirname(var.file_path)
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   dir_path = /path/to
   ```

7. `format`:
   The `format` function returns a formatted string using a format specifier and one or more arguments.

   Example:
   ```hcl
   variable "name" {
     type    = string
     default = "John Doe"
   }

   output "greeting" {
     value = format("Hello, %s!", var.name)
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   greeting = Hello, John Doe!
   ```

8. `lower` and `upper`:
   The `lower` function converts a string to lowercase, while the `upper` function converts a string to uppercase.

   Example:
   ```hcl
   variable "text" {
     type    = string
     default = "Hello World"
   }

   output "lowercase" {
     value = lower(var.text)
   }

   output "uppercase" {
     value = upper(var.text)
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   lowercase = hello world
   uppercase = HELLO WORLD
   ```

9. `regexall`:
   The `regexall` function extracts all matches of a regular expression in a given string.

   Example:
   ```hcl
   variable "text" {
     type    = string
     default = "Hello World!"
   }

   output "matches" {
     value = regexall("[a-zA-Z]+", var.text)
   }
   ```

   Output:
   ```
   Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

   Outputs:

   matches = [
     "Hello",
     "World"
   ]
   ```

10. `abspath`:
    The `abspath` function converts a relative path to an absolute path.

    Example:
    ```hcl
    variable "relative_path" {
      type    = string
      default = "path/to/file.txt"
    }

    output "absolute_path" {
      value = abspath(var.relative_path)
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    absolute_path = /full/path/to/file.txt
    ```

11. `merge`:
    The `merge` function combines multiple maps into a single map.

    Example:
    ```hcl
    variable "map1" {
      type = map(string)
      default = {
        key1 = "value1"
      }
    }

    variable "map2" {
      type = map(string)
      default = {
        key2 = "value2"
      }
    }

    output "merged_map" {
      value = merge(var.map1, var.map2)
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    merged_map = {
      "key1" = "value1"
      "key2" = "value2"
    }
    ```

12. `coalesce`:
    The `coalesce` function returns the first non-null value from a list of expressions.

    Example:
    ```hcl
    variable "var1" {
      type    = string
      default = null
    }

    variable "var2" {
      type    = string
      default = "default value"
    }

    output "result" {
      value = coalesce(var.var1, var.var2)
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    result = default value
    ```

Certainly! Here are a few more commonly used template functions in Terraform, along with example manifests and their corresponding outputs:

13. `slice`:
    The `slice` function returns a subset of elements from a list, based on the specified start and end indices.

    Example:
    ```hcl
    variable "numbers" {
      type    = list(number)
      default = [1, 2, 3, 4, 5]
    }

    output "subset" {
      value = slice(var.numbers, 1, 3)
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    subset = [
      2,
      3
    ]
    ```

14. `length`:
    The `length` function returns the number of elements in a list or the number of characters in a string.

    Example:
    ```hcl
    variable "numbers" {
      type    = list(number)
      default = [1, 2, 3, 4, 5]
    }

    output "count" {
      value = length(var.numbers)
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    count = 5
    ```

15. `join`:
    The `join` function concatenates a list of strings into a single string, using a specified separator.

    Example:
    ```hcl
    variable "words" {
      type    = list(string)
      default = ["Hello", "World", "!"]
    }

    output "sentence" {
      value = join(" ", var.words)
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    sentence = Hello World !
    ```

16. `timestamp`:
    The `timestamp` function returns the current timestamp in UTC.

    Example:
    ```hcl
    output "current_time" {
      value = timestamp()
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    current_time = 2022-07-12T10:15:30Z
    ```

17. `replace`:
    The `replace` function replaces all occurrences of a substring within a string with a new substring.

    Example:
    ```hcl
    variable "text" {
      type    = string
      default = "Hello, World!"
    }

    output "modified_text" {
      value = replace(var.text, "World", "Terraform")
    }
    ```

    Output:
    ```
    Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

    Outputs:

    modified_text = Hello, Terraform!
    ```

These additional template functions provide further flexibility and control when manipulating data, performing string operations, and working with lists in your Terraform manifests.