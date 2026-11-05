# DevOps Session — Ansible Modules, Troubleshooting and Automation

Date: 1 November 2026 · Instructor: Rochak Agrawal

Topics: Common Ansible modules, task execution statuses, check mode, diff mode, forks, SSH troubleshooting, privilege escalation, Python dependencies, inventory commands, package installation, Git version control, cron scheduling, Ansible Automation Controller and Linux uptime/load average.

I’ve corrected speech-to-text errors such as “Ensemble” to Ansible, “Pink” to ping, “folks” to forks, “Chrome tab” to crontab, and “known host” to known_hosts. I’ve also clarified a few important technical distinctions, particularly `ignored` vs `rescued`, Ansible's ping module and Linux load average.

## 1. Common Ansible modules

The instructor demonstrated several modules commonly used in day-to-day server administration.

| Module    | Purpose                                         | Example use                                     |
| --------- | ----------------------------------------------- | ----------------------------------------------- |
| `ping`    | Tests Ansible connectivity and module execution | Check whether a host is reachable               |
| `command` | Runs a command without a shell                  | Check memory or uptime                          |
| `shell`   | Runs commands through a shell                   | Use pipes, redirection and variables            |
| `copy`    | Copies files from control node to managed node  | Transfer configuration files                    |
| `file`    | Manages files, directories and permissions      | Create directories, change ownership            |
| `apt`     | Manages Debian/Ubuntu packages                  | Install NGINX                                   |
| `service` | Manages services                                | Start or stop NGINX                             |
| `user`    | Manages user accounts                           | Create or delete users                          |
| `package` | Generic package-management interface            | Install packages across supported distributions |

### 1.1 Ping module

The Ansible ping module checks whether Ansible can connect to the managed host and execute its module successfully.

```
ansible all -i inventory.ini -m ansible.builtin.ping
```

Successful output typically includes:

```
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```



PONG

&#x20;means Ansible successfully communicated with the host.



If a host cannot be reached, the output reports an unreachable error.

Important: Ansible's `ping` module is not the same as the Linux ICMP `ping` command. It tests Ansible connectivity and execution, not simply network packet reachability.

### 1.2 Command module vs shell module

This is an important interview topic.

\| Feature | `command` | `shell` |\
\|---|---|---|\
\| Executes commands | Yes | Yes |\
\| Uses shell interpreter | No | Yes |\
\| Pipes (`|`) | No | Yes |\
\| Redirection (`>`, `>>`) | No | Yes |\
\| Shell variables and expansion | No shell expansion | Yes |\
\| Recommended default | Yes, when sufficient | Only when shell features are required |

Using command:

```
ansible webservers -i inventory.ini \
  -m ansible.builtin.command \
  -a "free -h"
```

This executes `free -h` on the managed server and returns its memory information.

Using shell:

```
ansible webservers -i inventory.ini \
  -m ansible.builtin.shell \
  -a "ps aux | grep nginx"
```

The pipe works because the shell interprets it.

Best practice: Prefer `command` when possible. Use `shell` only when you genuinely need shell functionality.

### 1.3 Copy module

The copy module transfers a file from the Ansible control node to a managed node.

```
ansible webservers -i inventory.ini \
  -m ansible.builtin.copy \
  -a "src=copy.txt dest=/tmp/copy.txt"
```

Remember:

- `src` — source file on the control node.
- `dest` — destination path on the managed node.

### 1.4 File, apt, service and user modules

Examples:

```
# Create a directory
- name: Create application directory
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    mode: "0755"

# Install NGINX
- name: Install NGINX
  ansible.builtin.apt:
    name: nginx
    state: present

# Start NGINX
- name: Start NGINX
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: true

# Create a user
- name: Create deployment user
  ansible.builtin.user:
    name: deploy
    state: present
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 2. Ansible task execution statuses

When you run an Ansible playbook, each task returns a status. These statuses tell you what happened on the managed host.

| Status        | Meaning                                                 | Example                               |
| ------------- | ------------------------------------------------------- | ------------------------------------- |
| `ok`          | Task succeeded; no change was required                  | NGINX is already installed            |
| `changed`     | Task succeeded and changed something                    | NGINX was installed                   |
| `failed`      | Task execution failed                                   | Invalid file path or package error    |
| `unreachable` | Ansible could not connect to the host                   | SSH timeout                           |
| `skipped`     | Task was not executed because its condition was not met | A `when` condition evaluated to false |
| `ignored`     | A failure was allowed to continue using `ignore_errors` | An expected non-critical task failure |
| `rescued`     | A failure was handled by a `rescue` block               | Recovery task ran after a failure     |

### 2.1 `ok` vs `changed`

Suppose your playbook installs NGINX.

First run:

```
TASK [Install NGINX]
changed: [web1]
```

NGINX was missing, so Ansible installed it.

Second run:

```
TASK [Install NGINX]
ok: [web1]
```

NGINX already exists, so no change was necessary.

This is an example of idempotency.

### 2.2 Skipped status

A task can be skipped when a condition is not satisfied.

Example:

```
- name: Create development user
  ansible.builtin.user:
    name: devuser
    state: present
  when: environment == "dev"
```

If `environment` is `prod`, this task is skipped.

### 2.3 Ignored vs rescued — important distinction

The instructor discussed these statuses together, but they are not interchangeable.

Ignored failure:

```
- name: Check application endpoint
  ansible.builtin.command:
    cmd: curl -f http://localhost:8080
  register: health_check
  ignore_errors: true
```

If the command fails, Ansible records the failure as ignored and continues to subsequent tasks.

Rescued failure:

```
- name: Application deployment
  block:
    - name: Run deployment
      ansible.builtin.command:
        cmd: /opt/myapp/deploy.sh

  rescue:
    - name: Report deployment failure
      ansible.builtin.debug:
        msg: "Deployment failed. Recovery actions are running."

  always:
    - name: Deployment cleanup
      ansible.builtin.debug:
        msg: "Cleanup step executed."
```

Here:

- `block` contains the main tasks.
- `rescue` runs when a task in the block fails.
- `always` runs after the block/rescue processing, whether the block succeeds or fails.

How failure handling works

Execute task

Success

Continue normally

Failure

Handle the error

Rescue block (if configured)

Always block (if configured)

Real-world example from the lecture: NGINX may install and start successfully, but an HTTP check may fail because the AWS security group does not allow port 80. If that failure is expected, you may choose to handle it. However, ignoring the failure does not fix the security group or make the application accessible.

Important: Do not blindly ignore failures. Investigate why they occurred and decide whether they are genuinely acceptable.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Syntax check, dry run and diff mode

Before making changes to servers, Ansible provides ways to inspect what a playbook is expected to do.

| Option              | Purpose                                               |
| ------------------- | ----------------------------------------------------- |
| `--syntax-check`    | Checks playbook syntax                                |
| `--check`           | Runs in check mode to predict changes                 |
| `--diff`            | Shows differences for supported file/template changes |
| `-v`, `-vv`, `-vvv` | Increases output verbosity                            |

### Syntax check

```
ansible-playbook -i inventory.ini site.yml --syntax-check
```

This checks whether the playbook is syntactically valid. It does not execute the tasks.

### Dry run / check mode

```
ansible-playbook -i inventory.ini site.yml --check
```

Check mode attempts to predict changes without applying them.

Example:

Suppose your playbook copies a file from the control node to a managed node.

```
- name: Copy configuration
  ansible.builtin.copy:
    src: app.conf
    dest: /tmp/app.conf
```

With `--check`, Ansible can report that the file would be changed without actually copying it.

### Diff mode

```
ansible-playbook -i inventory.ini site.yml --check --diff
```

This attempts to show what content would change.

You can also run:

```
ansible-playbook -i inventory.ini site.yml --diff
```

This executes the playbook and displays differences for supported modules.

Check mode vs diff mode

`--check`

What would change?

`--diff`

What content differs?

Important limitations:

- Check mode is a simulation, not a guarantee that the actual run will succeed.
- Some modules do not fully support check mode.
- Tasks depending on registered results from earlier tasks may behave differently in check mode.
- Diff output may expose sensitive information. Use `no_log: true` for tasks handling secrets when appropriate.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 4. Forks — parallel execution

Ansible can manage multiple servers simultaneously.

The `forks` setting controls how many hosts Ansible can process concurrently.

Default: 5

Example:

Suppose you have 100 servers.

- With 5 forks, Ansible processes up to 5 hosts concurrently.
- With 10 forks, Ansible can process up to 10 hosts concurrently.
- With 20 forks, Ansible can process up to 20 hosts concurrently.

### Configure forks permanently

In `ansible.cfg`:

```
[defaults]
forks = 10
```

### Configure forks for one execution

```
ansible-playbook -i inventory.ini site.yml -f 10
```

### How it works

100 managed servers

Ansible control node

forks = 5

Host 1

Host 2

Host 3

Host 4

Host 5

Up to 5 hosts processed concurrently in this batch

Next batch of hosts

### How should you choose the forks value?

The instructor highlighted two considerations:

1. What work is the playbook performing?
2. What is the capacity of the Ansible control node?

For example:

- Copying a small file is a lightweight operation.
- Installing packages on many servers may consume more resources.
- Running complex automation on 100 servers simultaneously can overload a small control node.

Remember: More forks can improve parallelism, but they also increase resource consumption. Also, forks control concurrency, not the total number of servers Ansible can manage.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 5. Common Ansible troubleshooting errors

The instructor revisited several errors from the previous practical session.

| Error                           | Likely cause                          | What to check                     |
| ------------------------------- | ------------------------------------- | --------------------------------- |
| `Permission denied (publickey)` | Incorrect or missing SSH key          | Username, public key, private key |
| `Host key verification failed`  | Host identity not trusted             | `known_hosts`                     |
| `Missing sudo password`         | Privilege escalation needs a password | Sudo permissions and `-K`         |
| `Python not found`              | Python missing on managed node        | Install Python or bootstrap it    |
| `No hosts matched`              | Incorrect inventory group/pattern     | Inventory group name              |
| `Connection timed out`          | Network or firewall issue             | Security group, port 22, routing  |
| `ansible: command not found`    | Ansible not installed or not on PATH  | Installation and PATH             |

### 5.1 Permission denied (publickey)

Possible reasons:

- Public key was not copied to the remote server.
- Wrong SSH username.
- Wrong private key.
- Incorrect SSH file permissions.
- SSH key format or compatibility problem.

Test manually:

```
ssh -i ~/.ssh/id_ed25519 ubuntu@172.31.10.20
```

### 5.2 Host key verification failed

SSH uses a `known_hosts` file to remember the identity of previously contacted servers.

Common location:

```
~/.ssh/known_hosts
```

Connect manually once:

```
ssh ubuntu@172.31.10.20
```

Review the host fingerprint and accept it only if you have verified that it belongs to the intended server.

If a server was legitimately rebuilt and its host key changed, remove the stale entry and verify the new fingerprint before accepting it:

```
ssh-keygen -R 172.31.10.20
```

Security warning: Do not permanently disable host key checking merely to avoid this error. Host key verification helps protect against connecting to an impersonated server.

### 5.3 Missing sudo password

This is related to Linux privilege escalation.

Ansible connects using the normal remote user, such as `ubuntu`. That user must be authorized to run the required commands with sudo.

Ansible supports:

```
ansible-playbook -i inventory.ini site.yml -b
```

Here:

- `-b` means become.
- It requests privilege escalation, usually using sudo.

If a sudo password is required:

```
ansible-playbook -i inventory.ini site.yml -b -K
```

`-K` prompts for the become password.

In a playbook:

```
- name: Configure web servers
  hosts: webservers
  become: true

  tasks:
    - name: Install NGINX
      ansible.builtin.apt:
        name: nginx
        state: present
```

Important: `become: true` does not grant a user sudo permission. The remote account must already be authorized to escalate privileges. If it cannot use sudo, you need to fix the server's sudo configuration or use an appropriately authorized account.

### 5.4 Python not found

Most Ansible POSIX modules require Python on the managed node.

Check:

```
python3 --version
```

If missing, install Python using the operating system's package manager or bootstrap it using Ansible's `raw` module, which does not require Python on the target.

Example for Ubuntu:

```
ansible webservers -i inventory.ini \
  -m ansible.builtin.raw \
  -a "sudo apt-get update && sudo apt-get install -y python3"
```

### 5.5 No hosts matched

Suppose your inventory contains:

```
[webservers]
172.31.10.20
```

But your playbook contains:

```
hosts: webserver
```

The group names do not match.

Correct it:

```
hosts: webservers
```

### 5.6 Connection timed out

The instructor demonstrated this by changing the EC2 security group and removing the SSH rule.

Ansible could no longer connect to the server.

For Linux EC2 servers, ensure:

- Inbound TCP port 22 is allowed from the control node.
- The control node can reach the managed node.
- Network ACLs and routing permit the connection.
- The correct IP address is used.

SSH connectivity checklist

1\. Correct IP address?

2\. Correct SSH username and key?

3\. TCP port 22 allowed?

4\. Routing and network ACLs correct?

5\. Host key verified?

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 6. Inventory troubleshooting and useful commands

The instructor demonstrated how to inspect an inventory and visualize its groups.

### 6.1 List hosts matching a pattern

```
ansible webservers -i inventory.ini --list-hosts
```

This shows which hosts Ansible will target.

To list all hosts:

```
ansible all -i inventory.ini --list-hosts
```

This is useful when you get:

```
Could not match supplied host pattern
```

### 6.2 Display inventory in JSON

```
ansible-inventory -i inventory.ini --list
```

Displays the inventory structure and host variables.

### 6.3 Display inventory as a graph

```
ansible-inventory -i inventory.ini --graph
```

Example inventory:

```
[webservers:children]
web_staging
web_production

[web_staging]
web1.example.com

[web_production]
web2.example.com

[dbservers]
db1.example.com
```

The graph helps you understand parent groups, child groups and hosts.

Inventory group structure

all

webservers

web_staging

web1.example.com

web_production

web2.example.com

dbservers

db1.example.com

### Three useful inventory commands

| Command                     | What it shows            |
| --------------------------- | ------------------------ |
| `ansible-inventory --list`  | Inventory in JSON        |
| `ansible-inventory --graph` | Group hierarchy          |
| `ansible ... --list-hosts`  | Hosts matching a pattern |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 7. Installing Ansible — APT vs DNF vs PIP vs PIPX

The instructor explained why Ansible was installed using APT during the practical session and why PIP is another option.

### Understand the package managers

| Package manager | Used for                                                               |
| --------------- | ---------------------------------------------------------------------- |
| APT             | Debian/Ubuntu system packages                                          |
| DNF             | Fedora and modern RHEL-family systems, including Amazon Linux 2023     |
| YUM             | Older RHEL/CentOS systems; compatibility command on some newer systems |
| PIP             | Python packages                                                        |
| PIPX            | Python command-line applications installed in isolated environments    |

### APT installation

On Ubuntu:

```
sudo apt update
sudo apt install ansible -y
```

### DNF installation

On Amazon Linux 2023, Ansible may be available through configured repositories:

```
sudo dnf install ansible -y
```

Package availability depends on the enabled repositories.

### PIP installation

PIP is Python's package installer. It can install Python packages, including Ansible.

However, modern Linux distributions may mark their system Python installation as an externally managed environment. Installing packages directly into it with PIP can cause conflicts with system-managed packages.

A safer approach is to use a virtual environment:

```
python3 -m venv ~/ansible-venv

source ~/ansible-venv/bin/activate

python -m pip install --upgrade pip

python -m pip install ansible
```

### PIPX installation

PIPX installs Python command-line applications in isolated environments.

Conceptually:

System Python

PIP

Installs packages into a selected Python environment

PIPX

Isolates Python CLI applications

A common PIPX installation workflow on Ubuntu is:

```
sudo apt update
sudo apt install pipx -y

pipx ensurepath

pipx install --include-deps ansible
```

You may need to open a new terminal or reload your shell configuration after `pipx ensurepath`.

### Which should you choose?

- Use APT or DNF when you want a distribution-managed package and that package version meets your needs.
- Use PIP inside a virtual environment when you need a particular Python package version or isolated development environment.
- Use PIPX for isolated command-line applications such as Ansible.

Important correction: PIP is not a replacement for APT or DNF. They manage different package ecosystems. APT/DNF manage operating-system packages; PIP manages Python packages.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. Why playbooks should be version-controlled

The instructor discussed an important real-world DevOps practice: storing Ansible playbooks in Git.

A playbook is code. It should be maintained like application code, Terraform files, Dockerfiles and Kubernetes manifests.

### What happens without version control?

Imagine two engineers independently maintain playbooks for the same production server.

- Engineer A installs NGINX version X.
- Engineer B changes the playbook to install version Y.
- Both run their playbooks.
- The server configuration can change unexpectedly depending on which version runs last.

This can cause configuration drift and inconsistent results.

### Recommended approach

Central Git repository

Shared automation code

Engineer A

Pulls latest playbook

Engineer B

Pulls latest playbook

Consistent server configuration

Reviewed and tracked changes

### What should be version-controlled?

| Artifact                    | Why?                               |
| --------------------------- | ---------------------------------- |
| Application source code     | Track application changes          |
| Jenkinsfile                 | Track CI/CD pipeline changes       |
| Dockerfile                  | Track container build instructions |
| Kubernetes manifests        | Track deployment configuration     |
| Terraform files             | Track infrastructure definitions   |
| Ansible playbooks and roles | Track server configuration         |

Example repository:

```
platform-engineering/
├── ansible/
│   ├── inventories/
│   ├── playbooks/
│   └── roles/
├── terraform/
├── docker/
├── kubernetes/
└── jenkins/
```

An organization may maintain a central platform repository or store deployment automation alongside each application. The right structure depends on ownership and deployment workflows.

Key takeaway: Version control helps teams know which playbook version is being used, who changed it, what changed and how to roll back.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 9. Scheduling Ansible playbooks using cron

A student asked whether Ansible commands could run automatically every day at 9 AM.

The instructor explained that this is not an Ansible-specific feature. Linux provides a scheduling mechanism called cron.

Cron can schedule:

- Linux commands.
- Shell scripts.
- Ansible ad-hoc commands.
- Ansible playbooks.

### Useful cron commands

```
# List current user's scheduled jobs
crontab -l

# Edit current user's scheduled jobs
crontab -e
```

### Cron syntax

Cron uses five fields:

```
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week
│ │ │ └──── Month
│ │ └────── Day of month
│ └──────── Hour
└────────── Minute
```

| Field        | Allowed values                                        |
| ------------ | ----------------------------------------------------- |
| Minute       | 0–59                                                  |
| Hour         | 0–23                                                  |
| Day of month | 1–31                                                  |
| Month        | 1–12                                                  |
| Day of week  | 0–7 (Sunday is 0 or 7 on common cron implementations) |

### Common cron examples

| Schedule                        | Cron expression |
| ------------------------------- | --------------- |
| Every minute                    | `* * * * *`     |
| Every hour                      | `0 * * * *`     |
| Every day at midnight           | `0 0 * * *`     |
| Every day at 9 AM               | `0 9 * * *`     |
| Every day at 9 PM               | `0 21 * * *`    |
| Every Monday at 9 PM            | `0 21 * * 1`    |
| Every Monday and Friday at 9 PM | `0 21 * * 1,5`  |
| Every Monday in January at 9 PM | `0 9 * 1 1`     |

### Schedule an Ansible playbook

Suppose your playbook is stored at:

```
/home/ubuntu/ansible/playbooks/healthcheck.yml
```

Edit cron:

```
crontab -e
```

Add:

```
0 9 * * * /usr/bin/ansible-playbook -i /home/ubuntu/ansible/inventory.ini /home/ubuntu/ansible/playbooks/healthcheck.yml >> /home/ubuntu/ansible/healthcheck.log 2>&1
```

This runs the playbook every day at 9 AM.

Why use absolute paths?

Cron runs with a limited environment and may not use the same PATH or working directory as your interactive terminal. Use absolute paths for executables, inventory files, playbooks and log files.

### Scheduling workflow

Cron scheduler

Runs at the configured time

Ansible playbook

Executes health checks

Managed servers

Results saved to log file

Important: The cron job runs as the user whose crontab contains it. Ensure that user has the correct SSH key access, inventory permissions, Python environment and sudo privileges.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 10. Ansible Tower and Automation Controller

The instructor introduced Ansible Tower as an enterprise management solution for Ansible.

The terminology has evolved, so it is important to know the current names.

| Name                          | Meaning                                                                 |
| ----------------------------- | ----------------------------------------------------------------------- |
| Ansible Core                  | Open-source automation engine                                           |
| Ansible Tower                 | Earlier commercial automation management platform                       |
| AWX                           | Open-source upstream project related to Ansible Tower                   |
| Ansible Automation Controller | Enterprise automation controller in Red Hat Ansible Automation Platform |

### Why use an automation controller?

Instead of manually executing playbooks from a terminal, an organization can use a centralized web interface.

Typical capabilities include:

- Web-based job execution.
- Centralized inventory management.
- Credential management.
- Role-based access control.
- Job scheduling.
- Execution history and logs.
- Integration with source control.
- Automation workflow management.

### Comparison

Ansible Core

Run automation using terminal commands and playbooks.

Example: `ansible-playbook site.yml`

Automation Controller

Manage automation centrally through a web interface and API.

Example: Schedule jobs, manage credentials and track execution.

Important: Ansible Tower is not simply another name for Ansible Core. Also, Ansible Automation Controller is part of Red Hat Ansible Automation Platform, while AWX is its upstream open-source project.

### Other scheduling tools

The instructor briefly mentioned workflow schedulers such as Apache Airflow.

Airflow is widely used to orchestrate workflows and data pipelines. It can also coordinate other jobs, but it is not a direct replacement for Ansible or its automation controller.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 11. Linux uptime command

The instructor clarified that `uptime` is a Linux command, not an Ansible feature.

Run:

```
uptime
```

Example output:

```
09:30:00 up 1 hour, 40 min, 1 user, load average: 0.05, 0.10, 0.09
```

### Understand the output

| Output              | Meaning                      |
| ------------------- | ---------------------------- |
| `09:30:00`          | Current system time          |
| `up 1 hour, 40 min` | Time since the system booted |
| `1 user`            | Number of logged-in users    |
| `0.05`              | 1-minute load average        |
| `0.10`              | 5-minute load average        |
| `0.09`              | 15-minute load average       |

Important: Uptime tells you how long the system has been running. It does not directly measure CPU utilization.

## 12. Linux load average — important interview topic

This was one of the most important interview questions discussed in the session.

The instructor demonstrated how to interpret load average and why the number of CPU cores matters.

### What is load average?

Load average represents the average number of tasks that are:

- Runnable — running or waiting for CPU time.
- In uninterruptible sleep — commonly waiting for I/O operations.

It is measured over three time periods:

| Value         | Time period |
| ------------- | ----------- |
| First number  | 1 minute    |
| Second number | 5 minutes   |
| Third number  | 15 minutes  |

For example:

```
load average: 1.00, 1.00, 1.00
```

This means the average load was approximately 1 over each of those three periods.

Critical correction: Load average is not the same as CPU utilization percentage. It includes tasks waiting for certain I/O operations too.

### Why do CPU cores matter?

Consider two machines, both reporting a load average of 1.

- Machine A has 1 logical CPU.
- Machine B has 4 logical CPUs.

The same load number can mean different things on these machines.

Machine A — 1 logical CPU

Load average: 1

At capacity, approximately

Load-to-logical-CPU ratio: 1 ÷ 1 = 1

Machine B — 4 logical CPUs

Load average: 1

Lower relative load

Load-to-logical-CPU ratio: 1 ÷ 4 = 0.25

This ratio is a useful rough indicator, not an exact CPU utilization measurement. A load of 1 on a single-CPU machine does not prove CPU utilization was exactly 100%.

### How to check CPU information

The instructor demonstrated:

```
lscpu
```

This displays:

- CPU architecture.
- Number of CPUs.
- CPU sockets.
- Cores per socket.
- Threads per core.

Other useful commands:

```
nproc
```

Displays the number of processing units available to the current process.

```
top
```

Displays live system processes, CPU and memory usage, and load averages.

```
uptime
```

Displays uptime and load averages.

### Interview example

Question: A Linux server has a load average of 4. Is the server overloaded?

Answer: We cannot conclude that from the load average alone. First, we need to know the number of logical CPUs and examine the load trend. We should also check CPU utilization, I/O wait and running processes.

| Logical CPUs | Load average | Rough interpretation                                                |
| ------------ | ------------ | ------------------------------------------------------------------- |
| 1            | 4            | High relative load                                                  |
| 2            | 4            | High relative load                                                  |
| 4            | 4            | Around one runnable/uninterruptible task per logical CPU on average |
| 8            | 4            | Lower relative load                                                 |

Remember: Always compare load average with the number of logical CPUs. Do not automatically convert load average into a CPU percentage.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 13. Quick revision — commands from the session

| Command                                              | Purpose                            |
| ---------------------------------------------------- | ---------------------------------- |
| `ansible all -m ping`                                | Test Ansible connectivity          |
| `ansible all -m command -a "free -h"`                | Check memory on managed hosts      |
| `ansible all -m shell -a "ps aux \\| grep nginx"`    | Execute a shell command            |
| `ansible all -m copy -a "src=a.txt dest=/tmp/a.txt"` | Copy a file                        |
| `ansible-playbook --syntax-check site.yml`           | Validate playbook syntax           |
| `ansible-playbook --check site.yml`                  | Check mode / dry run               |
| `ansible-playbook --diff site.yml`                   | Show supported file changes        |
| `ansible-playbook -f 10 site.yml`                    | Run with 10 forks                  |
| `ansible-playbook -b site.yml`                       | Enable privilege escalation        |
| `ansible-playbook -b -K site.yml`                    | Prompt for become password         |
| `ansible-inventory -i inventory.ini --list`          | Display inventory                  |
| `ansible-inventory -i inventory.ini --graph`         | Display inventory hierarchy        |
| `ansible all -i inventory.ini --list-hosts`          | List matching hosts                |
| `ssh-keygen -R IP_ADDRESS`                           | Remove a known-host entry          |
| `python3 --version`                                  | Check Python version               |
| `pipx install --include-deps ansible`                | Install Ansible using PIPX         |
| `crontab -l`                                         | List scheduled jobs                |
| `crontab -e`                                         | Edit scheduled jobs                |
| `uptime`                                             | Check uptime and load average      |
| `lscpu`                                              | Check CPU information              |
| `nproc`                                              | Check available processing units   |
| `top`                                                | Monitor system processes and usage |

## 14. Interview questions

1. What is the difference between `command` and `shell` modules?
2. What is the difference between `ok` and `changed`?
3. What is the difference between `ignored` and `rescued`?
4. What is the purpose of `ignore_errors`?
5. What is a rescue block?
6. What is Ansible check mode?
7. What is the difference between `--check` and `--diff`?
8. What are forks in Ansible?
9. How do you troubleshoot `Permission denied (publickey)`?
10. What is `known_hosts` and why is it important?
11. What is the purpose of `become: true`?
12. What is the difference between `-b` and `-K`?
13. Why does Ansible need Python on managed nodes?
14. How do you troubleshoot `No hosts matched`?
15. How do you display an inventory hierarchy?
16. What is the difference between APT, DNF and PIP?
17. What is the difference between PIP and PIPX?
18. Why should Ansible playbooks be stored in Git?
19. How can you schedule a playbook to run every day at 9 AM?
20. What is Ansible Tower? What is Automation Controller?
21. What does the Linux `uptime` command display?
22. What is load average?
23. Why must load average be compared with logical CPU count?
24. Is load average the same as CPU utilization?

## 15. Final mental model

Ansible modules

Perform server operations

Task statuses

Understand what happened

Check mode, diff and forks

Validate changes and control execution

Troubleshooting and privilege escalation

Resolve connectivity and permission issues

Git + Cron + Automation Controller

Manage, share and schedule automation

One-line summary: Ansible automates server operations through modules and playbooks; Git manages the automation code, cron schedules execution, and monitoring commands such as `uptime`, `top` and `lscpu` help understand server health.

Original transcript: [DevOps Session — 1 November 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-11-01_09.34.39.txt?utm_source=chatgpt.com)

Session complete

These notes cover the full session, including the practical demonstrations, troubleshooting discussion, scheduling example and Linux load-average interview question.