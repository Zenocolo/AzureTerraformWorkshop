# Challenge 04 - Terraform Modules

In this challenge, you will create a module to contain a scalable virtual machine deployment, then create an environment where you will call the module.

A module in Terraform is a container for multiple resources that are used together. Modules can be used to create lightweight abstractions, so that you can describe your infrastructure in terms of its architecture, rather than directly in terms of physical objects.

## How to

### Create Folder Structure

From the Cloud Shell, change directory into a folder specific to this challenge. If you created the scaffolding in Challenge 00, then then you can use the command `cd ~/AzureWorkChallenges/challenge04/`.

In order to organize your code, create the following folder structure:

```sh
├── environments
│   └── dev
│       └── main.tf
└── modules
    └── my_virtual_machine
        └── main.tf
        └── variables.tf
        └── outputs.tf
        └── provider.tf
        
```

### Create the Module

Inside the `my_virtual_machine` module folder copy over the terraform configuration from challenge 03.

### Create Variables

Extract vm size, username and password into variables without defaults in the file `variables.tf`, adjust also the variable name to "challenge04".

This will result in them being required.

```hcl
variable "vm_size" {}
variable "username" {}
variable "password" {}
```

### Create the Environment

Change your working directory to the `environments/dev` folder.

Update `main.tf` to declare your module, it could look similar to this:

```hcl
module "myawesomewindowsvm" {
  source = "../../modules/my_virtual_machine"
  name   = "awesomeapp"
}
```

Create `variables.tf` to declare your variables:

```hcl
variable "username" {}
variable "password" {}
```


> Notice the relative module sourcing.

### Terraform Init

Run `terraform init`.

```sh
Initializing modules...
- module.myawesomewindowsvm
  Getting source "../../modules/my_virtual_machine"

Error: module "myawesomewindowsvm": missing required argument "name"
Error: module "myawesomewindowsvm": missing required argument "vm_size"
Error: module "myawesomewindowsvm": missing required argument "username"
Error: module "myawesomewindowsvm": missing required argument "password"
```

We have a problem! We didn't set required variables for our module.

Update the `main.tf` file:

```hcl
module "myawesomewindowsvm" {
  source = "../../modules/my_virtual_machine"
  name   = "awesomeapp"
  vm_size  = "Standard_A2_v2"
  username = var.username
  password = var.password
}
```

Run `terraform init` again, this time there should not be any errors.

## Terraform Plan

Run `terraform plan` and you should see your linux VM built from your module.

```sh
  + module.myawesomewindowsvm.azurerm_resource_group.module
      id:                                 <computed>
      location:                           "centralus"
      name:                               "awesomeapp-rg"

...

Plan: 6 to add, 0 to change, 0 to destroy.
```

## Add Another Module

Add another `module` block describing another set of Virtual Machines:

```hcl
module "differentwindowsvm" {
  source = "../../modules/my_virtual_machine"
  name   = "differentapp"
  vm_size  = "Standard_A2_v2"
  username = var.username
  password = var.password
}
```

## Scale a single module

Set the count of your first module to 2 and rerun a plan.

```hcl
...
    vmcount = 2
...
```

Run a plan and observer that your first module can scale independently of the second one.

## Terraform Plan

Since we added another module call, we must run `terraform init` again before running `terraform plan`.

We should see twice as much infrastructure in our plan.

```sh
  + module.myawesomewindowsvm.azurerm_resource_group.module
      id:                                 <computed>
      location:                           "centralus"
      name:                               "awesomeapp-rg"

...

  + module.differentlinuxvm.azurerm_resource_group.module
      id:                                 <computed>
      location:                           "centralus"
      name:                               "differentapp-rg"

...

Plan: 12 to add, 0 to change, 0 to destroy.

```

## More Variables

In your `environments/dev/main.tf` file we can see some duplication and secrets we do not want to store in configuration.

Create a new file and name it `terraform.tfvars` that will contain our secrets and automatically loaded when we run a `plan`.

```hcl
username = "testadmin"
password = "Password1234!"
```

## Terraform Plan

Run `terraform plan` and verify that your plan succeeds and looks the same.

## Advanced areas to explore

1. Use environment variables to load your secrets.


## Resources

- [Using Terraform Modules](https://www.terraform.io/docs/modules/usage.html)
- [Source Terraform Modiules](https://www.terraform.io/docs/modules/sources.html)

What's next?
==============

Once this section is completed, go back to [the agenda](../../README.md).
