I’ve opened the full October 26 session transcript. I’ll correct the speech-to-text errors and turn it into concise, structured Terraform notes, preserving the technical explanations and examples. I’ll also use visual flow layouts rather than relying on Mermaid diagrams.

# DevOps Session — Terraform Notes

Date: 26 October 2026 · Instructor: Rochak Agrawal

Topics: `for_each`, dynamic blocks, provisioners, Terraform workspaces and environment isolation.

I’ve corrected obvious transcript errors such as “4H” to `for_each`, “dynamic blob” to `dynamic block`, “telephone plan” to `terraform plan`, and “estate file” to `state file`. I’ve also clarified a few technical points where the spoken explanation needed more precision.

## 1. `count` vs `for_each`

Terraform provides two common ways to create multiple instances of a resource.

| Feature         | `count`                            | `for_each`                                           |
| --------------- | ---------------------------------- | ---------------------------------------------------- |
| Best suited for | Similar, interchangeable resources | Resources with distinct identities or configurations |
| Reference       | `count.index`                      | `each.key`, `each.value`                             |
| Identity        | Numeric index                      | Map key or set member                                |
| Example         | 3 similar EC2 instances            | Web, app and DB instances                            |

Important: `count` can also create resources with different configurations using `count.index`. However, `for_each` is usually easier to manage when resources have meaningful individual identities.

### Example: Create multiple IAM users

```
locals {
  iam_users = toset(["Alice", "Bob", "Charlie"])
}

resource "aws_iam_user" "users" {
  for_each = local.iam_users
  name     = each.value
}
```

How it works:

Input set

Alice

Bob

Charlie

for_each

Terraform creates three separate resources

Alice

IAM user

Bob

IAM user

Charlie

IAM user

- `toset()` converts a collection into a set and removes duplicate values.
- `each.value` gives the current set member.
- Adding another name creates another IAM user without recreating the existing users.

### Example: Different EC2 configurations

```
locals {
  ec2_instances = {
    web = {
      instance_type = "t3.micro"
    }
    app = {
      instance_type = "t3.small"
    }
    db = {
      instance_type = "t3.medium"
    }
  }
}

resource "aws_instance" "servers" {
  for_each      = local.ec2_instances
  ami           = "ami-xxxxxxxx"
  instance_type = each.value.instance_type
}
```

This creates three EC2 instances with different instance types. The AMI is an example placeholder; use a valid AMI for your region.

Remember: `each.key` gives `web`, `app` or `db`; `each.value` gives the corresponding configuration object.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 2. Dynamic blocks

A dynamic block generates repeated nested configuration blocks inside a resource.

This is different from `for_each`, which creates multiple resource instances.

### Understand the difference visually

Normal `for_each`

Creates multiple resources

Web SG

App SG

DB SG

Dynamic block

Creates multiple nested rules inside each resource

One Security Group

HTTP : 80

HTTPS : 443

SSH : 22

### Real-world example: Security groups

A security group can contain multiple inbound (ingress) and outbound (egress) rules.

Suppose you need:

- HTTP — port 80
- HTTPS — port 443
- SSH — port 22

Instead of manually repeating the ingress block, you can use a dynamic block.

```
locals {
  ingress_rules = [
    {
      description = "HTTP"
      port        = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      description = "HTTPS"
      port        = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    },
    {
      description = "SSH"
      port        = 22
      protocol    = "tcp"
      cidr_blocks = ["10.0.0.0/16"]
    }
  ]
}

resource "aws_security_group" "web" {
  name = "web-sg"

  dynamic "ingress" {
    for_each = local.ingress_rules

    content {
      description = ingress.value.description
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

How it works:

1. `dynamic "ingress"` tells Terraform to generate ingress blocks.
2. `for_each` loops through the list of rules.
3. `content` defines the contents of each generated block.
4. `ingress.value` accesses the current rule's values.

The result is one security group containing three ingress rules, not three security groups.

### Combining `for_each` and dynamic blocks

The instructor also demonstrated creating three security groups — Web, App and DB — each with its own set of rules.

Outer `for_each`: creates security groups

Web SG

App SG

DB SG

Inner dynamic block: creates rules inside each SG

Port 80

Port 443

App port

Port 443

DB port

Other examples of dynamic blocks include:

- Multiple EBS volume configuration blocks inside a supported resource.
- Multiple listener or backend configuration blocks where the provider resource schema supports them.

Important: A dynamic block can only generate nested block types supported by that Terraform resource. It does not create arbitrary AWS resources.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 3. Terraform provisioners

Provisioners execute commands or copy files during resource creation or destruction.

They are sometimes used when you need to perform additional actions after creating infrastructure.

### Types of provisioners

| Provisioner   | Purpose                                          |
| ------------- | ------------------------------------------------ |
| `local-exec`  | Runs commands on the machine executing Terraform |
| `remote-exec` | Runs commands on a remote machine                |
| `file`        | Copies files to a remote machine                 |

### Visual workflow: `remote-exec`

Terraform machine

Runs Terraform configuration

Creates EC2 instance

Remote EC2 instance

Connects using SSH

remote-exec

Install and start NGINX

Web server running

### Example: Install NGINX on an EC2 instance

```
resource "aws_instance" "web" {
  ami                    = "ami-xxxxxxxx"
  instance_type          = "t3.micro"
  key_name               = "my-ec2-key"
  vpc_security_group_ids = ["sg-xxxxxxxx"]

  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("my-ec2-key.pem")
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo dnf install nginx -y",
      "sudo systemctl enable nginx",
      "sudo systemctl start nginx"
    ]
  }
}

output "public_ip" {
  value = aws_instance.web.public_ip
}
```

This example uses Amazon Linux 2023 commands. Ubuntu typically uses `apt` instead of `dnf`.

Understanding the connection block:

| Attribute        | Meaning                                 |
| ---------------- | --------------------------------------- |
| `type`           | Connection protocol                     |
| `user`           | Remote Linux username                   |
| `private_key`    | Private SSH key used for authentication |
| `host`           | Remote machine's IP or hostname         |
| `self.public_ip` | Public IP of the resource being created |

The instructor demonstrated that a wrong package manager causes command failures, while a mismatched SSH key prevents the remote connection. The EC2 key pair and local private key must correspond.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### Provisioner troubleshooting

| Problem                | What to check                                         |
| ---------------------- | ----------------------------------------------------- |
| SSH connection timeout | Security group, network route, public IP, SSH port 22 |
| Authentication failure | Correct private key and matching EC2 key pair         |
| Permission denied      | Correct SSH username and permissions                  |
| `command not found`    | Correct Linux distribution and package manager        |
| NGINX fails to start   | Package installation, service name and logs           |

### Terraform provisioners vs Ansible

The instructor explained that Terraform is primarily used for infrastructure provisioning, while Ansible is commonly used for configuration management.

| Terraform                                    | Ansible                                            |
| -------------------------------------------- | -------------------------------------------------- |
| Creates infrastructure                       | Configures existing infrastructure                 |
| Creates EC2, VPC, S3, etc.                   | Installs software, creates users, manages services |
| Provisioners offer limited command execution | Designed for repeatable configuration management   |

Best practice: Prefer cloud-init/user data, configuration-management tools such as Ansible, or prebuilt machine images for software configuration. Terraform provisioners are generally a last resort.

Why? Provisioners have limited lifecycle and error-handling capabilities. They are not a substitute for a full configuration-management tool.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



## 4. Terraform state and environment isolation

This is one of the most important topics in this session.

Imagine you have two environments:

- Production: `t2.large`
- Development: `t2.micro`

You create two variable files:

`prod.tfvars`

```
instance_type = "t2.large"
```

`dev.tfvars`

```
instance_type = "t2.micro"
```

You run:

```
terraform apply -var-file="prod.tfvars"
```

Terraform creates the production instance and records it in the state file.

Now you run:

```
terraform apply -var-file="dev.tfvars"
```

The problem: Both commands use the same Terraform state. Terraform compares the configuration with the existing state and may modify or replace the production resource to match the development configuration.

One configuration folder and one state file

`prod.tfvars` → `t2.large`

`dev.tfvars` → `t2.micro`

Same Terraform state

Environments are not isolated

Risk: applying development configuration can affect production infrastructure.

The solution is to isolate environments using separate state files. Two common approaches are:

1. Separate folders and state.
2. Terraform workspaces.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 5. Terraform workspaces

Terraform workspaces allow you to manage multiple state instances using the same configuration directory.

Think of them as separate working contexts, similar in concept to Git branches — but they isolate Terraform state, not the entire infrastructure or cloud account.

### Workspace commands

| Command                                 | Purpose                             |
| --------------------------------------- | ----------------------------------- |
| `terraform workspace list`              | Lists available workspaces          |
| `terraform workspace show`              | Shows the current workspace         |
| `terraform workspace new dev`           | Creates and selects a new workspace |
| `terraform workspace select dev`        | Switches to an existing workspace   |
| `terraform workspace delete dev`        | Deletes a workspace                 |
| `terraform workspace delete -force dev` | Force-deletes a workspace           |

The `default` workspace exists automatically.

### Example workflow

```
# Check current workspace
terraform workspace show

# List all workspaces
terraform workspace list

# Create new workspaces
terraform workspace new dev
terraform workspace new prod

# Switch between workspaces
terraform workspace select dev
terraform workspace select prod
```

### Use `terraform.workspace` in configuration

You can use the current workspace name in your configuration.

```
resource "aws_s3_bucket" "app" {
  bucket = "my-app-${terraform.workspace}"
}
```

For example:

| Workspace | Bucket name      |
| --------- | ---------------- |
| default   | `my-app-default` |
| dev       | `my-app-dev`     |
| prod      | `my-app-prod`    |

S3 bucket names must be globally unique, so in real projects you should include an account or unique suffix as well.

### How workspace state files are stored

For the local backend:

Terraform project directory

terraform.tfstate

Default workspace state

terraform.tfstate.d/

dev/

terraform.tfstate

prod/

terraform.tfstate

Each workspace maintains its own state. With remote backends, the state is stored remotely according to that backend's workspace implementation.

### Destroying resources in a workspace

```
terraform workspace select dev
terraform destroy
```

This destroys the resources tracked by the current workspace's state, not the resources tracked by the other workspaces.

Important distinction:

- `terraform destroy` destroys managed infrastructure in the current workspace.
- `terraform workspace delete` deletes the workspace itself. It does not automatically destroy the infrastructure.
- A workspace normally must have no managed resources before it can be deleted, unless you use the force option. Be careful: force deletion can leave infrastructure unmanaged.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+3

### Are workspaces enough for production isolation?

Not always. Workspaces separate state, but they still share the same configuration directory, provider configuration and potentially credentials and backend context.

For critical production environments, separate accounts, permissions, backends and deployment pipelines are often safer than relying on workspaces alone.

## 6. Separate folders vs workspaces

| Feature                      | Separate folders                                | Workspaces                                  |
| ---------------------------- | ----------------------------------------------- | ------------------------------------------- |
| Configuration                | Separate directories                            | Same directory                              |
| State                        | Separate state per folder                       | Separate state per workspace                |
| Environment-specific changes | Easy to maintain independently                  | Can use `terraform.workspace` and variables |
| Complexity                   | More duplicated files possible                  | Less duplication                            |
| Production isolation         | Can be stronger with separate accounts/backends | State isolation only                        |

Example folder structure:

```
terraform/
├── dev/
│   ├── main.tf
│   └── terraform.tfvars
│
└── prod/
    ├── main.tf
    └── terraform.tfvars
```

Practical recommendation: Use workspaces for simple environment separation, testing and proof-of-concept work. For important production systems, consider separate configurations and stronger account/backend isolation.

## 7. Quick revision

| Concept               | Remember this                                                |
| --------------------- | ------------------------------------------------------------ |
| `count`               | Creates multiple resource instances using numeric indexes    |
| `for_each`            | Creates multiple resource instances using keys or set values |
| `each.key`            | Current map key or set member                                |
| `each.value`          | Current value                                                |
| `toset()`             | Converts a collection to a set and removes duplicates        |
| Dynamic block         | Generates repeated nested configuration blocks               |
| `local-exec`          | Executes on the Terraform machine                            |
| `remote-exec`         | Executes on the remote machine                               |
| `file` provisioner    | Copies files to the remote machine                           |
| Connection block      | Defines how Terraform connects to a remote machine           |
| State file            | Tracks Terraform-managed infrastructure                      |
| Workspace             | Separate state instance for the same configuration directory |
| `terraform.workspace` | Returns the current workspace name                           |

## 8. Interview questions

1. What is the difference between `count` and `for_each`?
2. What is a dynamic block in Terraform?
3. Can you use `for_each` and a dynamic block together?
4. What is the difference between a resource-level `for_each` and a dynamic block?
5. What are the three Terraform provisioners?
6. What is the difference between `local-exec` and `remote-exec`?
7. Why is an SSH connection block required for `remote-exec`?
8. Why are Terraform provisioners considered a last resort?
9. What problem do Terraform workspaces solve?
10. What is the difference between separate folders and workspaces?
11. Does deleting a workspace destroy its infrastructure?
12. Are workspaces sufficient for isolating production environments?

## 9. The mental model to remember

`for_each`

Create multiple resources

Dynamic block

Create multiple nested configurations

Provisioners

Run commands or copy files during resource lifecycle

Workspaces

Maintain separate Terraform states for environments

Original transcript: [DevOps Session — 26 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-10-26_09.28.00.txt?utm_source=chatgpt.com)

Session complete. These notes cover the main technical topics and demonstrations from the transcript, with corrections and practical examples for revision.