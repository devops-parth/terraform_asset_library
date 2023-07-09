To pass environment-specific backend configuration values to Terraform, you can utilize either environment variables or separate backend configuration files. Here's how you can implement each approach:

1. Environment Variables:
   - Define environment variables to store the backend configuration values specific to each environment.
   - Use these environment variables in your backend configuration block in the Terraform configuration files.

   Example:
   Let's say you have two environments: `dev` and `prod`, with different storage account names and container names for the backend. You can set the following environment variables:

   For `dev` environment:
   ```bash
   export TF_BACKEND_AZURE_STORAGE_ACCOUNT="dev-storage-account"
   export TF_BACKEND_AZURE_CONTAINER_NAME="dev-container"
   ```

   For `prod` environment:
   ```bash
   export TF_BACKEND_AZURE_STORAGE_ACCOUNT="prod-storage-account"
   export TF_BACKEND_AZURE_CONTAINER_NAME="prod-container"
   ```

   In your Terraform backend configuration, you can reference these environment variables:
   ```hcl
   terraform {
     backend "azurerm" {
       storage_account_name = var.TF_BACKEND_AZURE_STORAGE_ACCOUNT
       container_name       = var.TF_BACKEND_AZURE_CONTAINER_NAME
     }
   }
   ```

2. Separate Backend Configuration Files:
   - Create separate backend configuration files for each environment.
   - Store the environment-specific backend configuration values in these files.
   - Pass the relevant backend configuration file to Terraform during the initialization phase for each environment.

   Example:
   For `dev` environment, create a file named `backend-dev.tf`:
   ```hcl
   terraform {
     backend "azurerm" {
       storage_account_name = "dev-storage-account"
       container_name       = "dev-container"
     }
   }
   ```

   For `prod` environment, create a file named `backend-prod.tf`:
   ```hcl
   terraform {
     backend "azurerm" {
       storage_account_name = "prod-storage-account"
       container_name       = "prod-container"
     }
   }
   ```

   When initializing Terraform for each environment, provide the relevant backend configuration file:
   ```bash
   # For dev environment
   terraform init -backend-config=backend-dev.tf

   # For prod environment
   terraform init -backend-config=backend-prod.tf
   ```

Both approaches allow you to customize the backend configuration values based on the environment. Choose the method that suits your workflow and preferences.