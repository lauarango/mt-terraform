# Deploying Multiple GitHub Repositories and Files with Terraform 

## File Hierarchy

```
├── MT-TERRAFORM
│   ├── .terraform
│   │   └── terraform.tfstate
│   ├── terraform.tfstate.backup
│   ├── .terraform.lock.hcl
│   └── main.tf
├── backends
├── plan1
├── plan2
├── plan3.tf
├── providers.tf
├── utils
│   └── github_script.ps1
└── .gitignore
```

# Setting Up Terraform and Codespaces

## 1. Creating a New Codespace
- Create a new Codespace (like a development environment).
- Create the repository and add the `.gitignore` file for Terraform.
- Remember to turn off the auto-delete feature.

## 2. Configuring User Settings
- Open the user settings JSON file by pressing `Ctrl + Shift + P` and searching for “Preferences: Open Settings (JSON)”.
- Your `settings.json` should look like this:

```json
{
    "workbench.colorTheme": "Default Dark+",
    "files.exclude": {
        "**/.*": true
    }
}
```

## 3. Adjusting Permissions for Terraform
- By default, permissions will not allow Terraform to delete repositories. Run these scripts in the CLI to change this:
  ```sh
  echo $GITHUB_TOKEN
  unset $GITHUB_TOKEN
  ```

## 4. Authenticating with GitHub
- Authenticate with a GitHub host to log into your GitHub account and request the `delete_repo` scope for the OAuth token. This scope allows the token to delete repositories on your behalf.
  ```sh
  gh auth login -s delete_repo
  ```

## 5. Refreshing the Token
- Run this code every time you enter the Codespace to refresh the token:
  ```sh
  unset GITHUB_TOKEN && gh auth login -h github.com -p https -s delete_repo -w
  ```

## 6. Installing AWS Terraform CLI
- Follow the instructions to install Terraform from the [Terraform | HashiCorp Developer page](https://developer.hashicorp.com/terraform).

## 7. Checking Terraform Installation
- Verify that Terraform is working by running:
  ```sh
  terraform -help
  ```

## 8. Viewing the Repository
- To view the repository in your web browser, use the following command:
  ```sh
  gh repo view mtc-repo --web
  ```
  (In case it says your repo does not exist, please check your repositories. Since we configured it as private, it may not show up immediately, but it is created.)

# Configuring Providers

## 1. Understanding Providers
- A provider translates the API operations (create, update, delete, read) into standard Terraform commands, allowing the service API to be managed by Terraform.

## 2. Using the GitHub Provider
- For this lab, we will use the GitHub provider. You can find the documentation for any registered provider on the [Terraform Registry](https://registry.terraform.io/).

## 3. Adding the GitHub Provider
- The documentation provides the required code to create the integration:
  ```hcl
  terraform {
    required_providers {
      github = {
        source  = "integrations/github"
        version = "~> 6.0"
      }
    }
  }

  provider "github" {}
  ```

## 4. Initializing Terraform
- Run `terraform init` in the CLI to initialize Terraform. If something goes wrong, you can delete the `.terraform` folder and the `.terraform.lock.hcl` file that the init command creates.

## 5. Upgrading Providers
- To place updates, run:
  ```sh
  terraform init -upgrade
  ```

## 6. Ensuring Provider Version Consistency
- Make sure you use the same provider version across your configurations.

## 7. Understanding Terraform State
- Terraform maintains a state file that contains the state of the resources you’ve deployed. This file can be stored locally or in S3. The storage locations of this file are known as backends.

## 8. Configuring the Backend
- In this lab, we will configure the backend to store the state locally.

# Managing the State

## 1. Creating a State Folder
- Create a new folder in the main branch called `state`.

## 2. Creating the `backends.tf` File
- In your main branch, create a file named `backends.tf`. You can find the documentation for the local backend [here](https://www.terraform.io/docs/backends/types/local.html).
  ```hcl
  terraform {
    backend "local" {
      path = "relative/path/to/terraform.tfstate"
    }
  }
  ```

## 3. Creating the `main.tf` File
- Create a `main.tf` file in the main branch. This file declares the resources.
  ```hcl
  resource "github_repository" "mtc_repo" {
    name        = "mtc_repo"
    description = "Code for MTC"
  }
  ```

## 4. Applying the Terraform Workflow
- Follow the Terraform workflow: plan, apply, destroy. For now, we will plan and apply.
  ```sh
  terraform plan
  terraform plan -out=plan1
  terraform apply
  ```

  - `terraform plan`: Creates an execution plan, showing the changes Terraform will make to your infrastructure.
  - `terraform apply`: Applies the changes defined in the execution plan to your infrastructure.

## 5. Creating a New Resource in `main.tf`
- Add the following resource to your `main.tf` file to create a README file in the repository:
  ```hcl
  resource "github_repository_file" "readme" {
    repository          = github_repository.mtc_repo.name
    branch              = "main"
    file                = "README.md"
    content             = "# This repository is for infra developers."
    overwrite_on_create = true
  }
  ```

## 6. Creating Multiple Repositories with Random IDs
- You can create multiple repositories with random IDs using the random provider. First, define the random ID resource:
  ```hcl
  resource "random_id" "random" {
    count       = 2  // This will create 2 random IDs
    byte_length = 2
  }
  ```

- When you create the repository resource, include the count and specify the index:
  ```hcl
  resource "github_repository" "mtc_repo" {
    count       = 2
    name        = "mtc_repo-${random_id.random[count.index].dec}"
    description = "MTC Code"
    visibility  = "private"
    auto_init   = true
  }
  ```
