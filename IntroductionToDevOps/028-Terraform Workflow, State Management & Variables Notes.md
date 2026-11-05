# DevOps Session — Terraform Workflow, State Management & Variables Notes

Date: 22 October 2026\
Instructor: Rochak Agrawal\
Focus: Terraform execution workflow, resource dependencies, state files, outputs, variables and environment-specific configurations.

These notes condense the session transcript, remove participant chatter, correct speech-to-text errors and clarify a few technical details where necessary.

## 1. Terraform configuration files

Terraform uses HashiCorp Configuration Language (HCL) to define infrastructure.

A Terraform configuration can be written in one file or split across multiple files. Terraform automatically loads all `.tf` and `.tf.json` configuration files in the working directory and treats them as one configuration.

For example:

```
terraform-project/
├── main.tf
├── provider.tf
├── variables.tf
├── outputs.tf
├── dev.tfvars
└── prod.tfvars
```

The separate files are for readability and organization. Terraform does not require this particular structure.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### Important Terraform blocks

| Block       | Purpose                                             |
| ----------- | --------------------------------------------------- |
| `terraform` | Terraform settings, required versions and providers |
| `provider`  | Configures the cloud provider                       |
| `resource`  | Creates and manages infrastructure                  |
| `variable`  | Defines configurable input values                   |
| `output`    | Displays useful values after execution              |
| `data`      | Reads information about existing infrastructure     |

Example:

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Meridian-VPC"
  }
}
```

Understanding resource naming

```
resource "aws_vpc" "my_vpc" {
```

- `aws_vpc` — resource type. It tells Terraform what kind of AWS resource to create.
- `my_vpc` — Terraform's internal reference name.

The name `my_vpc` is not automatically the AWS resource's Name tag. The `tags` block sets the AWS-visible name in this example.

You can reference the resource using:

```
aws_vpc.my_vpc.id
```

This returns the ID of the VPC managed by that resource block.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 2. Terraform initialization

The first command to run in a new Terraform working directory is:

```
terraform init
```

It initializes the working directory and performs tasks such as:

- Downloading required provider plugins.
- Initializing the configured backend.
- Installing required modules.

Terraform creates a `.terraform` directory and a dependency lock file, `.terraform.lock.hcl`, among other initialization metadata.

Do we need to run `terraform init` every time we modify a `.tf` file?

No. Ordinary changes to resource arguments do not normally require another initialization.

Run `terraform init` when you introduce or change providers, modules or backend configuration, and when Terraform tells you initialization is required.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 3. Terraform's execution workflow

This was the central concept demonstrated in the session.

## Terraform workflow

1\. Write configuration

Define the desired infrastructure in `.tf` files.

2\. terraform init

Initialize providers, modules and backend.

3\. terraform fmt and validate

Format and check the configuration.

4\. terraform plan

Preview proposed infrastructure changes.

5\. terraform apply

Apply the changes to actual infrastructure.

6\. Terraform state updated

Record the managed infrastructure and its attributes.

### 3.1 `terraform init`

```
terraform init
```

Initializes the Terraform project.

### 3.2 `terraform fmt`

```
terraform fmt
```

Automatically formats Terraform files according to standard HCL formatting conventions.

For example, it corrects indentation and alignment.

To format files in subdirectories too:

```
terraform fmt -recursive
```

### 3.3 `terraform validate`

```
terraform validate
```

Checks whether the configuration is structurally valid, including syntax and argument definitions against the installed provider schemas.

Important distinction: `terraform validate` does not verify whether a resource already exists in your AWS account. It is not an infrastructure availability check.

### 3.4 `terraform plan`

```
terraform plan
```

Shows what Terraform intends to do without applying the changes.

Typical plan symbols:

| Symbol | Meaning                       |
| ------ | ----------------------------- |
| `+`    | Create resource               |
| `~`    | Update resource in place      |
| `-`    | Destroy resource              |
| `-/+`  | Destroy and recreate resource |

Example:

```
Plan: 1 to add, 0 to change, 0 to destroy.
```

This means Terraform plans to create one resource.

Another example:

```
Plan: 0 to add, 1 to change, 0 to destroy.
```

This means one existing managed resource needs modification.

You may also see:

```
known after apply
```

This means Terraform cannot determine that value until the resource is created or updated. For example, a newly created VPC's ID.

### 3.5 `terraform apply`

```
terraform apply
```

Displays the proposed plan and asks for confirmation before making changes.

Enter:

```
yes
```

To skip the interactive confirmation:

```
terraform apply -auto-approve
```

Best practice: Avoid `-auto-approve` when experimenting with unfamiliar infrastructure or when destructive changes are possible.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### 3.6 `terraform destroy`

```
terraform destroy
```

Destroys resources managed by the current Terraform configuration and state.

To skip confirmation:

```
terraform destroy -auto-approve
```

Be careful: this can delete multiple resources, not just one.

## 4. Terraform state file

One of the most important topics in this session was the Terraform state file.

When Terraform creates infrastructure, it records information about the managed resources in:

```
terraform.tfstate
```

Example project:

```
terraform-project/
├── main.tf
├── terraform.tfstate
└── .terraform.lock.hcl
```

The state file helps Terraform understand which real-world resources correspond to the resource blocks in its configuration.

### Why is the state file important?

Imagine you create a VPC:

```
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}
```

Terraform creates the VPC and records its details in the state file.

Later, you change the configuration:

```
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.1.0.0/16"
}
```

Terraform compares the desired configuration with the recorded state and refreshes information from the provider to determine what changes are needed.

The plan might show an update or replacement, depending on the particular resource argument.

### How Terraform uses state

Terraform configuration

What infrastructure should exist?

Terraform state

Which resources does Terraform manage?

Actual cloud infrastructure

What exists in AWS, Azure or GCP?

Terraform uses configuration, state and provider information to calculate the required changes.

Important correction: The state file is not the infrastructure itself. It is Terraform's record of the resources it manages and their attributes. It is essential for tracking and managing those resources, but Terraform can also refresh information from the actual provider.

### What happens when you delete a resource?

Suppose Terraform manages two VPCs:

```
resource "aws_vpc" "vpc1" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_vpc" "vpc2" {
  cidr_block = "10.1.0.0/16"
}
```

After applying, both resources are tracked in the state file.

If you remove one resource block from the configuration and run `terraform plan`, Terraform will generally propose destroying that resource, because it is still tracked in state but no longer declared in the desired configuration.

### Local backend vs remote backend

By default, Terraform uses a local state file.

This creates a problem when multiple engineers work on the same infrastructure.

For example:

- Engineer A runs Terraform on their laptop.
- Engineer B runs Terraform on another laptop.
- Both have different local state files.
- Their state information can become inconsistent.

The usual team solution is a remote backend.

Examples include:

- AWS S3 backend.
- Azure Blob Storage backend.
- Terraform Cloud.

Remote state allows team members to work with shared state. State locking, where supported by the backend, helps prevent concurrent operations from modifying the same state simultaneously.

Production best practices:

- Store state remotely for team-managed infrastructure.
- Enable state locking where supported.
- Enable encryption and restrict access.
- Protect state history with backups or versioning.
- Never commit state files containing sensitive information to a public Git repository.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 5. Resources created manually vs Terraform-managed resources

An important question from the session was:

If a VPC already exists in AWS, will Terraform automatically recognize it?

Not necessarily.

Terraform does not automatically start managing every resource that exists in your AWS account.

For example:

1. You manually create a VPC in the AWS Console.
2. You write a Terraform resource block for another VPC.
3. You run `terraform plan`.
4. Terraform may propose creating a new VPC because the manually created VPC is not tracked in its state.

The same AWS Name tag can also be used by multiple VPCs, because VPC names are tags rather than globally unique identifiers.

### How do we manage an existing resource?

Terraform provides the `import` mechanism.

For example:

```
terraform import aws_vpc.my_vpc vpc-0123456789abcdef0
```

This associates the existing VPC with the Terraform resource address in state.

You must also maintain a matching configuration for the resource. Importing alone does not generate a complete configuration.

Remember: Resource names and IDs are different.

- `my_vpc` is Terraform's internal reference.
- `vpc-0123456789abcdef0` is the actual AWS resource ID.
- `Meridian-VPC` could be its AWS Name tag.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 6. Terraform output block

Sometimes a resource has attributes that are only known after creation.

For example:

- VPC ID.
- EC2 instance ID.
- Public IP address.
- DNS name.

You can expose these values using an `output` block.

Example:

```
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.my_vpc.id
}
```

After running:

```
terraform apply
```

Terraform displays the output value.

Example:

```
Outputs:

vpc_id = "vpc-0123456789abcdef0"
```

Another example for EC2:

```
output "instance_public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.web.public_ip
}
```

Practical use: You can use outputs to obtain the IP address of an EC2 instance without manually opening the AWS Console.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 7. Resource dependencies

Terraform needs to know which resources must be created before others.

There are two main types of dependencies.

### 7.1 Implicit dependency

Terraform automatically understands a dependency when one resource references another resource's attribute.

Example:

```
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public_subnet" {
  vpc_id     = aws_vpc.my_vpc.id
  cidr_block = "10.0.1.0/24"
}
```

Here:

- The VPC must exist before its subnet can be created.
- The subnet references `aws_vpc.my_vpc.id`.
- Terraform automatically identifies the dependency.

You do not need to manually specify the creation order.

### 7.2 Explicit dependency

Sometimes two resources do not have a direct attribute reference, but your application requires one to be created before the other.

For example:

- An S3 bucket contains a script.
- An EC2 instance's user-data script downloads that file.
- You want Terraform to create the bucket before the EC2 instance.

You can specify:

```
resource "aws_s3_bucket" "app_bucket" {
  bucket = "meridian-app-bucket-example"
}

resource "aws_instance" "app_server" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.micro"

  depends_on = [
    aws_s3_bucket.app_bucket
  ]
}
```

`depends_on` explicitly tells Terraform about the dependency.

### Implicit vs explicit dependency

| Implicit                                      | Explicit                                |
| --------------------------------------------- | --------------------------------------- |
| Terraform identifies dependency automatically | Engineer declares dependency            |
| Usually created through resource references   | Uses `depends_on`                       |
| Example: subnet depends on VPC                | Example: EC2 must wait for an S3 bucket |

Terraform creates a dependency graph and uses it to determine the order of operations. Independent resources can often be processed in parallel.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 8. Terraform variables

The instructor demonstrated why hardcoding values is not ideal.

Imagine you have the following VPC configuration:

```
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"
}
```

This value is hardcoded.

But your environments may need different CIDR blocks:

| Environment | CIDR block    |
| ----------- | ------------- |
| Development | `10.0.1.0/16` |
| Testing     | `10.0.2.0/16` |
| Production  | `10.0.3.0/16` |

You do not want to create three separate Terraform configurations just because one value changes.

That is where variables help.

### 8.1 Declare a variable

Create a file named `variables.tf`:

```
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.1.0/16"
}
```

### 8.2 Reference the variable

Update `main.tf`:

```
resource "aws_vpc" "my_vpc" {
  cidr_block = var.vpc_cidr
}
```

Here:

- `variable "vpc_cidr"` declares the variable.
- `var.vpc_cidr` references the variable.
- `default` provides a fallback value.

### 8.3 Using default values

If you run:

```
terraform plan
```

and do not supply another value, Terraform uses the default:

```
10.0.1.0/16
```

### 8.4 Passing values through the command line

You can override the default value at runtime:

```
terraform plan -var="vpc_cidr=10.0.4.0/16"
```

To apply that value:

```
terraform apply -var="vpc_cidr=10.0.4.0/16"
```

### 8.5 Using `.tfvars` files

You can maintain separate variable files for different environments.

Project structure:

```
terraform-project/
├── main.tf
├── variables.tf
├── dev.tfvars
└── prod.tfvars
```

`dev.tfvars`:

```
vpc_cidr = "10.0.1.0/16"
```

`prod.tfvars`:

```
vpc_cidr = "10.0.3.0/16"
```

Run the development configuration:

```
terraform plan -var-file="dev.tfvars"
terraform apply -var-file="dev.tfvars"
```

Run the production configuration:

```
terraform plan -var-file="prod.tfvars"
terraform apply -var-file="prod.tfvars"
```

This allows you to reuse the same Terraform code across multiple environments.

### 8.6 Variable precedence

When multiple sources provide a value for the same variable, Terraform uses precedence rules.

A simplified order from lower to higher priority:

1. Variable default.
2. Automatically loaded `terraform.tfvars` and `*.auto.tfvars` files.
3. `-var-file` and `-var` command-line arguments, with command-line values taking precedence over variable files.

Environment variables using `TF_VAR_name` also provide variable values and have their own place in Terraform's precedence rules.

Interview point: A command-line value can override the default value defined in `variables.tf`.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 9. Terraform count — creating multiple resources

A participant asked how to create ten VPCs.

The instructor introduced the `count` meta-argument.

Example:

```
resource "aws_vpc" "my_vpc" {
  count      = 3
  cidr_block = "10.${count.index}.0.0/16"

  tags = {
    Name = "my-vpc-${count.index}"
  }
}
```

This creates three VPC resources.

Terraform addresses them using indexes:

```
aws_vpc.my_vpc[0]
aws_vpc.my_vpc[1]
aws_vpc.my_vpc[2]
```

`count.index` starts at zero.

Important: When creating multiple resources, make sure each resource receives valid and appropriate configuration values. In real environments, `for_each` is often a better choice when resources have distinct identities or configurations.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 10. Terraform destroy — deleting a specific resource

By default:

```
terraform destroy
```

Destroys all resources managed by the current configuration and state.

But what if you want to destroy only one resource?

Terraform supports resource targeting:

```
terraform destroy -target=aws_vpc.my_vpc
```

For a resource using `count`:

```
terraform destroy -target='aws_vpc.my_vpc[1]'
```

Use with caution

Resource targeting is mainly intended for exceptional situations and troubleshooting. It can result in an incomplete or unexpected infrastructure state if used as a routine deployment method.

For ordinary infrastructure changes, modify the configuration and let Terraform calculate the full plan.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 11. Terraform troubleshooting and logs

The session also introduced Terraform logging.

Terraform normally displays useful errors and warnings in the terminal. For deeper troubleshooting, you can enable detailed logs.

Example:

```
export TF_LOG=INFO
terraform plan
```

To capture logs in a file:

```
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log
terraform plan
```

Available log levels include:

| Level   | Information                                  |
| ------- | -------------------------------------------- |
| `TRACE` | Very detailed internal execution information |
| `DEBUG` | Debugging information                        |
| `INFO`  | General operational information              |
| `WARN`  | Warnings                                     |
| `ERROR` | Errors                                       |

Disable logging:

```
unset TF_LOG
unset TF_LOG_PATH
```

Security note: Detailed logs may contain sensitive information. Do not share them publicly without reviewing and redacting them.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 12. Provider configuration and cloud knowledge

Terraform uses providers to interact with cloud platforms and other services.

Examples:

- AWS provider.
- AzureRM provider.
- Google Cloud provider.

Example:

```
provider "aws" {
  region  = "ap-south-1"
  profile = "default"
}
```

The provider configuration tells Terraform how to communicate with AWS.

Credentials can be supplied through supported authentication methods such as environment variables, profiles, IAM roles or workload identity.

Important learning from the session: Knowing Terraform syntax alone is not enough. You also need cloud knowledge.

For example, to deploy an EC2 instance in a private subnet, you should understand:

- VPCs and subnets.
- Route tables.
- Internet gateways.
- NAT gateways.
- Security groups.
- Network connectivity.

Terraform automates the infrastructure creation process; it does not replace the need to understand the infrastructure itself.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 13. Terraform is operating-system independent

Terraform can run on:

- Windows PowerShell or Command Prompt.
- Git Bash.
- Linux terminal.
- macOS terminal.
- Cloud Shell.

The Terraform configuration and core commands remain the same across these environments, assuming the required Terraform executable and provider dependencies are available.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 14. Important commands — quick revision

| Command                                 | Purpose                                |
| --------------------------------------- | -------------------------------------- |
| `terraform init`                        | Initialize the project                 |
| `terraform fmt`                         | Format Terraform files                 |
| `terraform validate`                    | Validate configuration structure       |
| `terraform plan`                        | Preview changes                        |
| `terraform apply`                       | Apply infrastructure changes           |
| `terraform apply -auto-approve`         | Apply without interactive confirmation |
| `terraform destroy`                     | Destroy managed infrastructure         |
| `terraform output`                      | Display outputs                        |
| `terraform state list`                  | List resources tracked in state        |
| `terraform show`                        | Display current state or a saved plan  |
| `terraform import`                      | Import an existing resource into state |
| `terraform plan -var="key=value"`       | Supply a variable at runtime           |
| `terraform plan -var-file="dev.tfvars"` | Load variables from a file             |

## 15. Practical DevOps example: Same infrastructure code for Dev and Production

Imagine you work on a project where the same application must be deployed in development, testing and production.

Instead of writing separate Terraform code for each environment, you can write reusable configuration.

```

# variables.tf

variable "aws_region" {
  type    = string
  default = "ap-south-1"
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.1.0/16"
}

```

```

# main.tf

provider "aws" {
  region = var.aws_region
}

resource "aws_vpc" "app_vpc" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = "Application-VPC"
  }
}

```

```

# dev.tfvars

aws_region   = "ap-south-1"
instance_type = "t3.micro"
vpc_cidr      = "10.0.1.0/16"

```

```

# prod.tfvars

aws_region   = "ap-south-1"
instance_type = "t3.large"
vpc_cidr      = "10.0.3.0/16"

```

Run:

```
terraform init

terraform plan -var-file="dev.tfvars"
terraform apply -var-file="dev.tfvars"
```

For production:

```
terraform plan -var-file="prod.tfvars"
terraform apply -var-file="prod.tfvars"
```

The same code is reused; only the input values change.

Note: The example illustrates variable reuse. The `instance_type` variable would need to be referenced by an EC2 resource to affect an actual instance.

## 16. Interview questions and answers

1\. What is Terraform state?

Terraform state is a record that maps Terraform-managed resource addresses to real infrastructure objects and stores their attributes. Terraform uses it to plan and manage changes.

2\. What is the difference between `terraform plan` and `terraform apply`?

`plan` previews the proposed changes. `apply` executes those changes against the infrastructure.

3\. Does `terraform validate` check whether a resource already exists in AWS?

No. It checks the configuration's structural validity, not the existence of remote resources.

4\. What happens if you manually create a VPC in AWS and then create a VPC using Terraform?

Terraform does not automatically manage the manually created VPC. It may create another VPC unless the existing one is imported or referenced through an appropriate data source.

5\. What is the difference between implicit and explicit dependencies?

Implicit dependencies are identified through resource references. Explicit dependencies are declared using `depends_on`.

6\. Why do we use Terraform variables?

To make configuration reusable and allow different values for different environments without changing the core infrastructure code.

7\. What is a `.tfvars` file?

A file used to supply values for Terraform input variables, commonly for environment-specific configuration.

8\. What is a remote backend?

A backend that stores Terraform state remotely instead of only on the local machine. It supports shared team workflows and, depending on the backend, state locking.

9\. What does `terraform destroy` do?

It destroys resources managed by the current Terraform configuration and state.

10\. Can Terraform create multiple resources using one resource block?

Yes. It can use `count` or `for_each` to manage multiple instances of a resource.

11\. What is the purpose of an output block?

To expose useful resource attributes, such as an EC2 public IP or VPC ID, after Terraform operations.

12\. Why is cloud knowledge important when using Terraform?

Terraform automates infrastructure creation, but engineers still need to understand cloud architecture, resource relationships, networking and security to design correct configurations.

## 17. Quick revision — what you should remember

- Terraform reads all `.tf` files in the working directory as one configuration.
- `terraform init` initializes providers, modules and backend.
- `terraform fmt` formats the code.
- `terraform validate` checks configuration structure.
- `terraform plan` previews changes.
- `terraform apply` creates or modifies infrastructure.
- `terraform destroy` destroys managed resources.
- The state file tracks resources managed by Terraform.
- Existing manually created resources are not automatically managed by Terraform.
- `output` exposes useful resource attributes.
- Implicit dependencies are detected through references.
- Explicit dependencies use `depends_on`.
- Variables make code reusable across environments.
- `.tfvars` files provide environment-specific values.
- Remote backends are important for team collaboration.

## 18. Final mental model

Think of Terraform as a tool that translates your infrastructure requirements into actual cloud resources.

| Terraform concept   | Think of it as                               |
| ------------------- | -------------------------------------------- |
| `.tf` configuration | What you want                                |
| Provider            | How Terraform communicates with the cloud    |
| `terraform plan`    | Preview before execution                     |
| `terraform apply`   | Execute the changes                          |
| State file          | Terraform's record of managed infrastructure |
| Variables           | Values you can change without rewriting code |
| Outputs             | Information returned after deployment        |
| Dependencies        | Which resources must exist first             |
| Remote backend      | Shared state storage for a team              |

The most important concept from this session: Terraform is declarative. You describe the infrastructure you want, and Terraform calculates the operations required to move the managed infrastructure toward that desired configuration.

Next session: The instructor indicated that the next class would go deeper into writing Terraform HCL, understanding resource blocks and learning configuration parameters through documentation.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



Original transcript: [DevOps Session — 22 October 2026](https://raw.githubusercontent.com/jeevanadv2000-pixel/python/main/DevOps%20Session%28Morning%29%20transcript_2026-10-22_09.00.43.txt?utm_source=chatgpt.com).