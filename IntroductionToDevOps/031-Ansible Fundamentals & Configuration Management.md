# DevOps Session — Ansible Fundamentals & Configuration Management

Date: 30 October 2026\
Instructor: Rochak Agrawal\
Focus: Ansible architecture, inventory, playbooks, roles, modules, idempotency and execution workflow.

Notes are condensed from the transcript. Repeated discussions and participant chatter have been removed, speech-to-text errors corrected, and important technical details clarified.

## 1. What is Configuration Management?

Configuration management means automating the setup and maintenance of servers so that they remain consistent.

For example, a company may need to configure hundreds of servers with:

- Java, Python, NGINX and other runtimes.
- Users, groups and permissions.
- Security software and firewall settings.
- Application configurations.
- Services that must be running.

### Why do we need it?

| Problem             | Explanation                                                     |
| ------------------- | --------------------------------------------------------------- |
| Configuration drift | Servers gradually become different because of manual changes.   |
| Human errors        | Repeating manual commands can introduce mistakes.               |
| Time-consuming      | Configuring hundreds of servers individually is slow.           |
| Poor documentation  | Manual fixes may not be recorded.                               |
| Scalability         | Manual configuration becomes difficult as infrastructure grows. |

Solution: Use configuration management tools such as Ansible, Puppet, Chef and Salt.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 2. What is Ansible?

Ansible is an open-source automation and configuration management tool sponsored by Red Hat.

It helps automate tasks such as:

- Installing packages.
- Creating users and groups.
- Managing files and permissions.
- Starting and stopping services.
- Deploying applications.
- Configuring servers and some cloud resources.

Ansible uses YAML playbooks to describe the configuration you want.

### Ansible vs Shell Script

Suppose you want to create a Linux user named `deploy`.

A shell script might run:

```
sudo useradd deploy
```

- First execution: creates the user.
- Second execution: may return an error because the user already exists.

Ansible can express the desired condition:

```
- name: Create deploy user
  ansible.builtin.user:
    name: deploy
    state: present
```

- First execution: creates the user and reports `changed`.
- Second execution: reports `ok` because the user already exists.

This behaviour is called idempotency.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Why Ansible is popular

| Feature            | Explanation                                                    |
| ------------------ | -------------------------------------------------------------- |
| Agentless          | No Ansible agent needs to be installed on managed Linux hosts. |
| YAML               | Playbooks are relatively easy to read and understand.          |
| Push-based         | The control node connects to managed nodes and sends tasks.    |
| Idempotent         | Repeating tasks generally does not create unnecessary changes. |
| Modules            | Provides reusable units of automation for specific operations. |
| Parallel execution | Can manage multiple hosts concurrently.                        |
| Integration        | Can be executed from Jenkins and other CI/CD pipelines.        |

Ansible Community Edition is free and command-line based. Red Hat Ansible Automation Platform adds enterprise capabilities such as a web interface, access control and scheduling.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 4. Ansible Architecture

Ansible has two important types of machines:

- Control node: Machine where Ansible is installed and automation is initiated.
- Managed nodes: Servers that Ansible connects to and configures.

[Ansible for Configuration Management: The 2026 Guide | CloudCops Resources](https://images.openai.com/static-rsc-4/KKiv737Jzm5lwWwpVBc0R1Dl7IFiBlPgzYCm5qcBNXug3vluWhN6qsIG5lU018Nh4_e66gHqzi3bxHYcLvdmW_SghcrVZK_19BmngD1zWJhmQFYpAauvtEEsO9Zk0XYDPwBJfmgXR-TAKitBASPGTyc2LKwZ-yJB0jUCXErCKZg?purpose=inline)

[resources.cloudcops.com](https://resources.cloudcops.com/blogs/ansible-for-configuration-management?utm_source=chatgpt.com)

### Architecture diagram

\#chatgpt-mermaid-\_r_f78\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f78\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f78\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f78\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f78\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f78\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f78\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f78\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f78\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f78\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f78\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f78\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f78\_ p{margin:0;}#chatgpt-mermaid-\_r_f78\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f78\_ .label text,#chatgpt-mermaid-\_r_f78\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ .node rect,#chatgpt-mermaid-\_r_f78\_ .node circle,#chatgpt-mermaid-\_r_f78\_ .node ellipse,#chatgpt-mermaid-\_r_f78\_ .node polygon,#chatgpt-mermaid-\_r_f78\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ .rough-node .label text,#chatgpt-mermaid-\_r_f78\_ .node .label text,#chatgpt-mermaid-\_r_f78\_ .image-shape .label,#chatgpt-mermaid-\_r_f78\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f78\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ .rough-node .label,#chatgpt-mermaid-\_r_f78\_ .node .label,#chatgpt-mermaid-\_r_f78\_ .image-shape .label,#chatgpt-mermaid-\_r_f78\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f78\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f78\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f78\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f78\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f78\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f78\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f78\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f78\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f78\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f78\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f78\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f78\_ .icon-shape,#chatgpt-mermaid-\_r_f78\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f78\_ .icon-shape p,#chatgpt-mermaid-\_r_f78\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f78\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f78\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f78\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f78\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f78\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f78\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f78\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f78\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f78\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f78\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f78\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f78\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f78\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f78\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f78\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f78\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ .node rect,#chatgpt-mermaid-\_r_f78\_ .node circle,#chatgpt-mermaid-\_r_f78\_ .node ellipse,#chatgpt-mermaid-\_r_f78\_ .node polygon,#chatgpt-mermaid-\_r_f78\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f78\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f78\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f78\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f78\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f78\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Control NodeAnsible installedInventoryPlaybookAnsible Core / EngineSSH / WinRM / APIsWeb Server 1App Server 1Database Server 1

The control node contains the playbooks, inventory and Ansible execution engine. Managed nodes receive and execute the required tasks.

Connection methods:

| Target                     | Connection                           |
| -------------------------- | ------------------------------------ |
| Linux / Unix               | SSH                                  |
| Windows                    | WinRM or PSRP                        |
| Network and cloud services | APIs or supported connection plugins |

Important correction: Windows management through Ansible generally uses WinRM or PSRP, not RDP. RDP is primarily for interactive remote desktop access.

For typical Linux managed nodes, Python is required for most Ansible modules. You do not need to learn Python programming to write ordinary Ansible playbooks.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 5. Push vs Pull Architecture

This is a common interview question.

Push-based — Ansible

[\[Ansible\] 보안설정/모니터링 자동화, Semaphore (스터디 4주차) :: Ho_use](https://images.openai.com/static-rsc-4/Sh2vRyt2Nsaw_y8BaFQ4U4eBuRvknaYMNOkbssjNKgF5ycs29NzeEs6ctQnhOql2J_B2lp7XREs73JnY5FaJOCLt-UMIJord2DmCg3BbGgZ9u-w3I9mambyJU4sx4fXqw9qG2rzYwGpB8N5r6mR3VPotheDL8Dex6GdURXkpEso?purpose=inline)

[xn--vj5b11biyw.kr](https://xn--vj5b11biyw.kr/342?utm_source=chatgpt.com)

- Control node initiates the connection.
- Pushes automation tasks to managed nodes.
- Usually uses SSH for Linux and WinRM/PSRP for Windows.
- Requires connectivity when the automation runs.

Pull-based — commonly Puppet and Chef

[What is Puppet?](https://images.openai.com/static-rsc-4/W_EC0FUmTctm84mUGeUW2-6EhL0hHpcUNp0rQZbfM3qDqwhA1dhElzpZ26hsQYZP6BNsaHhw-_dawtPjTWt-GHOqhagwt1nB-H4kANhLTOaaNaUoQ3uo21LRdl94u9IBFAqpDVQzxqx6pGBEJCvaCRnIbT0YvTxI_dtdAdawt1c?purpose=inline)

[help.puppet.com](https://help.puppet.com/core/current/Content/PuppetCore/what_is_puppet.htm?utm_source=chatgpt.com)

- Agent is installed on managed nodes.
- Agents retrieve configuration from a central server.
- Requires the agent and its communication path to be available.

| Push                            | Pull                       |
| ------------------------------- | -------------------------- |
| Controller initiates connection | Agent initiates connection |
| No target-side Ansible agent    | Requires target-side agent |
| Example: Ansible                | Examples: Puppet and Chef  |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 6. Important Ansible Terminology

These terms are essential before writing playbooks.

| Term         | Meaning                                                      | Example                      |
| ------------ | ------------------------------------------------------------ | ---------------------------- |
| Task         | One unit of work                                             | Install NGINX                |
| Play         | Collection of tasks applied to selected hosts                | Configure web servers        |
| Playbook     | YAML file containing one or more plays                       | `site.yml`                   |
| Inventory    | List or source of managed hosts                              | Web and DB servers           |
| Module       | Reusable unit that performs a specific operation             | `package`, `service`, `copy` |
| Role         | Reusable package of tasks, variables, handlers and templates | Web server role              |
| Facts        | Information collected about managed hosts                    | OS, IP, memory               |
| Control node | Machine running Ansible                                      | Ansible controller           |
| Managed node | Machine being configured                                     | EC2 instance                 |

### Understand Task, Play and Playbook

Imagine you want to configure NGINX on web servers.

- Task 1: Install NGINX.
- Task 2: Start NGINX.
- Task 3: Copy configuration file.
- Task 4: Enable the service.

These tasks together form a Play targeting web servers.

The YAML file containing this play is called a Playbook.

\#chatgpt-mermaid-\_r_f89\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f89\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f89\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f89\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f89\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f89\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f89\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f89\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f89\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f89\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f89\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f89\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f89\_ p{margin:0;}#chatgpt-mermaid-\_r_f89\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f89\_ .label text,#chatgpt-mermaid-\_r_f89\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ .node rect,#chatgpt-mermaid-\_r_f89\_ .node circle,#chatgpt-mermaid-\_r_f89\_ .node ellipse,#chatgpt-mermaid-\_r_f89\_ .node polygon,#chatgpt-mermaid-\_r_f89\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ .rough-node .label text,#chatgpt-mermaid-\_r_f89\_ .node .label text,#chatgpt-mermaid-\_r_f89\_ .image-shape .label,#chatgpt-mermaid-\_r_f89\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f89\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ .rough-node .label,#chatgpt-mermaid-\_r_f89\_ .node .label,#chatgpt-mermaid-\_r_f89\_ .image-shape .label,#chatgpt-mermaid-\_r_f89\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f89\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f89\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f89\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f89\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f89\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f89\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f89\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f89\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f89\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f89\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f89\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f89\_ .icon-shape,#chatgpt-mermaid-\_r_f89\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f89\_ .icon-shape p,#chatgpt-mermaid-\_r_f89\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f89\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f89\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f89\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f89\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f89\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f89\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f89\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f89\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f89\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f89\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f89\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f89\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f89\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f89\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f89\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f89\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ .node rect,#chatgpt-mermaid-\_r_f89\_ .node circle,#chatgpt-mermaid-\_r_f89\_ .node ellipse,#chatgpt-mermaid-\_r_f89\_ .node polygon,#chatgpt-mermaid-\_r_f89\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f89\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f89\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f89\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f89\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f89\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}PlaybookPlay 1: Web ServersPlay 2: App ServersPlay 3: Database ServersTask 1: Install NGINXTask 2: Start NGINXTask 3: Copy Configuration

### What is a Role?

A role is a reusable, structured collection of automation content.

For example, a `webserver` role can contain:

- Tasks to install and configure NGINX.
- Variables for ports and package versions.
- Templates for configuration files.
- Handlers to restart NGINX when configuration changes.

Instead of writing the same tasks in multiple playbooks, teams can reuse the role.

Remember:

- Task = One operation.
- Play = Tasks applied to selected hosts.
- Playbook = One or more plays.
- Role = Reusable package of automation content.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 7. Inventory — Static and Dynamic

Inventory tells Ansible which machines it should manage.

### Static inventory

You manually provide the IP addresses or hostnames.

Example: `inventory.ini`

```
[webservers]
192.168.1.10
192.168.1.11

[appservers]
192.168.1.20

[dbservers]
192.168.1.30
```

You can target only the web server group instead of all machines.

### Dynamic inventory

Useful when infrastructure changes frequently, such as AWS Auto Scaling environments.

Instead of manually updating IP addresses, Ansible can retrieve the current hosts from a cloud provider using a dynamic inventory plugin.

| Static inventory                   | Dynamic inventory                               |
| ---------------------------------- | ----------------------------------------------- |
| Manually maintained                | Automatically discovers hosts                   |
| Suitable for stable infrastructure | Suitable for frequently changing infrastructure |
| IPs or hostnames entered manually  | Uses cloud APIs or other inventory sources      |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. Ansible Modules

A module is a small reusable unit that performs a specific operation.

Think of Linux commands: `touch` creates a file, `cp` copies a file. Similarly, Ansible modules perform specific automation tasks.

| Module                    | Purpose                                   |
| ------------------------- | ----------------------------------------- |
| `ansible.builtin.user`    | Create or manage users                    |
| `ansible.builtin.package` | Install or remove packages                |
| `ansible.builtin.service` | Manage services                           |
| `ansible.builtin.copy`    | Copy files                                |
| `ansible.builtin.file`    | Manage files, directories and permissions |
| `ansible.builtin.command` | Execute commands                          |
| `ansible.builtin.ping`    | Test Ansible connectivity                 |

Modules are available through Ansible collections. You can also develop your own modules.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 9. Terraform vs Ansible

Both tools have overlapping capabilities, but their primary purposes are different.

| Terraform                              | Ansible                                               |
| -------------------------------------- | ----------------------------------------------------- |
| Infrastructure provisioning            | Configuration management                              |
| Creates EC2, VPC, load balancers, etc. | Installs packages, creates users, configures services |
| Uses HCL                               | Uses YAML playbooks                                   |
| Maintains state using a state file     | Checks actual host conditions when tasks execute      |
| Uses providers and resources           | Uses inventories, plays and modules                   |

### Real-world workflow

Suppose you need to deploy an application on AWS.

1. Terraform creates the VPC, subnet, security group and EC2 instance.
2. Ansible connects to the EC2 instance.
3. Ansible installs Java and Tomcat.
4. Ansible configures the application and starts services.
5. Jenkins can trigger the automation as part of CI/CD.

\#chatgpt-mermaid-\_r_f90\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f90\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f90\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f90\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f90\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f90\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f90\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f90\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f90\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f90\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f90\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f90\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f90\_ p{margin:0;}#chatgpt-mermaid-\_r_f90\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f90\_ .label text,#chatgpt-mermaid-\_r_f90\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ .node rect,#chatgpt-mermaid-\_r_f90\_ .node circle,#chatgpt-mermaid-\_r_f90\_ .node ellipse,#chatgpt-mermaid-\_r_f90\_ .node polygon,#chatgpt-mermaid-\_r_f90\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ .rough-node .label text,#chatgpt-mermaid-\_r_f90\_ .node .label text,#chatgpt-mermaid-\_r_f90\_ .image-shape .label,#chatgpt-mermaid-\_r_f90\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f90\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ .rough-node .label,#chatgpt-mermaid-\_r_f90\_ .node .label,#chatgpt-mermaid-\_r_f90\_ .image-shape .label,#chatgpt-mermaid-\_r_f90\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f90\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f90\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f90\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f90\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f90\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f90\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f90\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f90\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f90\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f90\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f90\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f90\_ .icon-shape,#chatgpt-mermaid-\_r_f90\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f90\_ .icon-shape p,#chatgpt-mermaid-\_r_f90\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f90\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f90\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f90\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f90\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f90\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f90\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f90\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f90\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f90\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f90\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f90\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f90\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f90\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f90\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f90\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f90\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ .node rect,#chatgpt-mermaid-\_r_f90\_ .node circle,#chatgpt-mermaid-\_r_f90\_ .node ellipse,#chatgpt-mermaid-\_r_f90\_ .node polygon,#chatgpt-mermaid-\_r_f90\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f90\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f90\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f90\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f90\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f90\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}TerraformCreate AWS InfrastructureEC2 InstanceAnsibleInstall PackagesConfigure ServicesDeploy ApplicationApplication Running

### Why not use Terraform provisioners for everything?

Terraform provisioners can run commands on a server, but they are not a replacement for a dedicated configuration management tool.

For example, changing an installation command inside a provisioner can cause resource replacement in some situations, depending on the resource and lifecycle. That can be risky for servers with attached storage.

Ansible is better suited to repeatedly managing software and configuration on existing servers.

Interview answer: Terraform provisions infrastructure; Ansible configures it. They can overlap, but using each for its main purpose is generally a better design.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 10. How Ansible Executes a Playbook

Understand this flow carefully.

\#chatgpt-mermaid-\_r_f9b\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f9b\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f9b\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f9b\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f9b\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f9b\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f9b\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f9b\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f9b\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f9b\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f9b\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f9b\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f9b\_ p{margin:0;}#chatgpt-mermaid-\_r_f9b\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f9b\_ .label text,#chatgpt-mermaid-\_r_f9b\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ .node rect,#chatgpt-mermaid-\_r_f9b\_ .node circle,#chatgpt-mermaid-\_r_f9b\_ .node ellipse,#chatgpt-mermaid-\_r_f9b\_ .node polygon,#chatgpt-mermaid-\_r_f9b\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ .rough-node .label text,#chatgpt-mermaid-\_r_f9b\_ .node .label text,#chatgpt-mermaid-\_r_f9b\_ .image-shape .label,#chatgpt-mermaid-\_r_f9b\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f9b\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ .rough-node .label,#chatgpt-mermaid-\_r_f9b\_ .node .label,#chatgpt-mermaid-\_r_f9b\_ .image-shape .label,#chatgpt-mermaid-\_r_f9b\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f9b\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f9b\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f9b\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f9b\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f9b\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f9b\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f9b\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f9b\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f9b\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f9b\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f9b\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f9b\_ .icon-shape,#chatgpt-mermaid-\_r_f9b\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f9b\_ .icon-shape p,#chatgpt-mermaid-\_r_f9b\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f9b\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f9b\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f9b\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f9b\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f9b\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f9b\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f9b\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f9b\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f9b\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f9b\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f9b\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f9b\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f9b\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f9b\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f9b\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f9b\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ .node rect,#chatgpt-mermaid-\_r_f9b\_ .node circle,#chatgpt-mermaid-\_r_f9b\_ .node ellipse,#chatgpt-mermaid-\_r_f9b\_ .node polygon,#chatgpt-mermaid-\_r_f9b\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f9b\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f9b\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f9b\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f9b\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f9b\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Run ansible-playbookcommandRead Playbook and InventorySelect Target HostsConnect to HostsGather FactsSend Required ModulesExecute on Managed NodesReturn ResultsClean Up Temporary Files

### Execution details

1. Ansible reads the playbook and inventory.
2. It selects the hosts specified in the play.
3. It establishes connections to those hosts.
4. It normally gathers facts about the hosts.
5. It sends the required modules and arguments to the managed nodes.
6. Modules execute on the target machines.
7. Results are returned to the control node.
8. Temporary execution files are cleaned up.

Important: The actual configuration operation happens on the managed node, not on the control node.

### Parallel execution

Ansible can execute tasks across multiple hosts concurrently.

- The `forks` setting controls the maximum number of simultaneous worker processes.
- The commonly used default is 5.
- It can be configured in `ansible.cfg`.

With the default linear strategy, Ansible completes a task across the active hosts before moving to the next task. It does not necessarily finish the entire play on one host before starting the next host.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 11. Facts and Fact Gathering

Facts are information Ansible collects about managed nodes.

Examples:

- Operating system and distribution.
- IP addresses.
- Memory.
- Hostname.
- Network interfaces.
- System details.

Why are facts useful?

Imagine managing two servers:

- Ubuntu uses `apt` for package management.
- Amazon Linux uses `dnf`.

Ansible can gather facts and use the appropriate package management behaviour for each operating system.

Facts are normally gathered automatically at the beginning of a play. You can disable this with `gather_facts: false` when appropriate.

### Example

```
- name: Display host information
  hosts: all
  tasks:
    - name: Print operating system
      ansible.builtin.debug:
        var: ansible_distribution
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 12. Idempotency — Very Important

Idempotency means running the same automation repeatedly produces the same desired result without unnecessary changes.

Example: Create a user called `deploy`.

| Execution  | Actual condition              | Ansible result             |
| ---------- | ----------------------------- | -------------------------- |
| First run  | User does not exist           | `changed` — creates user   |
| Second run | User already exists           | `ok` — no change           |
| Third run  | Someone manually deletes user | `changed` — recreates user |

Ansible does not maintain a Terraform-style state file. It checks the current condition of the managed system and compares it with the condition described by the task.

### Compare the three tools

| Tool       | How it reconciles state                                        |
| ---------- | -------------------------------------------------------------- |
| Terraform  | Uses its state file and refresh/plan mechanisms                |
| Kubernetes | Uses controllers to reconcile actual and desired cluster state |
| Ansible    | Checks managed hosts when a playbook runs                      |

Important distinction: Ansible does not continuously enforce configuration by default. You need to run the playbook again, schedule it or integrate it into an automation workflow.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2

## 13. Ad-hoc Commands vs Playbooks

You do not always need to create a YAML playbook. For simple one-time operations, Ansible supports ad-hoc commands.

### Ad-hoc command

Example: Create a file on localhost.

```
ansible localhost -m ansible.builtin.file \
  -a "path=/tmp/file.txt state=touch" \
  -c local
```

Here:

- `localhost` = target machine.
- `-m` = module to execute.
- `ansible.builtin.file` = file module.
- `-a` = module arguments.
- `state=touch` = create the file if needed.

To delete the file:

```
ansible localhost -m ansible.builtin.file \
  -a "path=/tmp/file.txt state=absent" \
  -c local
```

### Ad-hoc vs Playbook

| Ad-hoc command                         | Playbook                               |
| -------------------------------------- | -------------------------------------- |
| Single, quick operation                | Multiple organized tasks               |
| Executed directly in terminal          | Written in YAML                        |
| Useful for testing and troubleshooting | Useful for repeatable automation       |
| Less structured                        | Easier to maintain and version-control |

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 14. Quick Revision — Interview Questions

| Question                                      | Short answer                                                  |
| --------------------------------------------- | ------------------------------------------------------------- |
| What is Ansible?                              | Agentless automation and configuration management tool.       |
| What is a control node?                       | Machine where Ansible is installed and runs.                  |
| What is a managed node?                       | Machine configured by Ansible.                                |
| What is a playbook?                           | YAML file containing one or more plays.                       |
| What is a task?                               | A single unit of automation.                                  |
| What is a module?                             | A reusable unit that performs a particular operation.         |
| What is a role?                               | Reusable package of tasks, variables, handlers and templates. |
| What is inventory?                            | List or source of managed hosts.                              |
| What is dynamic inventory?                    | Automatically discovers hosts from sources such as AWS.       |
| What is idempotency?                          | Repeated execution does not make unnecessary changes.         |
| Does Ansible use a state file like Terraform? | No. It checks the managed host when tasks run.                |
| What is fact gathering?                       | Collecting system information from managed hosts.             |
| What is the default forks value?              | Commonly 5.                                                   |
| Can Ansible integrate with Jenkins?           | Yes, Jenkins can execute Ansible playbooks in a pipeline.     |

## 15. Mental Model

Remember these five points:

1. Terraform — Creates infrastructure.
2. Ansible — Configures servers.
3. Inventory — Identifies which servers to manage.
4. Playbook — Describes what configuration to apply.
5. Modules — Perform the actual operations.

One-line workflow

## Terraform → EC2 → Ansible → Application Configuration

This is the basic infrastructure-to-configuration workflow used in many DevOps environments.

Original transcript: [DevOps Session — 30 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session\(Morning\)%20transcript_2026-10-30_09.24.55.txt?utm_source=chatgpt.com)

Session complete: This class introduced Ansible fundamentals and its execution model. The next useful step is hands-on practice: install Ansible, create an inventory, write a playbook and configure a managed EC2 instance.