# DevOps Session — Ansible Modules, Troubleshooting, Execution & Scheduling Notes

Date: 1 November 2026\
Instructor: Rochak Agrawal\
Focus: Ansible ad-hoc commands, commonly used modules, task execution results, dry runs, SSH troubleshooting, privilege escalation, inventory management, scheduling and Linux load average.

Notes are condensed from the complete transcript. Participant discussions have been removed, speech-to-text errors corrected and technical inaccuracies clarified.

## 1. Commonly Used Ansible Modules

The session started with practical demonstrations of commonly used Ansible modules.

| Module    | Purpose                                      | Example use                           |
| --------- | -------------------------------------------- | ------------------------------------- |
| `ping`    | Test Ansible connectivity                    | Check whether a managed host responds |
| `command` | Execute a command without shell processing   | Run `free -h`                         |
| `shell`   | Execute commands through a shell             | Use pipes, redirects and variables    |
| `copy`    | Copy files from control node to managed node | Transfer configuration files          |
| `file`    | Manage files, directories and permissions    | Create directories                    |
| `apt`     | Manage Debian/Ubuntu packages                | Install NGINX                         |
| `service` | Manage services                              | Start or stop NGINX                   |
| `user`    | Manage user accounts                         | Create or delete users                |

### Ansible ping module

```
ansible all -i inventory.ini -m ansible.builtin.ping
```

Expected successful response:

```
web1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Important: Ansible's `ping` module does not use ICMP like the Linux `ping` command. It checks whether Ansible can connect to the host and execute its module.

### Command vs Shell module

This is an important interview topic.

Command module:

```
ansible all -i inventory.ini \
  -m ansible.builtin.command \
  -a "free -h"
```

Use it for ordinary commands that do not require shell features.

Shell module:

```
ansible all -i inventory.ini \
  -m ansible.builtin.shell \
  -a "ps aux | grep nginx"
```

Use it when you need:

- Pipes (`|`)
- Redirection (`>` or `>>`)
- Shell variables
- Shell operators

| Command                            | Shell                   |
| ---------------------------------- | ----------------------- |
| Does not invoke a shell by default | Invokes a shell         |
| Safer for simple commands          | Supports shell features |
| Preferred when sufficient          | Use only when necessary |

Best practice: Prefer `command` over `shell` unless you specifically need shell functionality.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 2. Copy Module — Transfer Files to Remote Servers

The instructor demonstrated transferring a file from the control node to a managed node.

```
ansible webservers -i inventory.ini \
  -m ansible.builtin.copy \
  -a "src=copy.txt dest=/tmp/copy.txt"
```

Here:

| Parameter | Meaning                              |
| --------- | ------------------------------------ |
| `src`     | Source file on the control node      |
| `dest`    | Destination path on the managed node |

The copy module is useful for transferring configuration files, scripts and application-related files.

Remember: The source is normally on the control node; the destination is on the managed node.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 3. Understanding Ansible Task Results

When you execute an Ansible command or playbook, you see different statuses.

| Status        | Meaning                               | Example                             |
| ------------- | ------------------------------------- | ----------------------------------- |
| `ok`          | Desired state already exists          | NGINX is already installed          |
| `changed`     | System was modified                   | NGINX was installed                 |
| `failed`      | Task execution failed                 | Invalid destination path            |
| `unreachable` | Ansible cannot connect                | SSH timeout                         |
| `skipped`     | Task condition was not satisfied      | Dev-only task skipped on production |
| `ignored`     | Failure was explicitly ignored        | Task uses `ignore_errors: true`     |
| `rescued`     | Failure was handled by a rescue block | Recovery task executed              |

### Understand `ok` and `changed`

Suppose your playbook installs NGINX.

First execution:

```
TASK [Install NGINX]
changed: [web1]
```

NGINX was not installed, so Ansible installed it.

Second execution:

```
TASK [Install NGINX]
ok: [web1]
```

NGINX is already installed, so Ansible does not need to change anything.

This demonstrates idempotency.

### What is skipped?

You can use conditions to execute tasks only in specific environments.

Example:

```
- name: Create development user
  ansible.builtin.user:
    name: devuser
    state: present
  when: "'dev' in group_names"
```

This task runs only on hosts belonging to the `dev` group. On other hosts, it is skipped.

### What is ignored?

Sometimes you deliberately allow a task failure without stopping the play.

```
- name: Run optional health check
  ansible.builtin.command: /usr/local/bin/health-check
  ignore_errors: true
```

The play continues even if the task fails.

Warning: Do not ignore errors simply to make a playbook appear successful. Investigate the failure and ignore it only when the failure is genuinely acceptable.

### What is rescued?

Ansible also supports error handling using `block`, `rescue` and `always`.

```
- name: Application deployment
  block:
    - name: Deploy application
      ansible.builtin.command: /opt/deploy.sh

  rescue:
    - name: Record deployment failure
      ansible.builtin.debug:
        msg: "Deployment failed. Recovery is required."

  always:
    - name: Display completion message
      ansible.builtin.debug:
        msg: "Deployment attempt finished."
```

- `block`: Main operations.
- `rescue`: Runs when a task in the block fails.
- `always`: Runs regardless of success or failure.

Technical distinction: `ignored` and `rescued` are not interchangeable. `ignore_errors` ignores an eligible task failure; `rescue` executes recovery logic for a failed block.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 4. Dry Run — Check Before Applying Changes

What if you want to know what a playbook will do without actually changing the server?

Ansible provides check mode, also called dry-run mode.

```
ansible-playbook -i inventory.ini nginx.yml --check
```

This predicts changes where supported by the modules.

### Difference between normal execution and dry run

| Normal execution            | Dry run                          |
| --------------------------- | -------------------------------- |
| Executes tasks              | Simulates supported changes      |
| Modifies the managed system | Normally avoids applying changes |
| Shows actual results        | Shows predicted results          |

### View configuration differences

Use `--diff`:

```
ansible-playbook -i inventory.ini nginx.yml --diff
```

Combine both:

```
ansible-playbook -i inventory.ini nginx.yml --check --diff
```

This helps preview changes to files and templates.

Important: Check mode is not a perfect simulation. Some modules do not fully support it, and tasks dependent on previous changes may not behave exactly as they would during a normal run.

### Syntax check vs Dry run

| Command             | Purpose                                 |
| ------------------- | --------------------------------------- |
| `--syntax-check`    | Checks playbook syntax                  |
| `--check`           | Predicts supported changes              |
| `--diff`            | Displays file/configuration differences |
| `-v`, `-vv`, `-vvv` | Increases execution detail              |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 5. Forks — Parallel Execution

Ansible can execute tasks on multiple managed nodes simultaneously.

The setting controlling parallel execution is called forks.

Default: 5

Suppose you have 20 servers.

- With 5 forks, Ansible can process up to 5 hosts concurrently.
- With 10 forks, it can process up to 10 hosts concurrently.

### Configure forks permanently

In `ansible.cfg`:

```
[defaults]
forks = 10
```

### Configure forks for one execution

```
ansible-playbook -i inventory.ini nginx.yml -f 10
```

### What determines the appropriate fork value?

- Number of managed servers.
- Control node CPU and memory.
- Network capacity.
- Complexity of tasks.
- Resource consumption of the playbook.

Increasing forks does not always make execution faster. A small control node running resource-intensive tasks may struggle with too many parallel processes.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 6. SSH Troubleshooting

The instructor revisited the SSH errors encountered while connecting the control node to remote EC2 instances.

### Common SSH errors

| Error                         | Possible cause                                  | What to check                       |
| ----------------------------- | ----------------------------------------------- | ----------------------------------- |
| Permission denied (publickey) | Wrong key or username                           | SSH key and remote user             |
| Host key verification failed  | Host is not trusted                             | `known_hosts`                       |
| Connection timed out          | Network or firewall issue                       | Security group, routing and port 22 |
| Missing sudo password         | User cannot escalate or password is unavailable | Sudo permissions and authentication |
| Python not found              | Python unavailable on managed node              | Python installation and interpreter |
| No host matched               | Incorrect inventory group                       | Inventory and host pattern          |

### Error 1: Permission denied (publickey)

Check:

1. Is the public key installed on the managed node?
2. Are you using the correct SSH username?
3. Does the private key match the public key?
4. Are SSH file permissions correct?
5. Is the private key in a supported format?

### Error 2: Host key verification failed

SSH maintains a file called:

```
~/.ssh/known_hosts
```

It stores the identities of previously trusted SSH servers.

Connect once interactively and verify the host fingerprint:

```
ssh ubuntu@MANAGED_SERVER_IP
```

If the fingerprint is correct, accept it. For managed automation environments, you can also populate `known_hosts` through a controlled process.

Avoid disabling host-key verification as a permanent workaround because it weakens protection against server impersonation.

### Error 3: Connection timed out

For AWS EC2, verify:

- SSH is running on the destination.
- Security group allows inbound TCP port 22 from the control node.
- Network ACLs and routes permit communication.
- The control node can reach the destination IP.

[Installing Jenkins on AWS EC2](https://images.openai.com/static-rsc-4/YF7TrSCtkM84EgvmCNIHhqgA8vH8H8p2ypiB21QiDVZ7MK7d-DlBbjlgTxQfkGhOOOpxOYbtPVYxRJ_xd8Tptvs7XXjATkRjVXel06b110u5KQWChr__FSwJAme1hFftBknwKDKJbQ5qii7-ysKUctsrxCmK5ioLSHgNeI4BTKg?purpose=inline)

[cloudzenia.com](https://cloudzenia.com/blog/installing-jenkins-on-aws-ec2/?utm_source=chatgpt.com)

The instructor demonstrated that removing the security group rule for port 22 caused Ansible connectivity to fail. Restoring the rule fixed the issue.

### Error 4: Missing sudo password

The SSH user may connect successfully but may not have permission to become root.

Check whether the user has appropriate sudo permissions.

On Linux, sudo access is controlled through `/etc/sudoers` and configuration files under `/etc/sudoers.d/`.

Use:

```
sudo visudo
```

Do not casually edit `/etc/sudoers` using a normal text editor. `visudo` validates the syntax before saving.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+3

## 7. Privilege Escalation — `become`

Ansible normally connects using the configured SSH user.

For example:

```
ubuntu@192.168.1.10
```

But this user may not have permission to install packages or modify protected system directories.

Ansible provides privilege escalation.

### Ad-hoc command

```
ansible webservers -i inventory.ini \
  -m ansible.builtin.command \
  -a "whoami" \
  -b
```

The `-b` flag enables privilege escalation, usually through sudo.

### Playbook

```
---
- name: Install NGINX
  hosts: webservers
  become: true

  tasks:
    - name: Install NGINX
      ansible.builtin.apt:
        name: nginx
        state: present
```

### What if sudo requires a password?

Use:

```
ansible-playbook -i inventory.ini nginx.yml --ask-become-pass
```

Or:

```
ansible-playbook -i inventory.ini nginx.yml -K
```

`become` does not automatically grant privileges. The remote user must be authorized to use sudo.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. Python Requirement on Managed Nodes

Ansible is written in Python, and most Ansible modules require Python on the managed node.

The control node and managed node have different requirements:

| Machine            | Requirement                                    |
| ------------------ | ---------------------------------------------- |
| Control node       | Ansible and its dependencies                   |
| Managed Linux node | Python for most modules, plus SSH connectivity |

Some cloud images already have Python installed. Others may not.

Check the managed node:

```
python3 --version
```

If missing, install it using the operating system's package manager.

For example, Ubuntu:

```
sudo apt update
sudo apt install python3 -y
```

For Amazon Linux 2023:

```
sudo dnf install python3 -y
```

Important: Not every Ansible operation requires Python on the managed node. However, most standard modules do. Ansible has special raw execution capabilities that can help bootstrap Python on a host where it is missing.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 9. Inventory Management — List Hosts and Groups

The instructor demonstrated how to inspect inventory information.

### List all hosts

```
ansible-inventory -i inventory.ini --list
```

### Display inventory hierarchy

```
ansible-inventory -i inventory.ini --graph
```

The graph helps you understand how servers are organized into groups and subgroups.

Example inventory:

```
[web_staging]
web1 ansible_host=192.168.1.10

[web_production]
web2 ansible_host=192.168.1.11

[webservers:children]
web_staging
web_production

[dbservers]
db1 ansible_host=192.168.1.20
```

Visual structure:

\#chatgpt-mermaid-\_r_for\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_for\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_for\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_for\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_for\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_for\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_for\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_for\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_for\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_for\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_for\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_for\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_for\_ p{margin:0;}#chatgpt-mermaid-\_r_for\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_for\_ .label text,#chatgpt-mermaid-\_r_for\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ .node rect,#chatgpt-mermaid-\_r_for\_ .node circle,#chatgpt-mermaid-\_r_for\_ .node ellipse,#chatgpt-mermaid-\_r_for\_ .node polygon,#chatgpt-mermaid-\_r_for\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ .rough-node .label text,#chatgpt-mermaid-\_r_for\_ .node .label text,#chatgpt-mermaid-\_r_for\_ .image-shape .label,#chatgpt-mermaid-\_r_for\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_for\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ .rough-node .label,#chatgpt-mermaid-\_r_for\_ .node .label,#chatgpt-mermaid-\_r_for\_ .image-shape .label,#chatgpt-mermaid-\_r_for\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_for\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_for\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_for\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_for\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_for\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_for\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_for\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_for\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_for\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_for\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_for\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_for\_ .icon-shape,#chatgpt-mermaid-\_r_for\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_for\_ .icon-shape p,#chatgpt-mermaid-\_r_for\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_for\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_for\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_for\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_for\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_for\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_for\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_for\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_for\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_for\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_for\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_for\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_for\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_for\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_for\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_for\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_for\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_for\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_for\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ .node rect,#chatgpt-mermaid-\_r_for\_ .node circle,#chatgpt-mermaid-\_r_for\_ .node ellipse,#chatgpt-mermaid-\_r_for\_ .node polygon,#chatgpt-mermaid-\_r_for\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_for\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_for\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_for\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_for\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_for\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}allwebserversdbserversweb_stagingweb_productionweb1web2db1

### List hosts matching a pattern

```
ansible webservers -i inventory.ini --list-hosts
```

This is useful before running a playbook because you can confirm which machines Ansible will target.

Common mistake: If your inventory group is named `webservers` but your playbook specifies `webserver`, Ansible will not find the expected hosts.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 10. Installing Ansible — APT vs PIP vs PIPX

The instructor discussed different methods of installing Ansible.

| Method  | Purpose                                                            |
| ------- | ------------------------------------------------------------------ |
| APT     | Ubuntu/Debian system package manager                               |
| DNF/YUM | Fedora, RHEL-family and related distributions                      |
| PIP     | Python package installer                                           |
| PIPX    | Installs Python command-line applications in isolated environments |

### Install through APT

```
sudo apt update
sudo apt install ansible -y
```

### Install through DNF

```
sudo dnf install ansible -y
```

### Install through PIPX

```
pipx install --include-deps ansible
```

### Why use PIPX?

Ansible is also distributed as Python software.

Installing Python applications directly into a system-managed Python environment can cause dependency conflicts or externally managed environment errors.

PIPX isolates command-line applications in separate virtual environments.

Important correction: PIP and PIPX are not the same. PIP installs Python packages into a Python environment; PIPX isolates applications and their dependencies.

You do not need to install Ansible using every method. Choose the installation method supported by your operating system and organization's standards.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 11. Ad-hoc Commands vs Playbooks — When to Use Which?

The instructor revisited this distinction.

| Ad-hoc command            | Playbook                        |
| ------------------------- | ------------------------------- |
| Quick one-time operation  | Repeatable automation           |
| Usually one task          | Multiple tasks and plays        |
| Direct terminal execution | YAML file                       |
| Good for troubleshooting  | Good for team collaboration     |
| Less structured           | Version-controlled and reusable |

Example:

```
ansible webservers -i inventory.ini \
  -m ansible.builtin.ping
```

The same connectivity check can be written as a playbook:

```
---
- name: Check web server connectivity
  hosts: webservers

  tasks:
    - name: Verify Ansible connectivity
      ansible.builtin.ping:
```

Industry best practice: Store playbooks in Git so that team members can review changes, track versions and collaborate safely.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 12. Version Control for DevOps Artifacts

One of the important discussions was about storing automation code in a shared repository.

In real projects, version control is not limited to application source code.

You should also version-control:

- Jenkinsfiles.
- Dockerfiles.
- Kubernetes YAML files.
- Terraform configurations.
- Ansible playbooks.
- Deployment scripts.
- Application configuration templates.

### Example repository structure

```
platform-engineering/
├── ansible/
│   ├── inventory/
│   │   ├── dev.ini
│   │   └── prod.ini
│   └── playbooks/
│       ├── nginx.yml
│       └── users.yml
│
├── terraform/
│   ├── main.tf
│   └── variables.tf
│
├── kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
│
└── jenkins/
    └── Jenkinsfile
```

### Why is this important?

Imagine two engineers independently maintain playbooks.

- Engineer A installs NGINX version 1.24.
- Engineer B changes the playbook to install version 1.26.
- Both execute different versions against the same infrastructure.

This can introduce configuration drift and unexpected changes.

A shared Git repository with reviews and controlled releases helps prevent these problems.

Remember: Infrastructure code and automation code should be treated like application code.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 13. Scheduling Ansible Using Cron

Ansible does not need to be manually executed every time.

You can schedule a playbook using Linux cron.

For example, suppose you want to run an Ansible connectivity check every day at 9 PM.

### Check existing cron jobs

```
crontab -l
```

### Edit cron jobs

```
crontab -e
```

### Cron syntax

Cron uses five fields:

```
Minute Hour Day-of-month Month Day-of-week
```

| Field        | Allowed values             |
| ------------ | -------------------------- |
| Minute       | 0–59                       |
| Hour         | 0–23                       |
| Day of month | 1–31                       |
| Month        | 1–12                       |
| Day of week  | 0–7 (Sunday can be 0 or 7) |

### Examples

| Schedule                        | Cron expression |
| ------------------------------- | --------------- |
| Every minute                    | `* * * * *`     |
| Every hour                      | `0 * * * *`     |
| Every day at midnight           | `0 0 * * *`     |
| Every day at 9 AM               | `0 9 * * *`     |
| Every day at 9 PM               | `0 21 * * *`    |
| Every Monday at 9 AM            | `0 9 * * 1`     |
| Every Monday and Friday at 9 PM | `0 21 * * 1,5`  |

### Schedule an Ansible playbook

Add this to your crontab:

```
0 21 * * * /usr/bin/ansible-playbook -i /home/ubuntu/ansible/inventory.ini /home/ubuntu/ansible/ping.yml >> /home/ubuntu/ansible/cron.log 2>&1
```

This runs the playbook every day at 9 PM and appends its output to a log file.

Important: Use absolute paths, and ensure the cron execution user has access to the inventory, SSH keys and required permissions. Also check the server's timezone.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 14. Ansible Tower and Automation Scheduling

The instructor also introduced Ansible Tower.

[What's new in Ansible Automation Platform 2.4](https://images.openai.com/static-rsc-4/PBUKBAyWwO90heayK_T2lbSgeozcG-hJl1i6ypPYxj_mguc1mTVatoZrten2ODhyT7l3p5YjDWAg0rOg-vAJNvIfnpLoHe_sYt1hH-ke1WoOB8A312Z_2qucXDejXUmkVoOpdTzIGYGBHdVVUDwm1T3xFIQA9kZ7CN3GGHGnZJ0?purpose=inline)

[redhat.com](https://www.redhat.com/en/blog/whats-new-in-ansible-automation-platform-2.4?utm_source=chatgpt.com)

Ansible Tower was the commercial automation management interface associated with Ansible. Its capabilities are now part of Red Hat Ansible Automation Platform, particularly Automation Controller.

It provides capabilities such as:

- Web-based interface.
- Job templates.
- Scheduling.
- Role-based access control.
- Centralized management.
- Job execution history.

### Scheduling options

| Method                      | Purpose                                       |
| --------------------------- | --------------------------------------------- |
| Cron                        | Schedule playbooks using Linux                |
| Ansible Automation Platform | Enterprise automation and scheduling          |
| Jenkins                     | Trigger Ansible as part of CI/CD              |
| Other schedulers            | Integrate with supported automation workflows |

Practical example: Jenkins can trigger an Ansible playbook after a successful application build, while cron can run a daily server health-check playbook.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 15. Linux Uptime and Load Average

The final part of the session moved into an important Linux interview topic.

### What is uptime?

The `uptime` command displays:

- Current time.
- How long the system has been running.
- Number of logged-in users.
- System load averages.

Run:

```
uptime
```

Example:

```
09:30:00 up 1 hour, 40 minutes, 1 user,
load average: 0.05, 0.10, 0.09
```

The three load-average values represent:

| Value | Meaning                          |
| ----- | -------------------------------- |
| 0.05  | Average over the last 1 minute   |
| 0.10  | Average over the last 5 minutes  |
| 0.09  | Average over the last 15 minutes |

### What is load average?

Load average represents the average number of tasks that are runnable or waiting for CPU time, plus tasks in uninterruptible sleep (commonly I/O waits).

It is not the same as CPU utilization percentage.

### How do you interpret load average?

The instructor discussed why CPU core count matters.

For a rough interpretation, compare load average with the number of logical CPUs.

| Logical CPUs | Load average of 1 | Rough interpretation             |
| ------------ | ----------------- | -------------------------------- |
| 1            | 1                 | Load around capacity             |
| 2            | 1                 | Load around half capacity        |
| 4            | 1                 | Load around one-quarter capacity |
| 8            | 1                 | Load around one-eighth capacity  |

This is a simplified comparison, not an exact CPU utilization calculation. I/O wait and other factors also affect load average.

### Example

Suppose two servers both report a load average of 4.

- Server A has 4 logical CPUs.
- Server B has 8 logical CPUs.

The same load average can indicate very different levels of pressure.

Interview answer: You cannot interpret load average without knowing the number of logical CPUs and considering whether the system is experiencing CPU or I/O contention.

### Commands to check CPU information

```
lscpu
```

This displays processor information, including:

- CPU architecture.
- CPU count.
- Threads per core.
- Cores per socket.
- Sockets.

Another useful command:

```
nproc
```

This displays the number of processing units available to the current process.

For live system information:

```
top
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+3

## 16. Quick Revision — Interview Questions

| Question                                     | Answer                                                               |
| -------------------------------------------- | -------------------------------------------------------------------- |
| What is the Ansible ping module?             | Tests Ansible connectivity, not ICMP connectivity.                   |
| Difference between command and shell?        | Shell supports pipes, redirection and shell features.                |
| What is the copy module?                     | Transfers files from control node to managed node.                   |
| What does `ok` mean?                         | Desired state already exists.                                        |
| What does `changed` mean?                    | Ansible modified the system.                                         |
| What does `unreachable` mean?                | Ansible could not connect to the host.                               |
| What is `ignore_errors`?                     | Allows eligible task failures without stopping the play.             |
| What is rescue?                              | Executes recovery tasks after a block failure.                       |
| What is check mode?                          | Simulates supported changes without applying them.                   |
| What does `--diff` do?                       | Displays file/configuration differences.                             |
| What are forks?                              | Maximum number of hosts Ansible processes concurrently.              |
| What is the default forks value?             | 5.                                                                   |
| What is `become: true`?                      | Enables privilege escalation.                                        |
| Why is Python required on managed nodes?     | Most standard Ansible modules require Python.                        |
| How do you display inventory groups?         | `ansible-inventory --graph`.                                         |
| How do you schedule a playbook?              | Cron or Ansible Automation Platform.                                 |
| What is uptime?                              | Displays system running time and load averages.                      |
| What does load average represent?            | Average runnable and uninterruptible tasks over 1, 5 and 15 minutes. |
| Is load average the same as CPU utilization? | No.                                                                  |
| How do you check CPU cores?                  | `lscpu` or `nproc`.                                                  |

## 17. Complete Session Mental Model

\#chatgpt-mermaid-\_r_ft6\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_ft6\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_ft6\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_ft6\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_ft6\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_ft6\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_ft6\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_ft6\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_ft6\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_ft6\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_ft6\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_ft6\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_ft6\_ p{margin:0;}#chatgpt-mermaid-\_r_ft6\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_ft6\_ .label text,#chatgpt-mermaid-\_r_ft6\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ .node rect,#chatgpt-mermaid-\_r_ft6\_ .node circle,#chatgpt-mermaid-\_r_ft6\_ .node ellipse,#chatgpt-mermaid-\_r_ft6\_ .node polygon,#chatgpt-mermaid-\_r_ft6\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ .rough-node .label text,#chatgpt-mermaid-\_r_ft6\_ .node .label text,#chatgpt-mermaid-\_r_ft6\_ .image-shape .label,#chatgpt-mermaid-\_r_ft6\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_ft6\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ .rough-node .label,#chatgpt-mermaid-\_r_ft6\_ .node .label,#chatgpt-mermaid-\_r_ft6\_ .image-shape .label,#chatgpt-mermaid-\_r_ft6\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_ft6\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_ft6\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_ft6\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_ft6\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_ft6\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_ft6\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_ft6\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_ft6\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_ft6\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_ft6\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_ft6\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_ft6\_ .icon-shape,#chatgpt-mermaid-\_r_ft6\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_ft6\_ .icon-shape p,#chatgpt-mermaid-\_r_ft6\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_ft6\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_ft6\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_ft6\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_ft6\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_ft6\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_ft6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_ft6\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_ft6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_ft6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ft6\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_ft6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_ft6\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_ft6\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_ft6\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_ft6\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_ft6\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ .node rect,#chatgpt-mermaid-\_r_ft6\_ .node circle,#chatgpt-mermaid-\_r_ft6\_ .node ellipse,#chatgpt-mermaid-\_r_ft6\_ .node polygon,#chatgpt-mermaid-\_r_ft6\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_ft6\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_ft6\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_ft6\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_ft6\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_ft6\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Ansible Control NodeInventorySelect Managed HostsSSH ConnectivityExecute ModulesCheck Task Resultsok / changed / failedskipped / ignored / rescuedCheck Mode + DiffCron / Jenkins / AutomationControllerGit Repository

Remember these key concepts:

1. Modules perform the actual operations.
2. Ad-hoc commands are useful for quick tasks; playbooks are better for reusable automation.
3. SSH and sudo permissions are essential for remote configuration.
4. Check mode helps preview changes.
5. Forks control parallel execution.
6. Git keeps automation code consistent across teams.
7. Cron and automation platforms schedule execution.
8. Load average must be interpreted relative to CPU capacity.

Original transcript: [DevOps Session — 1 November 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-11-01_09.34.39.txt?utm_source=chatgpt.com)

Session complete

This session extends the previous Ansible classes with practical execution, troubleshooting, scheduling and Linux monitoring concepts. The next logical topic is Ansible variables, conditions, loops, handlers and roles.