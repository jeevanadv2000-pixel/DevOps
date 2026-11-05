# DevOps Session — Ansible Playbooks, Inventory & Remote Server Configuration

Date: 31 October 2026\
Instructor: Rochak Agrawal\
Focus: Writing playbooks, inventory management, SSH authentication, Ansible modules, privilege escalation and configuring remote EC2 instances.

Notes are condensed from the transcript. Repeated discussions and troubleshooting chatter have been removed. Technical inaccuracies have been corrected.

## 1. Ad-hoc Commands vs Playbooks

In the previous session, we learned that Ansible supports two ways of executing tasks.

| Ad-hoc commands             | Playbooks                        |
| --------------------------- | -------------------------------- |
| Execute one-time tasks      | Execute multiple organized tasks |
| Run directly from terminal  | Written in YAML                  |
| Useful for quick operations | Useful for repeatable automation |
| Less structured             | Easier to maintain and reuse     |

Example of an ad-hoc command:

```
ansible localhost -m ansible.builtin.file \
  -a "path=/tmp/demo.txt state=touch" \
  -c local
```

The same operation using a playbook:

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

Important: Both approaches use Ansible modules. The difference is how you organize and execute the automation.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 2. Inventory File — Where to Execute?

An inventory file contains the servers Ansible should manage.

For example, an organization might have:

- 10 web servers.
- 20 application servers.
- 5 database servers.

Instead of maintaining every server in one shared default inventory, teams can organize their servers using separate inventory files or groups.

### Example: `inventory.ini`

```
[webservers]
192.168.1.10
192.168.1.11

[appservers]
192.168.1.20

[dbservers]
192.168.1.30
```

### Can we create multiple inventory files?

Yes. Inventory files can have different names and locations.

For example:

```
ansible-project/
├── inventory/
│   ├── dev.ini
│   ├── prod.ini
│   └── database.ini
│
├── playbooks/
│   ├── nginx.yml
│   └── database.yml
```

Run a playbook with a specific inventory:

```
ansible-playbook -i inventory/dev.ini playbooks/nginx.yml
```

Here:

| Command part          | Purpose                 |
| --------------------- | ----------------------- |
| `ansible-playbook`    | Runs a playbook         |
| `-i`                  | Specifies the inventory |
| `inventory/dev.ini`   | Inventory file location |
| `playbooks/nginx.yml` | Playbook to execute     |

### What happens if we don't specify `-i`?

Ansible uses its configured default inventory source. A common default is `/etc/ansible/hosts`, although the actual path can be changed in `ansible.cfg`.

Remember:

- Inventory = Where to execute.
- Playbook = What to execute.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Understanding Plays and Tasks

A playbook can contain multiple plays, and each play can contain multiple tasks.

Example:

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

    - name: Change file permissions
      ansible.builtin.file:
        path: /tmp/demo.txt
        mode: '0644'

- name: Manage directories
  hosts: localhost
  connection: local

  tasks:
    - name: Create project directory
      ansible.builtin.file:
        path: /tmp/new-project
        state: directory
        mode: '0755'
```

### Visualize the structure

\#chatgpt-mermaid-\_r_fd2\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_fd2\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_fd2\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_fd2\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_fd2\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_fd2\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_fd2\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_fd2\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_fd2\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_fd2\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fd2\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fd2\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_fd2\_ p{margin:0;}#chatgpt-mermaid-\_r_fd2\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_fd2\_ .label text,#chatgpt-mermaid-\_r_fd2\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ .node rect,#chatgpt-mermaid-\_r_fd2\_ .node circle,#chatgpt-mermaid-\_r_fd2\_ .node ellipse,#chatgpt-mermaid-\_r_fd2\_ .node polygon,#chatgpt-mermaid-\_r_fd2\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ .rough-node .label text,#chatgpt-mermaid-\_r_fd2\_ .node .label text,#chatgpt-mermaid-\_r_fd2\_ .image-shape .label,#chatgpt-mermaid-\_r_fd2\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_fd2\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ .rough-node .label,#chatgpt-mermaid-\_r_fd2\_ .node .label,#chatgpt-mermaid-\_r_fd2\_ .image-shape .label,#chatgpt-mermaid-\_r_fd2\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_fd2\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_fd2\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fd2\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fd2\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_fd2\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_fd2\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fd2\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fd2\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_fd2\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_fd2\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fd2\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_fd2\_ .icon-shape,#chatgpt-mermaid-\_r_fd2\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_fd2\_ .icon-shape p,#chatgpt-mermaid-\_r_fd2\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_fd2\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_fd2\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fd2\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_fd2\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_fd2\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_fd2\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_fd2\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_fd2\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_fd2\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fd2\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_fd2\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fd2\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_fd2\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_fd2\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_fd2\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_fd2\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ .node rect,#chatgpt-mermaid-\_r_fd2\_ .node circle,#chatgpt-mermaid-\_r_fd2\_ .node ellipse,#chatgpt-mermaid-\_r_fd2\_ .node polygon,#chatgpt-mermaid-\_r_fd2\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_fd2\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_fd2\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_fd2\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_fd2\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fd2\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}PlaybookPlay 1: Manage FilesPlay 2: Manage DirectoriesTask 1: Create FileTask 2: Change PermissionsTask 1: Create Directory

### Important: Task execution order

Ansible normally executes tasks in the order in which they appear in a play.

For example:

1. Install NGINX.
2. Start NGINX.

You cannot expect the service to start successfully if you try to start it before installing it.

Unlike Terraform's dependency graph, Ansible generally follows the task order you define. You must organize the sequence correctly.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 4. YAML Indentation and Syntax Validation

Ansible playbooks use YAML, so indentation is very important.

Correct:

```
- name: Install NGINX
  hosts: webservers

  tasks:
    - name: Install package
      ansible.builtin.apt:
        name: nginx
        state: present
```

Incorrect indentation can cause YAML parsing errors or change the meaning of the configuration.

### How to validate a playbook?

The instructor discussed syntax validation but did not demonstrate the exact command.

The correct command is:

```
ansible-playbook --syntax-check -i inventory.ini playbook.yml
```

To format and lint your playbooks, you can also use:

```
ansible-lint playbook.yml
```

`ansible-lint` is an additional tool that may need to be installed separately.

Tip: Use the Ansible extension in VS Code for YAML validation, syntax highlighting and autocomplete.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 5. Connecting Ansible to a Remote EC2 Instance

This was the major practical demonstration in the session.

Previously, the instructor executed playbooks on `localhost`. Now, the objective was to configure a separate Ubuntu EC2 instance from the Ansible control node.

### Architecture

[Ansible tutorial with AWS EC2 | End Point Dev](https://images.openai.com/static-rsc-4/5zr2irx1z_e-v19FqtSyeJdC3O_ziZqO33pJtrYjfoXdVcP9MKkxVqoVO6ijgvzxi_ZiSmpm6iFO30OGNT-gzmOycSWl3SBU06BSTd7qx-VfmG77AzU9JvFouq5IV6kuwtSAico2WI-dtUeEFyukiuln_1rL0arak0B4jBjT9-U?purpose=inline)

[endpointdev.com](https://www.endpointdev.com/blog/2022/08/ansible-tutorial-with-aws-ec2/?utm_source=chatgpt.com)

\#chatgpt-mermaid-\_r_fe3\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_fe3\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_fe3\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_fe3\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_fe3\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_fe3\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_fe3\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_fe3\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_fe3\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_fe3\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fe3\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fe3\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_fe3\_ p{margin:0;}#chatgpt-mermaid-\_r_fe3\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_fe3\_ .label text,#chatgpt-mermaid-\_r_fe3\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ .node rect,#chatgpt-mermaid-\_r_fe3\_ .node circle,#chatgpt-mermaid-\_r_fe3\_ .node ellipse,#chatgpt-mermaid-\_r_fe3\_ .node polygon,#chatgpt-mermaid-\_r_fe3\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ .rough-node .label text,#chatgpt-mermaid-\_r_fe3\_ .node .label text,#chatgpt-mermaid-\_r_fe3\_ .image-shape .label,#chatgpt-mermaid-\_r_fe3\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_fe3\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ .rough-node .label,#chatgpt-mermaid-\_r_fe3\_ .node .label,#chatgpt-mermaid-\_r_fe3\_ .image-shape .label,#chatgpt-mermaid-\_r_fe3\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_fe3\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_fe3\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fe3\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fe3\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_fe3\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_fe3\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fe3\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fe3\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_fe3\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_fe3\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fe3\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_fe3\_ .icon-shape,#chatgpt-mermaid-\_r_fe3\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_fe3\_ .icon-shape p,#chatgpt-mermaid-\_r_fe3\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_fe3\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_fe3\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fe3\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_fe3\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_fe3\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_fe3\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_fe3\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_fe3\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_fe3\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fe3\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_fe3\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fe3\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_fe3\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_fe3\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_fe3\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_fe3\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ .node rect,#chatgpt-mermaid-\_r_fe3\_ .node circle,#chatgpt-mermaid-\_r_fe3\_ .node ellipse,#chatgpt-mermaid-\_r_fe3\_ .node polygon,#chatgpt-mermaid-\_r_fe3\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_fe3\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_fe3\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_fe3\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_fe3\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fe3\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Control NodeAnsible installedManaged EC2UbuntuInventory FilePlaybookExecute tasksReturn resultsSSH / Port 22

### Prerequisites

Before Ansible can configure a remote machine, ensure:

1. The control node can reach the managed node over the network.
2. SSH is running on the managed node.
3. The SSH username is correct.
4. Authentication is configured.
5. The security group allows inbound SSH on port 22 from the control node.
6. The managed node has Python available for most Ansible modules.

### SSH passwordless authentication

The instructor demonstrated public-key authentication.

The process:

1. Generate an SSH key pair on the control node.
2. Copy the public key to the managed node.
3. Connect using the corresponding private key.

Generate an Ed25519 key:

```
ssh-keygen -t ed25519
```

This generates:

| File                    | Purpose                    |
| ----------------------- | -------------------------- |
| `~/.ssh/id_ed25519`     | Private key — keep secret  |
| `~/.ssh/id_ed25519.pub` | Public key — can be shared |

Copy the public key:

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@MANAGED_SERVER_IP
```

Test the connection:

```
ssh ubuntu@MANAGED_SERVER_IP
```

Important correction from the session: The instructor initially tried an RSA key and later suggested RSA might not be supported by AWS EC2. That is not generally true. EC2 Linux instances support SSH RSA keys when the client and server configurations permit them. Ed25519 is a convenient modern option, but the RSA algorithm itself is not inherently incompatible with EC2.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

### SSH troubleshooting checklist

| Error                         | What to check                                                |
| ----------------------------- | ------------------------------------------------------------ |
| Permission denied (publickey) | Username, correct private/public key pair, `authorized_keys` |
| Connection timed out          | Security group, network ACL, routing, firewall               |
| No route to host              | Network connectivity and routing                             |
| Host key verification failed  | Remote host key in `known_hosts`                             |
| Unprotected private key       | Private key permissions, normally `600` or stricter          |

Useful commands:

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
```

For 100 servers, you do not need to copy the public key manually to every server. You can automate the process using `ssh-copy-id` in a loop, provided you have the necessary initial authentication and access.

## 6. Inventory Configuration for Remote Servers

Once SSH connectivity is established, add the managed server to your inventory.

Example: `inventory.ini`

```
[webservers]
web1 ansible_host=192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

Here:

| Parameter                      | Meaning                             |
| ------------------------------ | ----------------------------------- |
| `web1`                         | Inventory alias                     |
| `ansible_host`                 | Actual IP address                   |
| `ansible_user`                 | SSH username                        |
| `ansible_ssh_private_key_file` | Private key used for authentication |

Test connectivity:

```
ansible webservers -i inventory.ini -m ansible.builtin.ping
```

The Ansible `ping` module checks whether Ansible can connect and execute a module. It is not the same as the Linux network `ping` command.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 7. Privilege Escalation — `become: true`

Installing packages and modifying system services generally requires administrator privileges.

Ansible provides privilege escalation through `become`.

Example:

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
        update_cache: true
```

### What does `become: true` do?

It tells Ansible to execute tasks with elevated privileges, usually through `sudo`.

| Setting               | Purpose                       |
| --------------------- | ----------------------------- |
| `become: true`        | Enable privilege escalation   |
| `become_user: root`   | Specify the target user       |
| `become_method: sudo` | Specify the escalation method |

You can define `become` at the play level or task level.

Important: The SSH user must have permission to escalate privileges. Ansible does not automatically grant administrator access.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 8. Installing and Starting NGINX Using Ansible

The instructor demonstrated installing NGINX on an Ubuntu managed node.

Complete example:

```
---
- name: Install and configure NGINX
  hosts: webservers
  become: true

  tasks:
    - name: Update APT cache and install NGINX
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Ensure NGINX is running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

Run:

```
ansible-playbook -i inventory.ini nginx.yml
```

### Understand the modules

| Module                    | Purpose                                    |
| ------------------------- | ------------------------------------------ |
| `ansible.builtin.apt`     | Installs or removes Debian/Ubuntu packages |
| `ansible.builtin.service` | Starts, stops or restarts services         |

### Understand the arguments

| Argument             | Meaning                          |
| -------------------- | -------------------------------- |
| `name: nginx`        | Package or service name          |
| `state: present`     | Ensure package is installed      |
| `state: started`     | Ensure service is running        |
| `update_cache: true` | Refresh package repository cache |
| `enabled: true`      | Enable service at system startup |

Important: The `apt` module is for Debian-based systems such as Ubuntu. For Amazon Linux 2023, use the `dnf` module instead.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 9. Understanding Ansible Output and Idempotency

When you execute a playbook, Ansible reports the result of each task.

| Status        | Meaning                               |
| ------------- | ------------------------------------- |
| `ok`          | Task completed; no change was needed  |
| `changed`     | Task modified the managed system      |
| `failed`      | Task execution failed                 |
| `skipped`     | Task was skipped                      |
| `unreachable` | Ansible could not connect to the host |

Example:

First execution:

```
TASK [Install NGINX]
changed: [web1]

TASK [Ensure NGINX is running]
changed: [web1]
```

Second execution:

```
TASK [Install NGINX]
ok: [web1]

TASK [Ensure NGINX is running]
ok: [web1]
```

This demonstrates idempotency.

If someone manually removes NGINX, running the playbook again will install it because the desired state is `present`.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 10. Ansible Verbosity — Debugging

When a playbook fails, you may need more information.

Ansible supports different verbosity levels:

| Command | Verbosity |
| ------- | --------- |
| `-v`    | Level 1   |
| `-vv`   | Level 2   |
| `-vvv`  | Level 3   |
| `-vvvv` | Level 4   |

Example:

```
ansible-playbook -i inventory.ini nginx.yml -vvv
```

This provides more detailed execution information, useful for troubleshooting SSH connections, task execution and module behaviour.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 11. Multiple Playbooks vs Multiple Plays

A common question in the session was whether multiple playbooks can be run together.

An individual `ansible-playbook` invocation normally takes one playbook file. However, one playbook can contain multiple plays targeting different server groups.

Example:

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

- name: Configure database servers
  hosts: dbservers
  become: true

  tasks:
    - name: Install PostgreSQL
      ansible.builtin.apt:
        name: postgresql
        state: present
```

\#chatgpt-mermaid-\_r_fh7\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_fh7\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_fh7\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_fh7\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_fh7\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_fh7\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_fh7\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_fh7\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_fh7\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_fh7\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fh7\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fh7\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_fh7\_ p{margin:0;}#chatgpt-mermaid-\_r_fh7\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_fh7\_ .label text,#chatgpt-mermaid-\_r_fh7\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ .node rect,#chatgpt-mermaid-\_r_fh7\_ .node circle,#chatgpt-mermaid-\_r_fh7\_ .node ellipse,#chatgpt-mermaid-\_r_fh7\_ .node polygon,#chatgpt-mermaid-\_r_fh7\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ .rough-node .label text,#chatgpt-mermaid-\_r_fh7\_ .node .label text,#chatgpt-mermaid-\_r_fh7\_ .image-shape .label,#chatgpt-mermaid-\_r_fh7\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_fh7\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ .rough-node .label,#chatgpt-mermaid-\_r_fh7\_ .node .label,#chatgpt-mermaid-\_r_fh7\_ .image-shape .label,#chatgpt-mermaid-\_r_fh7\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_fh7\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_fh7\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fh7\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fh7\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_fh7\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_fh7\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fh7\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fh7\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_fh7\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_fh7\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_fh7\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_fh7\_ .icon-shape,#chatgpt-mermaid-\_r_fh7\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_fh7\_ .icon-shape p,#chatgpt-mermaid-\_r_fh7\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_fh7\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_fh7\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_fh7\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_fh7\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_fh7\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_fh7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_fh7\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_fh7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_fh7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fh7\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_fh7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_fh7\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_fh7\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_fh7\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_fh7\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_fh7\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ .node rect,#chatgpt-mermaid-\_r_fh7\_ .node circle,#chatgpt-mermaid-\_r_fh7\_ .node ellipse,#chatgpt-mermaid-\_r_fh7\_ .node polygon,#chatgpt-mermaid-\_r_fh7\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_fh7\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_fh7\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_fh7\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_fh7\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_fh7\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}site.ymlPlay 1Play 2webservers groupdbservers groupInstall NGINXInstall PostgreSQL

Run:

```
ansible-playbook -i inventory.ini site.yml
```

This is a clean way to manage different types of servers from one playbook.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 12. Ansible Modules and Collections

Ansible provides modules for different operations.

Examples:

| Requirement          | Module                       |
| -------------------- | ---------------------------- |
| Install packages     | `apt`, `dnf`, `package`      |
| Manage services      | `service`, `systemd_service` |
| Create files         | `file`                       |
| Copy files           | `copy`                       |
| Execute commands     | `command`, `shell`           |
| Manage AWS resources | AWS collection modules       |
| Manage users         | `user`                       |

### What is a Collection?

A collection is a package of related Ansible content, such as modules, plugins, roles and documentation.

For example, to automate AWS resources, you can use the `amazon.aws` collection.

Install it:

```
ansible-galaxy collection install amazon.aws
```

Some AWS modules also require dependencies such as the AWS SDK for Python (`boto3`) and valid AWS credentials.

### Does Ansible need a special module for everything?

No.

Consider these examples:

- Restart Jenkins service → Use the service module.
- Create a Jenkins configuration file → Use the copy or template module.
- Run a `kubectl` command → Use the command or shell module.
- Create an AWS VPC → Use an appropriate AWS collection module.

Remember: If an operation can be performed through an existing module or command, you may not need an additional collection. Specialized integrations may require additional modules or collections.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 13. Quick Revision — Interview Questions

| Question                                        | Answer                                                                 |
| ----------------------------------------------- | ---------------------------------------------------------------------- |
| What is an inventory file?                      | Contains the managed hosts or sources from which hosts are discovered. |
| Can we create multiple inventory files?         | Yes, and select one using `-i`.                                        |
| What is the purpose of `-i`?                    | Specifies the inventory source.                                        |
| What is a play?                                 | A set of tasks applied to selected hosts.                              |
| Can a playbook have multiple plays?             | Yes.                                                                   |
| What is `become: true`?                         | Enables privilege escalation, usually through sudo.                    |
| How does Ansible connect to Linux servers?      | Usually through SSH.                                                   |
| What is `ssh-copy-id`?                          | Copies a public key to a remote user's authorized keys.                |
| How do you validate playbook syntax?            | `ansible-playbook --syntax-check`.                                     |
| What is the APT module used for?                | Managing packages on Debian-based systems.                             |
| What does `state: present` mean?                | Ensure the package or resource exists.                                 |
| What does `state: started` mean?                | Ensure the service is running.                                         |
| What does `-vvv` do?                            | Provides detailed debugging output.                                    |
| What is a collection?                           | A package of Ansible modules, plugins, roles and related content.      |
| How can one playbook manage web and DB servers? | Define separate plays with different host groups.                      |

## 14. Mental Model — Remember This Workflow

## Ansible execution workflow

1. Inventory — Identify the servers.
2. SSH — Establish secure connectivity.
3. Playbook — Define what needs to be configured.
4. Tasks and Modules — Perform the operations.
5. Become — Obtain required privileges.
6. Execution — Apply the desired configuration.
7. Result — Check `ok`, `changed` or `failed`.

Original transcript: [DevOps Session — 31 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-10-31_09.29.29.txt?utm_source=chatgpt.com)

Session complete: You have now covered Ansible playbooks, inventory files, SSH authentication, remote server configuration, privilege escalation and modules.

The next useful hands-on exercise is to connect your existing Amazon Linux EC2 control node to a second EC2 instance and deploy NGINX using an Ansible playbook.