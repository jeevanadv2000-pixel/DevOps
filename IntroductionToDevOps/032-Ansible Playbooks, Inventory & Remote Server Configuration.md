# DevOps Session — Ansible Playbooks and Remote Server Configuration

Date: 31 October 2026 · Instructor: Rochak Agrawal

Topics: Ad-hoc commands vs playbooks, inventory, YAML syntax, SSH authentication, remote server management, privilege escalation, NGINX installation, modules and collections.

I’ve corrected speech-to-text errors such as “Ensemble” to Ansible, “SSEC” to SSH, “APT cache” references and other technical terms. I've also clarified a few inaccuracies in the lecture, especially around SSH key compatibility and playbook execution.

## 1. Ad-hoc commands vs playbooks

In the previous session, we learned about Ansible ad-hoc commands. In this session, the instructor demonstrated how to perform the same operations using playbooks.

| Ad-hoc command                   | Playbook                                             |
| -------------------------------- | ---------------------------------------------------- |
| Executes a single task           | Executes one or more plays containing multiple tasks |
| Runs directly from the terminal  | Written in a YAML file                               |
| Suitable for one-time operations | Suitable for repeatable automation                   |
| Less structured                  | Organized and reusable                               |
| Good for quick checks            | Good for configuration management                    |

### Example: Ad-hoc command

Create a file on localhost:

```
ansible localhost \
  -m ansible.builtin.file \
  -a "path=/tmp/demo.txt state=touch"
```

### Same operation using a playbook

Create `playbook.yml`:

```
---
- name: Manage local files
  hosts: localhost
  connection: local

  tasks:
    - name: Ensure file exists
      ansible.builtin.file:
        path: /tmp/demo.txt
        state: touch
```

Execute:

```
ansible-playbook -i inventory.ini playbook.yml
```

Understanding the command:

| Part               | Purpose                          |
| ------------------ | -------------------------------- |
| `ansible-playbook` | Executes the playbook            |
| `-i inventory.ini` | Specifies the inventory file     |
| `playbook.yml`     | Specifies what Ansible should do |

Remember: Inventory tells Ansible where to run. The playbook tells Ansible what to do.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 2. Inventory file — where to manage servers

An inventory file contains the list of servers that Ansible should manage.

The instructor demonstrated two approaches:

1. Using the default inventory file.
2. Creating a separate inventory file and passing it through `-i`.

### Default inventory

The commonly configured default inventory path is:

```
/etc/ansible/hosts
```

When you run Ansible without specifying `-i`, Ansible uses the inventory configured in `ansible.cfg`. In a standard installation, this is commonly `/etc/ansible/hosts`.

Example:

```
ansible all -m ansible.builtin.ping
```

### Custom inventory

You can create your own inventory file anywhere.

Example:

```
project/
├── inventory.ini
└── playbook.yml
```

`inventory.ini`:

```
[webservers]
192.168.1.10
192.168.1.11

[dbservers]
192.168.1.20
```

Execute:

```
ansible-playbook -i inventory.ini playbook.yml
```

### Can we create multiple inventory files?

Yes. The instructor explained why organizations often maintain different inventory files for different teams or environments.

For example:

```
inventories/
├── java-servers.ini
├── python-servers.ini
├── dev-servers.ini
└── prod-servers.ini
```

Run a playbook using a particular inventory:

```
ansible-playbook -i inventories/java-servers.ini install-java.yml
```

Or:

```
ansible-playbook -i inventories/python-servers.ini install-python.yml
```

Important points:

- Inventory filenames can be anything.
- `.ini` is a common format, but YAML inventory is also supported.
- Inventory files can be stored in different directories.
- Provide the correct path using `-i`.
- You can organize hosts into groups instead of maintaining separate files.

### Why not always use the default inventory?

Imagine an organization has:

- 10 servers requiring Java.
- 20 servers requiring Python.
- 50 web servers.
- 30 database servers.

Separate inventories or groups make it easier to target the correct machines and allow teams to maintain their own host lists.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Playbook structure — play and tasks

The instructor demonstrated creating multiple tasks within a play and multiple plays within a single playbook.

### Understand the hierarchy

Playbook

playbook.yml

Play 1 — Manage files

Task 1 — Create file

Task 2 — Change ownership

Play 2 — Manage directories

Task 1 — Create directory

Task 2 — Set permissions

### Example: Multiple tasks in one play

```
---
- name: Manage local files
  hosts: localhost
  connection: local

  tasks:
    - name: Ensure file exists
      ansible.builtin.file:
        path: /tmp/demo.txt
        state: touch

    - name: Change file ownership
      ansible.builtin.file:
        path: /tmp/demo.txt
        owner: ubuntu
```

### Example: Multiple plays in one playbook

```
---
- name: Manage files
  hosts: localhost
  connection: local

  tasks:
    - name: Create a file
      ansible.builtin.file:
        path: /tmp/demo.txt
        state: touch

- name: Manage directories
  hosts: localhost
  connection: local

  tasks:
    - name: Create project directory
      ansible.builtin.file:
        path: /home/ubuntu/new-project
        state: directory
        owner: ubuntu
        mode: "0755"
```

Key distinction:

- One playbook can contain multiple plays.
- Each play can contain multiple tasks.
- Each task normally calls a module.
- Tasks within a play execute in the order written.

### Task execution order matters

Unlike Terraform's dependency graph, Ansible normally executes tasks sequentially in the order they appear within a play.

For example:

Task 1: Install NGINX

Task 2: Configure NGINX

Task 3: Start NGINX

If you try to start NGINX before installing it, the task may fail because the service does not exist.

Remember: Plan the order of your tasks according to their dependencies.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 4. YAML syntax and indentation

Ansible playbooks are generally written in YAML.

YAML is indentation-sensitive, meaning spaces determine the structure and hierarchy.

Example:

```
---
- name: Install NGINX
  hosts: webservers

  tasks:
    - name: Install package
      ansible.builtin.apt:
        name: nginx
        state: present
```

Notice:

- `tasks` is under the play.
- The task name is under `tasks`.
- The module is under the task.
- Module arguments are indented under the module.

### Common YAML mistakes

Incorrect:

```
tasks:
- name: Install NGINX
  ansible.builtin.apt:
  name: nginx
  state: present
```

Correct:

```
tasks:
  - name: Install NGINX
    ansible.builtin.apt:
      name: nginx
      state: present
```

### Validate playbook syntax

The instructor briefly discussed syntax validation. The correct command is:

```
ansible-playbook --syntax-check playbook.yml
```

With a custom inventory:

```
ansible-playbook \
  -i inventory.ini \
  --syntax-check \
  playbook.yml
```

This checks playbook syntax without executing the tasks.

Additional recommendation: Use a YAML-aware editor or VS Code with Ansible and YAML extensions to catch indentation and syntax mistakes while writing.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 5. Connecting Ansible to remote EC2 servers

This was the major practical demonstration in the session.

Previously, the instructor executed tasks on `localhost`. Now the goal was to configure a separate Ubuntu EC2 instance from the Ansible control node.

### Architecture

Ansible control node

Runs Ansible and stores playbooks

SSH connection

TCP port 22

Remote Ubuntu EC2

Managed node

### Step 1: Create the remote EC2 instance

The instructor launched a second Ubuntu EC2 instance.

Important requirements:

- Ubuntu AMI.
- Valid SSH username (`ubuntu`).
- Network connectivity between the two instances.
- Security group allowing inbound SSH on TCP port 22 from the control node.

If both instances are in the same VPC and have suitable routing, private IP connectivity is usually preferable.

### Step 2: Add the remote server to inventory

Example:

```
[webservers]
web1 ansible_host=172.31.10.20 ansible_user=ubuntu
```

Here:

| Variable       | Meaning             |
| -------------- | ------------------- |
| `webservers`   | Inventory group     |
| `web1`         | Inventory alias     |
| `ansible_host` | Actual server IP    |
| `ansible_user` | Remote SSH username |

### Step 3: Configure SSH authentication

Ansible needs to authenticate with the remote server.

The common approach is SSH key-based authentication.

Control node

`id_ed25519` — private key

Keep secret

Copy public key only

Remote server

`authorized_keys`

Stores public key

Generate an ED25519 key pair on the control node:

```
ssh-keygen -t ed25519
```

This creates:

```
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

| File             | Purpose                                     |
| ---------------- | ------------------------------------------- |
| `id_ed25519`     | Private key; never share                    |
| `id_ed25519.pub` | Public key; can be copied to remote servers |

### Step 4: Copy the public key

If you already have a way to authenticate to the remote server, use:

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@172.31.10.20
```

Alternatively, manually append the public key to the remote user's:

```
~/.ssh/authorized_keys
```

Typical permissions:

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
```

Test the connection:

```
ssh -i ~/.ssh/id_ed25519 ubuntu@172.31.10.20
```

Once SSH works from the control node, Ansible can use the same connection.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+3

### Important correction: RSA vs ED25519

During the demonstration, the instructor suggested that RSA 4096 was not supported by AWS EC2 and that changing to the default SSH key resolved the problem.

That explanation is not technically correct.

AWS EC2 supports both RSA and ED25519 key pairs. The demonstration's failure could have resulted from a key mismatch, key format, SSH configuration or another authentication issue. We cannot conclude that RSA 4096 was unsupported.

AWS documentation confirms the supported key types.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.aws.amazon.com\&sz=32)

Amazon Elastic Compute Cloud+1



## 6. SSH authentication troubleshooting

The instructor spent a considerable amount of time troubleshooting SSH connectivity.

These are the main problems to understand.

| Error                           | Possible reason                  | What to check                                        |
| ------------------------------- | -------------------------------- | ---------------------------------------------------- |
| `Permission denied (publickey)` | Incorrect or unaccepted key      | Username, private/public key pair, `authorized_keys` |
| `Connection timed out`          | Network connectivity problem     | Security group, routing, firewall                    |
| `No route to host`              | No valid network path            | Subnet routes, network ACLs, IP address              |
| `Host key verification failed`  | Host identity verification issue | `known_hosts` entry and server identity              |
| `UNREACHABLE`                   | Ansible cannot connect to target | SSH connectivity and authentication                  |

### Test SSH manually first

Before running a playbook, test the connection:

```
ssh -i ~/.ssh/id_ed25519 ubuntu@172.31.10.20
```

If this does not work, troubleshoot SSH first.

### Useful Ansible inventory configuration

```
[webservers]
web1 ansible_host=172.31.10.20 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

The `ansible_ssh_private_key_file` variable tells Ansible which private key to use.

### SSH key setup for many servers

The instructor also explained how to automate public-key distribution to many servers.

For example, if you have 100 servers:

1. Store their IP addresses in a file.
2. Read the file line by line.
3. Use `ssh-copy-id` to distribute the public key.
4. Test the SSH connection.

This avoids manually copying the key to every machine.

Security note: Do not put passwords directly into inventory files. Use SSH keys or a secure credential-management method. Password-based SSH authentication is possible, but it should be configured carefully.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 7. Install NGINX using an Ansible playbook

This was the main successful remote configuration demonstration.

The instructor created a separate playbook to install NGINX on the remote Ubuntu server.

### What is `become: true`?

The remote user `ubuntu` normally does not have permission to install system packages directly.

Installing NGINX requires elevated privileges.

Ansible provides privilege escalation through `become`.

```
become: true
```

This is equivalent to requesting elevated privileges, commonly through `sudo`.

### NGINX playbook

```
---
- name: Install and configure NGINX
  hosts: webservers
  become: true

  tasks:
    - name: Update apt cache and install NGINX
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start and enable NGINX
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

### Understand the important parameters

| Parameter                 | Meaning                       |
| ------------------------- | ----------------------------- |
| `hosts`                   | Target inventory group        |
| `become: true`            | Use privilege escalation      |
| `ansible.builtin.apt`     | Manage Ubuntu/Debian packages |
| `name: nginx`             | Package to install            |
| `state: present`          | Ensure package is installed   |
| `update_cache: true`      | Update package metadata       |
| `ansible.builtin.service` | Manage system services        |
| `state: started`          | Ensure service is running     |
| `enabled: true`           | Enable service at boot        |

### Execute the playbook

```
ansible-playbook -i inventory.ini nginx-playbook.yml
```

### What happens?

Ansible control node

Runs nginx-playbook.yml

Remote Ubuntu server

SSH connection + privilege escalation

1. Update package cache.
2. Install NGINX if missing.
3. Start NGINX.
4. Enable NGINX at boot.

NGINX configured

### Why is this idempotent?

When you run the same playbook again:

- If NGINX is installed, it does not reinstall unnecessarily.
- If NGINX is running, it does not restart unnecessarily.
- If NGINX is enabled at boot, it does not need to change that setting.

The goal is to maintain the desired state without unnecessary changes.

Important: `state: touch` in the earlier file demonstration can update timestamps on repeated executions. It should not be treated as a perfect example of a no-change second run. The package and service examples above are better illustrations of idempotent configuration.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. Ansible task results and verbosity

The instructor demonstrated how to inspect Ansible execution results.

### Common task statuses

| Status        | Meaning                                 |
| ------------- | --------------------------------------- |
| `ok`          | Task completed; no change was necessary |
| `changed`     | Ansible made a change                   |
| `failed`      | Task execution failed                   |
| `unreachable` | Ansible could not connect to the host   |
| `skipped`     | Task was skipped                        |

Example:

```
PLAY RECAP
web1 : ok=3 changed=2 unreachable=0 failed=0
```

Interpretation:

- `ok=3`: Three tasks completed successfully without changes.
- `changed=2`: Two tasks made changes.
- `unreachable=0`: All targeted hosts were reachable.
- `failed=0`: No task failed.

The play recap may include fact gathering as an additional task. This explains why the instructor saw more successful tasks than the number of tasks explicitly written in the playbook.

### Verbosity levels

Ansible provides verbosity options for troubleshooting.

| Command | Meaning           |
| ------- | ----------------- |
| `-v`    | Level 1 verbosity |
| `-vv`   | Level 2 verbosity |
| `-vvv`  | Level 3 verbosity |
| `-vvvv` | Level 4 verbosity |

Example:

```
ansible-playbook -i inventory.ini nginx-playbook.yml -vvv
```

Use verbosity when investigating connection problems, task failures or unexpected changes.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 9. Modules and collections

The instructor explained that Ansible uses modules to perform different operations.

Examples:

| Module    | Purpose                                         |
| --------- | ----------------------------------------------- |
| `file`    | Create files/directories and manage permissions |
| `copy`    | Copy files                                      |
| `command` | Execute commands without a shell                |
| `shell`   | Execute commands through a shell                |
| `apt`     | Manage Ubuntu/Debian packages                   |
| `service` | Manage services                                 |
| `user`    | Manage users                                    |
| `package` | Generic package-management interface            |

### Why use modules instead of shell commands?

Suppose you want to install NGINX.

You could use:

```
ansible.builtin.shell: apt install nginx -y
```

Or you could use:

```
ansible.builtin.apt:
  name: nginx
  state: present
```

The second option is generally better because the Ansible module understands package state and can avoid unnecessary changes.

### What is a collection?

A collection is a package of related Ansible content, such as:

- Modules.
- Plugins.
- Roles.
- Documentation.

Ansible's built-in modules are available through the `ansible.builtin` collection.

For example:

```
ansible.builtin.service:
```

The instructor also discussed AWS modules. To manage AWS infrastructure, you can install an appropriate AWS collection, such as `amazon.aws`, and configure its dependencies and AWS credentials.

Example installation:

```
ansible-galaxy collection install amazon.aws
```

For AWS resource operations, you also need suitable AWS credentials, IAM permissions and required Python dependencies on the control node.

### Can Ansible configure Jenkins?

Yes.

Examples:

| Requirement                         | Possible approach                            |
| ----------------------------------- | -------------------------------------------- |
| Install Jenkins                     | Package-management module                    |
| Start Jenkins                       | Service module                               |
| Restart Jenkins                     | Service module                               |
| Copy Jenkins configuration          | Copy/template modules                        |
| Execute Jenkins CLI commands        | Command or shell module                      |
| Configure Jenkins-specific settings | Appropriate Jenkins collection/module or API |

The important distinction is that you do not need a special Jenkins collection for every task. Built-in modules can perform many operating-system-level operations.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 10. Can one playbook manage different server groups?

Yes. This was another important question raised during the session.

Imagine you have:

- Web servers.
- Application servers.
- Database servers.

You can create different plays inside one playbook.

```
---
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install NGINX
      ansible.builtin.apt:
        name: nginx
        state: present

- name: Configure application servers
  hosts: appservers
  become: true

  tasks:
    - name: Install Java
      ansible.builtin.apt:
        name: default-jdk
        state: present

- name: Configure database servers
  hosts: dbservers
  become: true

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.apt:
        name: postgresql
        state: present
```

### Visual understanding

One playbook

Play 1

webservers

NGINX

Play 2

appservers

Java

Play 3

dbservers

PostgreSQL

Each play targets its own inventory group.

Correction to the lecture: The instructor suggested that the `ansible-playbook` command takes only one playbook. Current Ansible CLI documentation supports passing multiple playbook files as arguments. However, combining related plays into one playbook is often a clearer approach when they form one workflow.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.ansible.com\&sz=32)

Ansible Core Documentation



## 11. Important commands from this session

| Command                                               | Purpose                        |
| ----------------------------------------------------- | ------------------------------ |
| `ansible --version`                                   | Check Ansible version          |
| `ansible-inventory --list`                            | Display inventory              |
| `ansible-inventory --graph`                           | Display inventory groups       |
| `ansible all -m ping`                                 | Test host connectivity         |
| `ssh-keygen -t ed25519`                               | Generate an SSH key pair       |
| `ssh-copy-id user@host`                               | Copy public key to remote host |
| `ssh user@host`                                       | Test SSH connectivity          |
| `ansible-playbook -i inventory.ini playbook.yml`      | Execute playbook               |
| `ansible-playbook --syntax-check playbook.yml`        | Validate playbook syntax       |
| `ansible-playbook -i inventory.ini playbook.yml -vvv` | Execute with detailed logs     |
| `ansible-doc ansible.builtin.apt`                     | View module documentation      |
| `ansible-galaxy collection install amazon.aws`        | Install AWS collection         |

## 12. Quick revision

| Concept                        | Remember this                                                |
| ------------------------------ | ------------------------------------------------------------ |
| Ad-hoc command                 | One-time task from terminal                                  |
| Playbook                       | YAML automation file                                         |
| Inventory                      | List of servers to manage                                    |
| `-i`                           | Specifies inventory                                          |
| Play                           | Tasks associated with selected hosts                         |
| Task                           | Calls a module                                               |
| Module                         | Performs an operation                                        |
| `become: true`                 | Enables privilege escalation                                 |
| SSH                            | Connects Ansible to Linux managed nodes                      |
| `authorized_keys`              | Stores accepted public keys                                  |
| `ansible_user`                 | Defines remote login username                                |
| `ansible_ssh_private_key_file` | Defines SSH private key path                                 |
| `apt`                          | Manages Ubuntu/Debian packages                               |
| `service`                      | Manages services                                             |
| `changed`                      | Ansible made a change                                        |
| `ok`                           | No change required                                           |
| Collection                     | Packages Ansible modules, plugins, roles and related content |

## 13. Interview questions

1. What is the difference between an ad-hoc command and a playbook?
2. What is an inventory file, and why do we need it?
3. Can we create multiple inventory files?
4. What is the difference between a play and a task?
5. Can one playbook contain multiple plays?
6. Why is YAML indentation important?
7. How do you validate an Ansible playbook?
8. How does Ansible connect to remote EC2 instances?
9. How do you configure passwordless SSH authentication?
10. What is the purpose of `authorized_keys`?
11. What is `become: true`?
12. How do you install NGINX using Ansible?
13. What is the difference between `command` and `shell` modules?
14. What is the difference between a module and a collection?
15. How can Ansible configure Jenkins?
16. How do you troubleshoot an unreachable managed node?

## 14. The mental model

Inventory

Where are my servers?

SSH authentication

Can I securely connect?

Playbook

What should I configure?

Tasks and modules

Perform the required operations

Configured remote servers

Desired state achieved

One-line summary: Inventory identifies the servers, SSH connects to them, and playbooks use modules to configure them.

Original transcript: [DevOps Session — 31 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-10-31_09.29.29.txt?utm_source=chatgpt.com)

Session complete

These notes cover the demonstrations and questions from the session, including the SSH troubleshooting exercise and the successful NGINX installation.