# DevOps Session — Terraform for_each, Dynamic Blocks, Provisioners & Workspaces Notes

Date: 26 October 2026\
Instructor: Rochak Agrawal\
Focus: Terraform iteration, `for_each`, dynamic blocks, provisioners and managing multiple environments with workspaces.

These notes are condensed from the transcript. Participant discussions and repetitions have been removed, speech-to-text errors corrected, and important technical clarifications added.

## 1. Terraform looping — count vs for_each

Terraform provides two commonly used meta-arguments to create multiple instances of a resource.

| Feature          | `count`                            | `for_each`                                           |
| ---------------- | ---------------------------------- | ---------------------------------------------------- |
| Identification   | Numeric index                      | Map key or set element                               |
| Resource address | `resource.name[0]`                 | `resource.name["key"]`                               |
| Best suited for  | Similar, interchangeable instances | Instances with distinct identities or configurations |
| Example          | 3 identical EC2 instances          | Web, app and DB servers with different sizes         |

### 1.1 Using for_each with a set

Example: Create three IAM users.

```
locals {
  iam_users = toset(["alice", "bob", "charlie"])
}

resource "aws_iam_user" "users" {
  for_each = local.iam_users
  name     = each.value
}
```

How it works:

- `toset()` converts the list into a set and removes duplicates.
- `for_each` iterates over each set element.
- `each.value` provides the current username.
- Each user becomes a separate AWS IAM resource.

Resource addresses:

```
aws_iam_user.users["alice"]
aws_iam_user.users["bob"]
aws_iam_user.users["charlie"]
```

### 1.2 Using for_each with a map

Use a map when each resource needs different configuration.

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

  tags = {
    Name = each.key
  }
}
```

Here:

- `each.key` identifies the server (`web`, `app`, `db`).
- `each.value.instance_type` retrieves its instance size.

Result: Three EC2 instances with different configurations.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 2. Dynamic blocks

This is the most important concept from the session.

A normal `for_each` creates multiple resource instances. A `dynamic` block generates repeated nested configuration blocks inside a single resource.

Example: One AWS Security Group can contain multiple inbound rules:

- HTTP — port 80
- HTTPS — port 443
- SSH — port 22

Instead of repeating the ingress block manually, use a dynamic block.

### Example: Security Group with dynamic ingress rules

```
locals {
  ingress_rules = {
    http = {
      port        = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }

    https = {
      port        = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }

    ssh = {
      port        = 22
      protocol    = "tcp"
      cidr_blocks = ["YOUR_PUBLIC_IP/32"]
    }
  }
}

resource "aws_security_group" "web" {
  name = "web-security-group"

  dynamic "ingress" {
    for_each = local.ingress_rules

    content {
      description = ingress.key
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

### Understanding the dynamic block

| Keyword             | Meaning                                      |
| ------------------- | -------------------------------------------- |
| `dynamic "ingress"` | Generates repeated ingress blocks            |
| `for_each`          | Iterates over the supplied collection        |
| `content`           | Defines the contents of each generated block |
| `ingress.key`       | Current map key                              |
| `ingress.value`     | Current map value                            |

The result is one Security Group with three ingress rules, not three Security Groups.

Key distinction

### for_each vs dynamic for_each

| Normal for_each                  | Dynamic block                                                                |
| -------------------------------- | ---------------------------------------------------------------------------- |
| Creates multiple resources       | Generates multiple nested blocks                                             |
| Example: three Security Groups   | Example: three ingress rules inside one Security Group                       |
| Uses `each.key` and `each.value` | Uses the dynamic block's iterator, such as `ingress.key` and `ingress.value` |

### Mermaid diagram — understanding the difference

\#chatgpt-mermaid-\_r_f28\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f28\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f28\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f28\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f28\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f28\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f28\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f28\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f28\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f28\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f28\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f28\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f28\_ p{margin:0;}#chatgpt-mermaid-\_r_f28\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f28\_ .label text,#chatgpt-mermaid-\_r_f28\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ .node rect,#chatgpt-mermaid-\_r_f28\_ .node circle,#chatgpt-mermaid-\_r_f28\_ .node ellipse,#chatgpt-mermaid-\_r_f28\_ .node polygon,#chatgpt-mermaid-\_r_f28\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ .rough-node .label text,#chatgpt-mermaid-\_r_f28\_ .node .label text,#chatgpt-mermaid-\_r_f28\_ .image-shape .label,#chatgpt-mermaid-\_r_f28\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f28\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ .rough-node .label,#chatgpt-mermaid-\_r_f28\_ .node .label,#chatgpt-mermaid-\_r_f28\_ .image-shape .label,#chatgpt-mermaid-\_r_f28\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f28\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f28\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f28\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f28\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f28\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f28\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f28\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f28\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f28\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f28\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f28\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f28\_ .icon-shape,#chatgpt-mermaid-\_r_f28\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f28\_ .icon-shape p,#chatgpt-mermaid-\_r_f28\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f28\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f28\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f28\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f28\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f28\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f28\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f28\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f28\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f28\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f28\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f28\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f28\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f28\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f28\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f28\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f28\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ .node rect,#chatgpt-mermaid-\_r_f28\_ .node circle,#chatgpt-mermaid-\_r_f28\_ .node ellipse,#chatgpt-mermaid-\_r_f28\_ .node polygon,#chatgpt-mermaid-\_r_f28\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f28\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f28\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f28\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f28\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f28\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Terraform configurationWhat needs repeating?for_eachdynamic block3 Security Groups1 Security GroupHTTP ruleHTTPS ruleSSH ruleEntire resourcesNested configuration

Other dynamic block use cases:

- Multiple EBS block-device configurations.
- Multiple listener or backend blocks where supported.
- Multiple ingress and egress rules.
- Repeated nested configuration supported by a provider resource.

Remember: A dynamic block can only generate nested blocks supported by that resource's schema. It cannot generate arbitrary resource blocks.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 3. Three ways of implementing Terraform loops

| Requirement                                           | Approach        |
| ----------------------------------------------------- | --------------- |
| Create 3 similar EC2 instances                        | `count`         |
| Create web, app and DB instances with different sizes | `for_each`      |
| Create one Security Group with multiple ingress rules | `dynamic` block |

You can also combine `for_each` with a dynamic block.

For example, create three Security Groups, each with its own set of ingress rules.

\#chatgpt-mermaid-\_r_f2j\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f2j\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f2j\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f2j\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f2j\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f2j\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f2j\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f2j\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f2j\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f2j\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f2j\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f2j\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f2j\_ p{margin:0;}#chatgpt-mermaid-\_r_f2j\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f2j\_ .label text,#chatgpt-mermaid-\_r_f2j\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ .node rect,#chatgpt-mermaid-\_r_f2j\_ .node circle,#chatgpt-mermaid-\_r_f2j\_ .node ellipse,#chatgpt-mermaid-\_r_f2j\_ .node polygon,#chatgpt-mermaid-\_r_f2j\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ .rough-node .label text,#chatgpt-mermaid-\_r_f2j\_ .node .label text,#chatgpt-mermaid-\_r_f2j\_ .image-shape .label,#chatgpt-mermaid-\_r_f2j\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f2j\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ .rough-node .label,#chatgpt-mermaid-\_r_f2j\_ .node .label,#chatgpt-mermaid-\_r_f2j\_ .image-shape .label,#chatgpt-mermaid-\_r_f2j\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f2j\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f2j\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f2j\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f2j\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f2j\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f2j\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f2j\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f2j\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f2j\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f2j\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f2j\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f2j\_ .icon-shape,#chatgpt-mermaid-\_r_f2j\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f2j\_ .icon-shape p,#chatgpt-mermaid-\_r_f2j\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f2j\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f2j\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f2j\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f2j\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f2j\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f2j\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f2j\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f2j\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f2j\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f2j\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f2j\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f2j\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f2j\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f2j\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f2j\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f2j\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ .node rect,#chatgpt-mermaid-\_r_f2j\_ .node circle,#chatgpt-mermaid-\_r_f2j\_ .node ellipse,#chatgpt-mermaid-\_r_f2j\_ .node polygon,#chatgpt-mermaid-\_r_f2j\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f2j\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f2j\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f2j\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f2j\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f2j\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}for_each: Security GroupsWeb SGApp SGDB SGDynamic ingress rulesDynamic ingress rulesDynamic ingress rules

This is useful when different application tiers need different network access rules.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 4. Terraform provisioners

Provisioners execute commands or transfer files during Terraform resource operations.

The session focused on two execution locations:

| Provisioner   | Execution location              | Example                          |
| ------------- | ------------------------------- | -------------------------------- |
| `local-exec`  | Machine running Terraform       | Run a local script               |
| `remote-exec` | Remote server                   | Install NGINX on EC2             |
| `file`        | Copies files to a remote server | Upload application configuration |

### 4.1 Remote-exec example

The instructor demonstrated creating an EC2 instance and installing NGINX automatically.

```
resource "aws_instance" "web" {
  ami                    = "ami-xxxxxxxx"
  instance_type          = "t3.micro"
  key_name               = "my-key"

  vpc_security_group_ids = ["sg-xxxxxxxx"]

  provisioner "remote-exec" {
    inline = [
      "sudo dnf install nginx -y",
      "sudo systemctl enable --now nginx"
    ]
  }

  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("~/.ssh/my-key.pem")
    host        = self.public_ip
  }
}

output "public_ip" {
  value = aws_instance.web.public_ip
}
```

Execution sequence:

1. Terraform creates the EC2 instance.
2. It obtains the instance's public IP.
3. Terraform connects using SSH.
4. It authenticates using the private key.
5. It executes the commands in `inline`.
6. NGINX gets installed and started.

### Mermaid diagram — remote-exec workflow

\#chatgpt-mermaid-\_r_f32\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f32\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f32\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f32\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f32\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f32\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f32\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f32\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f32\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f32\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f32\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f32\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f32\_ p{margin:0;}#chatgpt-mermaid-\_r_f32\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f32\_ .label text,#chatgpt-mermaid-\_r_f32\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ .node rect,#chatgpt-mermaid-\_r_f32\_ .node circle,#chatgpt-mermaid-\_r_f32\_ .node ellipse,#chatgpt-mermaid-\_r_f32\_ .node polygon,#chatgpt-mermaid-\_r_f32\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ .rough-node .label text,#chatgpt-mermaid-\_r_f32\_ .node .label text,#chatgpt-mermaid-\_r_f32\_ .image-shape .label,#chatgpt-mermaid-\_r_f32\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f32\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ .rough-node .label,#chatgpt-mermaid-\_r_f32\_ .node .label,#chatgpt-mermaid-\_r_f32\_ .image-shape .label,#chatgpt-mermaid-\_r_f32\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f32\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f32\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f32\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f32\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f32\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f32\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f32\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f32\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f32\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f32\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f32\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f32\_ .icon-shape,#chatgpt-mermaid-\_r_f32\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f32\_ .icon-shape p,#chatgpt-mermaid-\_r_f32\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f32\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f32\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f32\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f32\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f32\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f32\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f32\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f32\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f32\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f32\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f32\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f32\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f32\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f32\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f32\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f32\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ .node rect,#chatgpt-mermaid-\_r_f32\_ .node circle,#chatgpt-mermaid-\_r_f32\_ .node ellipse,#chatgpt-mermaid-\_r_f32\_ .node polygon,#chatgpt-mermaid-\_r_f32\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f32\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f32\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f32\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f32\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f32\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}terraform applyCreate EC2 instanceObtain public IPConnect through SSHAuthentication successful?Connection failsExecute remote commandsInstall NGINXStart NGINXProvisioning completeNoYes

### 4.2 Common remote-exec errors

The instructor encountered two important problems.

| Error                      | Cause                                             |
| -------------------------- | ------------------------------------------------- |
| `command not found`        | Used Ubuntu commands on a CentOS-based server     |
| SSH authentication failure | Incorrect private key or mismatched key pair      |
| Connection timeout         | Network, security group or SSH connectivity issue |

Important: The Linux distribution determines which package manager and commands should be used.

| Distribution      | Package manager |
| ----------------- | --------------- |
| Ubuntu/Debian     | `apt`           |
| Amazon Linux 2023 | `dnf`           |
| Modern RHEL       | `dnf`           |

The private key must correspond to the public key configured for the EC2 instance.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 5. Terraform vs Ansible — infrastructure provisioning and configuration management

A question raised during the session was: If Ansible is used to configure servers, why do we need Terraform provisioners?

The answer is that infrastructure provisioning and configuration management are related but different responsibilities.

| Terraform                                      | Ansible                                                  |
| ---------------------------------------------- | -------------------------------------------------------- |
| Primarily provisions infrastructure            | Primarily manages server configuration                   |
| Creates EC2, VPC, subnets and security groups  | Installs packages, creates users and configures services |
| Declarative infrastructure management          | Automates configuration tasks                            |
| Has provisioners for limited command execution | Has broader configuration-management capabilities        |

Example DevOps workflow:

\#chatgpt-mermaid-\_r_f3d\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f3d\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f3d\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f3d\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f3d\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f3d\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f3d\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f3d\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f3d\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f3d\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f3d\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f3d\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f3d\_ p{margin:0;}#chatgpt-mermaid-\_r_f3d\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f3d\_ .label text,#chatgpt-mermaid-\_r_f3d\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ .node rect,#chatgpt-mermaid-\_r_f3d\_ .node circle,#chatgpt-mermaid-\_r_f3d\_ .node ellipse,#chatgpt-mermaid-\_r_f3d\_ .node polygon,#chatgpt-mermaid-\_r_f3d\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ .rough-node .label text,#chatgpt-mermaid-\_r_f3d\_ .node .label text,#chatgpt-mermaid-\_r_f3d\_ .image-shape .label,#chatgpt-mermaid-\_r_f3d\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f3d\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ .rough-node .label,#chatgpt-mermaid-\_r_f3d\_ .node .label,#chatgpt-mermaid-\_r_f3d\_ .image-shape .label,#chatgpt-mermaid-\_r_f3d\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f3d\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f3d\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f3d\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f3d\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f3d\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f3d\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f3d\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f3d\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f3d\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f3d\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f3d\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f3d\_ .icon-shape,#chatgpt-mermaid-\_r_f3d\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f3d\_ .icon-shape p,#chatgpt-mermaid-\_r_f3d\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f3d\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f3d\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f3d\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f3d\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f3d\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f3d\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f3d\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f3d\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f3d\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f3d\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f3d\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f3d\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f3d\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f3d\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f3d\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f3d\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ .node rect,#chatgpt-mermaid-\_r_f3d\_ .node circle,#chatgpt-mermaid-\_r_f3d\_ .node ellipse,#chatgpt-mermaid-\_r_f3d\_ .node polygon,#chatgpt-mermaid-\_r_f3d\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f3d\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f3d\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f3d\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f3d\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f3d\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}TerraformCreate EC2 instanceAnsible connects to EC2Install Java and NGINXConfigure applicationDeploy application

Best practice: Prefer Ansible, cloud-init, user data or prebuilt images for substantial server configuration. Terraform provisioners are generally a last resort for simple tasks.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 6. Managing multiple environments — the state file problem

This was another major topic in the session.

Imagine you maintain two environments:

| Environment | Instance type |
| ----------- | ------------- |
| Development | `t3.micro`    |
| Production  | `t3.large`    |

You have two variable files:

`dev.tfvars`

```
instance_type = "t3.micro"
```

`prod.tfvars`

```
instance_type = "t3.large"
```

### What is the problem with using both in one working directory?

If you use the same configuration and the same state file:

1. Apply `prod.tfvars` and Terraform creates a production instance.
2. State records the production resource.
3. Apply `dev.tfvars` using that same state.
4. Terraform sees that the desired configuration has changed.
5. It may plan to modify or replace the production instance.

This can accidentally affect production infrastructure.

The important issue is not the `.tfvars` files themselves. It is sharing one state between independent environments.

### Solution 1: Separate folders

Maintain separate directories for each environment.

```
terraform/
├── dev/
│   ├── main.tf
│   ├── variables.tf
│   └── dev.tfvars
│
└── prod/
    ├── main.tf
    ├── variables.tf
    └── prod.tfvars
```

Each directory has its own Terraform working directory and state.

Advantage: Clear separation.

Disadvantage: Some configuration may need to be duplicated or maintained through shared modules.

### Solution 2: Terraform workspaces

Terraform workspaces allow you to use the same configuration with separate state instances.

For example:

- `default`
- `dev`
- `prod`

Each workspace tracks its own resources in a separate state.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 7. Terraform workspace commands

| Command                           | Purpose                                 |
| --------------------------------- | --------------------------------------- |
| `terraform workspace list`        | Lists available workspaces              |
| `terraform workspace show`        | Shows the current workspace             |
| `terraform workspace new dev`     | Creates and switches to a new workspace |
| `terraform workspace select prod` | Switches to an existing workspace       |
| `terraform workspace delete dev`  | Deletes a workspace                     |

### Example workflow

```
# List workspaces
terraform workspace list

# Create development workspace
terraform workspace new dev

# Check current workspace
terraform workspace show

# Create production workspace
terraform workspace new prod

# Switch to development
terraform workspace select dev

# Preview changes
terraform plan

# Apply changes
terraform apply
```

### Using terraform.workspace

Terraform provides a built-in expression:

```
terraform.workspace
```

It returns the currently selected workspace name.

Example:

```
resource "aws_s3_bucket" "app" {
  bucket = "meridian-app-${terraform.workspace}"
}
```

This generates different bucket names:

| Workspace | Bucket name            |
| --------- | ---------------------- |
| default   | `meridian-app-default` |
| dev       | `meridian-app-dev`     |
| prod      | `meridian-app-prod`    |

S3 bucket names must be globally unique, so production configurations should include an appropriate unique suffix as well.

### Workspace state structure

With the local backend, the state files are stored approximately like this:

```
terraform-project/
├── terraform.tfstate
└── terraform.tfstate.d/
    ├── dev/
    │   └── terraform.tfstate
    └── prod/
        └── terraform.tfstate
```

The default workspace uses the root `terraform.tfstate`; named workspaces have separate state files.

### Mermaid diagram — workspace isolation

\#chatgpt-mermaid-\_r_f4p\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_f4p\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f4p\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_f4p\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_f4p\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_f4p\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_f4p\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_f4p\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_f4p\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_f4p\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f4p\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f4p\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_f4p\_ p{margin:0;}#chatgpt-mermaid-\_r_f4p\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_f4p\_ .label text,#chatgpt-mermaid-\_r_f4p\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ .node rect,#chatgpt-mermaid-\_r_f4p\_ .node circle,#chatgpt-mermaid-\_r_f4p\_ .node ellipse,#chatgpt-mermaid-\_r_f4p\_ .node polygon,#chatgpt-mermaid-\_r_f4p\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ .rough-node .label text,#chatgpt-mermaid-\_r_f4p\_ .node .label text,#chatgpt-mermaid-\_r_f4p\_ .image-shape .label,#chatgpt-mermaid-\_r_f4p\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_f4p\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ .rough-node .label,#chatgpt-mermaid-\_r_f4p\_ .node .label,#chatgpt-mermaid-\_r_f4p\_ .image-shape .label,#chatgpt-mermaid-\_r_f4p\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_f4p\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_f4p\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f4p\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f4p\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_f4p\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f4p\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f4p\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f4p\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_f4p\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_f4p\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_f4p\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_f4p\_ .icon-shape,#chatgpt-mermaid-\_r_f4p\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_f4p\_ .icon-shape p,#chatgpt-mermaid-\_r_f4p\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_f4p\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_f4p\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_f4p\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_f4p\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_f4p\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_f4p\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_f4p\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_f4p\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_f4p\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f4p\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_f4p\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_f4p\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f4p\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_f4p\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_f4p\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_f4p\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ .node rect,#chatgpt-mermaid-\_r_f4p\_ .node circle,#chatgpt-mermaid-\_r_f4p\_ .node ellipse,#chatgpt-mermaid-\_r_f4p\_ .node polygon,#chatgpt-mermaid-\_r_f4p\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_f4p\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_f4p\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_f4p\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_f4p\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_f4p\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}One Terraform configurationWorkspace selectiondefault workspacedev workspaceprod workspaceDefault stateDevelopment stateProduction state

Important correction for real-world projects: Workspaces separate state, but they do not provide complete environment isolation. Workspaces generally share the same backend configuration and credentials context. For critical production environments, separate accounts, permissions, backends and deployment pipelines are often safer than relying only on workspaces.

Also, deleting a workspace is not the same as destroying its infrastructure. Destroy resources in that workspace before deleting it.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 8. Quick revision — interview questions

| Question                                         | Short answer                                                                                  |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------- |
| What is `for_each`?                              | Creates multiple resource instances using map keys or set elements.                           |
| What is a dynamic block?                         | Generates repeated nested configuration blocks inside a resource.                             |
| Difference between `for_each` and dynamic block? | One creates resources; the other creates nested blocks.                                       |
| What is `each.value`?                            | The current value in a `for_each` iteration.                                                  |
| What is `local-exec`?                            | Executes commands on the Terraform machine.                                                   |
| What is `remote-exec`?                           | Executes commands on a remote machine.                                                        |
| Why does remote-exec need a connection block?    | It needs connection details and authentication to access the remote server.                   |
| Why are provisioners discouraged?                | They are harder to manage reliably than dedicated configuration tools.                        |
| What is a Terraform workspace?                   | A separate state instance using the same Terraform configuration.                             |
| Why use separate states?                         | To prevent changes in one environment from affecting another environment's managed resources. |
| What does `terraform.workspace` return?          | The currently selected workspace name.                                                        |

## 9. Final mental model

| Concept          | Remember it as                                           |
| ---------------- | -------------------------------------------------------- |
| `count`          | Repeat similar resource instances                        |
| `for_each`       | Create resources with distinct identities/configurations |
| `dynamic`        | Repeat nested blocks inside a resource                   |
| `remote-exec`    | Run commands on a remote server                          |
| Ansible          | Configure and manage servers                             |
| Separate folders | Separate Terraform working directories and states        |
| Workspaces       | Same configuration, separate state instances             |

Most important takeaway: Understand what you are repeating in Terraform. If you are repeating entire resources, use `count` or `for_each`. If you are repeating nested configuration inside one resource, use a dynamic block.

Original transcript: [DevOps Session — 26 October 2026](https://github.com/jeevanadv2000-pixel/python/blob/main/DevOps%20Session%28Morning%29%20transcript_2026-10-26_09.28.00.txt?utm_source=chatgpt.com).