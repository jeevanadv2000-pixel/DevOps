# DevOps Session — Ansible Fundamentals

Date: 30 October 2026 · Instructor: Rochak Agrawal

Topics: Configuration management, Ansible architecture, agentless operation, push vs pull, inventory, modules, tasks, plays, playbooks, roles and facts.

The transcript is focused on Ansible. I’m correcting speech-recognition errors such as “Ensemble” to Ansible, “SSC” to SSH, and “BinRM” to WinRM, while clarifying concepts that were oversimplified during the lecture.

## 1. Why do we need configuration management?

Imagine an organization has hundreds or thousands of servers. Each server needs software, runtime environments, security tools, users and service configurations.

Configuring every server manually creates several problems:

| Problem             | Explanation                                                           |
| ------------------- | --------------------------------------------------------------------- |
| Configuration drift | Servers gradually become different because of manual changes          |
| Human errors        | Repeated manual commands can introduce mistakes                       |
| Time-consuming      | Configuring hundreds of servers individually takes time               |
| No consistency      | Different administrators may configure servers differently            |
| Poor documentation  | Manual fixes may not be recorded, making troubleshooting difficult    |
| Scalability         | Manual configuration becomes difficult as the number of servers grows |

Configuration management automates the process of maintaining consistent server configurations.

## 2. What is Ansible?

Ansible is an open-source automation tool used for:

- Installing and configuring software.
- Creating users and groups.
- Managing services.
- Copying files and templates.
- Executing commands.
- Automating repetitive server administration tasks.

Ansible commonly connects to Linux servers using SSH and Windows servers using WinRM. It does not normally require an agent installed on the managed machines.

### Ansible's key feature: Idempotency

Idempotency means that running the same automation repeatedly produces the same desired end state without making unnecessary changes.

For example, consider creating a Linux user named `deploy`.

A simple shell command:

```
sudo useradd deploy
```

Running it again may return an error because the user already exists.

Ansible's user module:

```
- name: Ensure deploy user exists
  ansible.builtin.user:
    name: deploy
    state: present
```

| Run        | Expected result              |
| ---------- | ---------------------------- |
| First run  | `changed` — creates the user |
| Second run | `ok` — user already exists   |
| Later runs | `ok` — no unnecessary change |

Ansible's desired-state approach

Desired state

The deploy user must exist

Compare desired state with actual state

User missing

changed

Create user

User exists

ok

No action needed

Important: Ansible is declarative in the sense that you describe the desired configuration, rather than writing every step needed to achieve it. Idempotency depends on using modules and tasks appropriately.

## 3. Ansible architecture

Ansible follows a control-node and managed-node architecture.

| Component     | Meaning                                                                     |
| ------------- | --------------------------------------------------------------------------- |
| Control node  | Machine where Ansible is installed and executed                             |
| Managed node  | Server or device Ansible manages                                            |
| Inventory     | List of managed nodes, organized into groups                                |
| Playbook      | YAML file containing one or more plays                                      |
| Ansible Core  | Automation engine that executes playbooks and tasks                         |
| `ansible.cfg` | Main Ansible configuration file                                             |
| Module        | Reusable unit of work, such as installing a package                         |
| Plugin        | Extends Ansible's functionality                                             |
| Collection    | Distribution package containing modules, plugins, roles and related content |
| Role          | Reusable, organized collection of tasks, variables, templates and handlers  |
| Facts         | Information gathered about managed hosts                                    |

### Architecture diagram

Control node

Machine running Ansible

Ansible Core

Inventory

Playbooks

ansible.cfg

SSH / WinRM / APIs

Connection from control node

Web 1

Managed node

App 1

Managed node

DB 1

Managed node

Important corrections to the lecture:

- The control node is generally Linux-based. Supported control environments include Linux distributions and macOS; Windows users commonly use WSL.
- Linux managed nodes normally use SSH.
- Windows managed nodes commonly use WinRM or SSH. RDP is for interactive remote desktop access, not Ansible's usual automation transport.
- Managed nodes do not normally need an Ansible agent installed.
- Most POSIX managed-node modules require Python. Windows modules use PowerShell and do not have the same Python requirement.
- Network devices and cloud APIs can also be managed through suitable Ansible modules and plugins.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 4. Agentless architecture

Ansible is called agentless because it does not normally require a continuously running Ansible agent on every managed node.

Compare it with traditional agent-based configuration management tools.

| Tool      | Typical architecture                          |
| --------- | --------------------------------------------- |
| Ansible   | Agentless                                     |
| Puppet    | Traditionally agent-based                     |
| Chef      | Traditionally agent-based                     |
| Salt      | Supports agent-based and agentless approaches |
| Terraform | Agentless infrastructure provisioning         |

### Agentless vs agent-based

Ansible

Agentless

Control node

SSH / WinRM

Managed server

No permanent Ansible agent

Agent-based

Agent required

Central server

Agent communication

Managed server

Agent installed

Advantages of Ansible's agentless approach:

- No separate agent installation and maintenance.
- Easier initial setup.
- Uses existing remote access mechanisms.
- Can manage many different server types.

Limitation: The control node still needs network access and valid authentication to the managed nodes. Agentless does not mean connectionless.

## 5. Push vs pull architecture

This is a common interview question.

| Feature            | Push                            | Pull                           |
| ------------------ | ------------------------------- | ------------------------------ |
| Who initiates?     | Central control node            | Agent on managed node          |
| Configuration flow | Controller sends configuration  | Agent retrieves configuration  |
| Agent required?    | Not normally for Ansible        | Typically yes                  |
| Example            | Ansible                         | Traditional Puppet/Chef setups |
| Main dependency    | Controller-to-node connectivity | Agent-to-server connectivity   |

### Push architecture — Ansible

Ansible control node

Push configuration over SSH / WinRM

Server 1

Server 2

Server 3

### Pull architecture

Central configuration server

Agents request configuration

Agent 1

Agent 2

Agent 3

Interview answer: In a push model, the controller initiates the connection and sends the configuration to managed nodes. In a pull model, agents on the managed nodes periodically contact a central server and retrieve their configuration.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 6. Terraform vs Ansible

The instructor explained that these tools have some overlapping capabilities, but their primary purposes are different.

| Terraform                               | Ansible                                                  |
| --------------------------------------- | -------------------------------------------------------- |
| Infrastructure as Code                  | Configuration management and automation                  |
| Creates VPCs, EC2, load balancers, etc. | Installs software and configures servers                 |
| Uses HCL                                | Uses YAML playbooks                                      |
| Uses providers                          | Uses modules and collections                             |
| Tracks infrastructure using state       | Checks the current state on managed nodes when tasks run |
| Commonly provisions infrastructure      | Commonly configures infrastructure after provisioning    |

### Real-world workflow

Terraform

Create VPC, subnet, security group and EC2

Ansible

Install NGINX, create users, copy files and configure services

Configured application server

Ready to deploy

### Why not use Terraform provisioners for everything?

The instructor gave an example:

1. Terraform creates an EC2 instance.
2. A provisioner installs NGINX version X.
3. Two months later, you want NGINX version Y.
4. Changing a provisioner can cause Terraform to replace the associated resource, depending on the lifecycle and resource changes.
5. Replacing an EC2 instance could also affect attached infrastructure or data if not designed carefully.

With Ansible, you can update the NGINX version in the playbook and rerun it. Ansible checks the installed version and changes the package without needing to recreate the EC2 instance.

Practical takeaway: Use Terraform for infrastructure and Ansible for ongoing server configuration. Use Terraform provisioners only when there is a good reason and the lifecycle implications are understood.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 7. Ansible terminology — understand the hierarchy

This is especially important because the instructor spent time answering questions about the difference between a task, play, playbook and role.

| Term       | Meaning                                       | Example                          |
| ---------- | --------------------------------------------- | -------------------------------- |
| Module     | Performs one specific operation               | `user`, `copy`, `apt`, `service` |
| Task       | Calls a module with arguments                 | Ensure NGINX is installed        |
| Play       | Maps a set of tasks to selected hosts         | Configure web servers            |
| Playbook   | YAML file containing one or more plays        | `site.yml`                       |
| Role       | Reusable, organized package of automation     | `webserver` role                 |
| Collection | Package of related modules, plugins and roles | `ansible.posix`                  |

### Hierarchy

Playbook

site.yml

Play 1 — Web servers

Task 1 — Install NGINX

Task 2 — Copy configuration

Task 3 — Start service

Modules

apt · copy · service

A playbook can contain multiple plays. For example:

- Play 1: Configure web servers.
- Play 2: Configure application servers.
- Play 3: Configure database servers.

### Example playbook

```
---
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install NGINX
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Start and enable NGINX
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

In this example:

- `name`: Human-readable description.
- `hosts`: Which inventory group to target.
- `become`: Request privilege escalation.
- `tasks`: List of tasks to execute.
- `ansible.builtin.package`: Module used to manage packages.
- `state: present`: Ensure the package exists.
- `state: started`: Ensure the service is running.
- `enabled: true`: Start the service automatically at boot.

### What is a role?

A role is a reusable structure that organizes automation code.

For example, if ten teams need to configure NGINX, instead of each team writing the same 100-line playbook, you can create a reusable `webserver` role and share it.

Typical role structure:

```
webserver/
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── vars/
│   └── main.yml
└── meta/
    └── main.yml
```

Remember: A role is not simply another name for a playbook. It is a reusable directory structure containing related automation components.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. Inventory — static and dynamic

An inventory is the list of hosts that Ansible manages.

It can contain:

- IP addresses or hostnames.
- Host groups and child groups.
- Host-specific variables.
- Group-specific variables.

### Static inventory example

```
[webservers]
192.168.1.10
192.168.1.11

[appservers]
192.168.1.20

[dbservers]
192.168.1.30
```

Here, you manually specify the hosts.

### Dynamic inventory

Consider AWS Auto Scaling:

- EC2 instances are created automatically.
- Instances may be terminated.
- New instances receive different IP addresses.

Maintaining IP addresses manually becomes difficult.

Dynamic inventory allows Ansible to discover hosts from an external source such as AWS.

AWS account

EC2 instances / Auto Scaling groups

Dynamic inventory plugin queries AWS

Ansible inventory

Discovers current managed hosts

Run playbook against selected hosts

| Static inventory                   | Dynamic inventory                               |
| ---------------------------------- | ----------------------------------------------- |
| Manually maintained                | Automatically discovers hosts                   |
| Suitable for stable infrastructure | Suitable for frequently changing infrastructure |
| IPs and hostnames entered manually | Uses cloud APIs or other inventory sources      |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 9. How Ansible executes a playbook

The instructor explained the execution process in detail.

1

Read playbook and inventory

Determine the selected hosts and tasks.

2

Establish connection

Connect to managed hosts using SSH, WinRM or another supported connection.

3

Gather facts

Collect operating system, IP, memory and other host information when fact gathering is enabled.

4

Transfer module and arguments

For many POSIX modules, transfer temporary module code to the managed node.

5

Execute module

Perform the requested operation on the managed node.

6

Return result

Return status and task details to the control node.

7

Clean up

Remove temporary execution files where applicable.

### Important points

- Modules normally perform their work on the managed node, not the control node.
- Many POSIX modules use Python on the managed node.
- Ansible gathers facts at the beginning of a play by default (`gather_facts: true`).
- Fact gathering can be disabled using `gather_facts: false`.
- Ansible does not maintain a Terraform-style infrastructure state file.
- Modules inspect the current state and determine whether a change is necessary.

### Ansible vs Terraform state

| Terraform                                                                         | Ansible                                             |
| --------------------------------------------------------------------------------- | --------------------------------------------------- |
| Maintains a state file                                                            | Does not maintain a Terraform-style state file      |
| Tracks managed resources                                                          | Inspects managed nodes when tasks execute           |
| Compares configuration and recorded state, refreshing provider data as configured | Modules check actual conditions on the managed host |
| Stores resource metadata in state                                                 | Returns task results such as `ok` and `changed`     |

For example, if an Ansible playbook ensures that a user exists:

- First run: user is created, status `changed`.
- Second run: user already exists, status `ok`.
- If someone deletes the user manually, the next run creates it again.

This is why Ansible is useful for maintaining the desired configuration repeatedly.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 10. Parallel execution and forks

Ansible can manage multiple hosts in parallel.

The `forks` setting controls the maximum number of hosts Ansible processes concurrently.

| Setting       | Meaning                               |
| ------------- | ------------------------------------- |
| Default forks | 5                                     |
| `forks = 10`  | Up to 10 hosts processed concurrently |
| `forks = 20`  | Up to 20 hosts processed concurrently |

You can configure this in `ansible.cfg`:

```
[defaults]
forks = 10
```

Or specify it on the command line:

```
ansible-playbook -i inventory.ini site.yml -f 10
```

Important: The instructor's explanation that every task completes on every host before the next task begins describes Ansible's default `linear` strategy.

For example:

1. Install NGINX on all selected hosts.
2. After that task completes across the hosts, start the next task.
3. Start NGINX on all selected hosts.

The `serial` option can limit the number of hosts processed in each batch. Other execution strategies, such as `free`, change the default task synchronization behavior.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 11. Facts and fact gathering

Facts are information Ansible collects about managed hosts.

Examples:

| Fact                           | Information                    |
| ------------------------------ | ------------------------------ |
| `ansible_distribution`         | Linux distribution             |
| `ansible_distribution_version` | Distribution version           |
| `ansible_default_ipv4`         | Default IPv4 interface details |
| `ansible_hostname`             | Hostname                       |
| `ansible_memtotal_mb`          | Total memory                   |
| `ansible_processor_vcpus`      | Number of virtual CPUs         |

### Why are facts useful?

Imagine you have:

- One Ubuntu server.
- One Amazon Linux server.

Ubuntu uses `apt`, while Amazon Linux 2023 uses `dnf`.

Facts allow your playbook to identify the operating system and make decisions accordingly.

### Run the fact-gathering command

```
ansible localhost -m ansible.builtin.setup
```

This gathers and displays facts about the local machine.

You can also gather facts for a group of hosts:

```
ansible webservers -m ansible.builtin.setup
```

Remember: Facts are gathered information about the host. Inventory tells Ansible which hosts to target; facts tell Ansible about those hosts.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 12. Installing Ansible

The instructor demonstrated installation on an Ubuntu EC2 instance.

The general Ubuntu installation sequence:

```
# Update package information
sudo apt update

# Install Ansible
sudo apt install ansible -y

# Verify installation
ansible --version
```

Ansible is implemented in Python, but you do not need to become a Python programmer to use standard Ansible playbooks.

The session also discussed the languages used by some DevOps tools:

| Tool       | Implementation language | Common user-facing configuration |
| ---------- | ----------------------- | -------------------------------- |
| Ansible    | Python                  | YAML                             |
| Jenkins    | Java                    | Groovy / Jenkinsfile             |
| Kubernetes | Go                      | YAML manifests                   |
| Terraform  | Go                      | HCL                              |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 13. Inventory commands and ad-hoc commands

### What are ad-hoc commands?

Ad-hoc commands allow you to execute a single task without creating a playbook.

General syntax:

```
ansible <host-pattern> -m <module> -a "<arguments>"
```

| Part             | Meaning                        |
| ---------------- | ------------------------------ |
| `ansible`        | Ansible command                |
| `<host-pattern>` | Host or group to target        |
| `-m`             | Module to execute              |
| `-a`             | Arguments passed to the module |

### Example: Create a file

```
ansible localhost \
  -m ansible.builtin.file \
  -a "path=/tmp/file.txt state=touch"
```

This creates an empty file at `/tmp/file.txt`.

### Example: Delete a file

```
ansible localhost \
  -m ansible.builtin.file \
  -a "path=/tmp/file.txt state=absent"
```

### Example: Ping all inventory hosts

```
ansible all -m ansible.builtin.ping
```

This checks Ansible connectivity and module execution. It is not the same as sending an ICMP network ping.

### Useful commands

| Command                              | Purpose                                         |
| ------------------------------------ | ----------------------------------------------- |
| `ansible --version`                  | Shows installed version and configuration paths |
| `ansible all -m ping`                | Tests connectivity to inventory hosts           |
| `ansible localhost -m setup`         | Gathers local facts                             |
| `ansible-inventory --list`           | Displays inventory in JSON                      |
| `ansible-inventory --graph`          | Displays inventory groups                       |
| `ansible-inventory --list-hosts all` | Lists hosts matching a pattern                  |
| `ansible-doc ansible.builtin.file`   | Displays module documentation                   |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 14. Quick revision

| Question                   | Answer                                                 |
| -------------------------- | ------------------------------------------------------ |
| What is Ansible?           | Automation and configuration management tool           |
| Is Ansible agentless?      | Yes, for its usual managed-node architecture           |
| What is a control node?    | Machine where Ansible runs                             |
| What is a managed node?    | Machine or device Ansible manages                      |
| What is an inventory?      | List of managed hosts                                  |
| What is a module?          | Unit of work that performs a specific operation        |
| What is a task?            | A module invocation with arguments                     |
| What is a play?            | Tasks mapped to a group of hosts                       |
| What is a playbook?        | YAML file containing one or more plays                 |
| What is a role?            | Reusable, organized automation package                 |
| What are facts?            | Information collected about managed hosts              |
| What is idempotency?       | Repeated execution does not create unnecessary changes |
| What is an ad-hoc command? | One-time task executed without a playbook              |
| What is dynamic inventory? | Inventory discovered from an external source           |
| What are forks?            | Maximum concurrent hosts processed by Ansible          |

## 15. Interview questions

1. What is configuration management and why is it needed?
2. What is Ansible and why is it called agentless?
3. What is the difference between push and pull architecture?
4. What is the difference between a control node and managed node?
5. What is the difference between a task, play, playbook and role?
6. What is the difference between a module and a collection?
7. What is inventory? Explain static and dynamic inventory.
8. How does Ansible execute a playbook?
9. What are Ansible facts and why are they useful?
10. How does Ansible achieve idempotency without maintaining a state file?
11. How does Ansible execute tasks across multiple hosts?
12. What is the purpose of forks?
13. Why should you use Ansible instead of Terraform provisioners for ongoing configuration management?
14. Can Ansible manage AWS resources and on-premises servers?

## 16. Final mental model

Inventory

Which machines should I manage?

Playbook

What configuration do I want?

Plays and tasks

Which tasks should run on which hosts?

Modules

Perform the actual operations

Managed nodes

Configured servers

One-line summary: Terraform creates the infrastructure; Ansible configures and maintains the servers running on that infrastructure.

Original transcript: [DevOps Session — 30 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-10-30_09.24.55.txt?utm_source=chatgpt.com)

Session complete

The notes now cover the full transcript, including the lecture, student questions, installation demonstration, inventory, facts and ad-hoc commands.