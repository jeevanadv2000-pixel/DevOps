# Azure – Class Notes (Morning Session)

Date: 2 November 2026 | Instructor: Rochak Agrawal

## 1. Introduction to Microsoft Azure

- Azure is Microsoft's cloud computing platform.
- Major cloud providers discussed:
  - AWS
  - Microsoft Azure
  - Google Cloud Platform (GCP)
- Around 90% of cloud concepts are similar across providers, although terminology and implementation may differ.
- The session focuses on understanding Azure infrastructure and administration.

## 2. Azure Global Infrastructure

Azure infrastructure is organized geographically into Regions, Availability Zones and Data Centers.

### 2.1 Region

A region is a geographical location containing one or more data centers in close proximity.

Why choose a region?

1. User location: Deploy close to customers to reduce network latency.
2. Cost: Pricing may vary between regions.
3. Compliance: Certain regulations require customer data to remain within specific geographical boundaries.
4. Service availability: Not every Azure service is available in every region.

Example: An Indian financial institution may need to keep Indian customer data within India due to applicable data residency requirements.

### 2.2 Availability Zones

- Availability Zones provide protection against data center failures.
- They are physically separate locations within the same Azure region.
- Each zone has independent power, cooling and networking.
- Zones are connected through private fiber-optic networks.

### 2.3 Data Center

A physical facility where Azure infrastructure and resources run.

Infrastructure hierarchy (conceptual):

\#chatgpt-mermaid-\_r_hac\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_hac\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hac\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hac\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_hac\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_hac\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_hac\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_hac\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_hac\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_hac\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hac\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hac\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_hac\_ p{margin:0;}#chatgpt-mermaid-\_r_hac\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_hac\_ .label text,#chatgpt-mermaid-\_r_hac\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ .node rect,#chatgpt-mermaid-\_r_hac\_ .node circle,#chatgpt-mermaid-\_r_hac\_ .node ellipse,#chatgpt-mermaid-\_r_hac\_ .node polygon,#chatgpt-mermaid-\_r_hac\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ .rough-node .label text,#chatgpt-mermaid-\_r_hac\_ .node .label text,#chatgpt-mermaid-\_r_hac\_ .image-shape .label,#chatgpt-mermaid-\_r_hac\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_hac\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ .rough-node .label,#chatgpt-mermaid-\_r_hac\_ .node .label,#chatgpt-mermaid-\_r_hac\_ .image-shape .label,#chatgpt-mermaid-\_r_hac\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_hac\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_hac\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hac\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hac\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_hac\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hac\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hac\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hac\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_hac\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_hac\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hac\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_hac\_ .icon-shape,#chatgpt-mermaid-\_r_hac\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hac\_ .icon-shape p,#chatgpt-mermaid-\_r_hac\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_hac\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_hac\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hac\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_hac\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_hac\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_hac\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_hac\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_hac\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_hac\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hac\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_hac\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hac\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hac\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hac\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_hac\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_hac\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ .node rect,#chatgpt-mermaid-\_r_hac\_ .node circle,#chatgpt-mermaid-\_r_hac\_ .node ellipse,#chatgpt-mermaid-\_r_hac\_ .node polygon,#chatgpt-mermaid-\_r_hac\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hac\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_hac\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_hac\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_hac\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hac\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Azure Global InfrastructureRegion - IndiaRegion - USAvailability Zone 1Availability Zone 2Data CenterData CenterServers and InfrastructureServers and Infrastructure

### 2.4 Availability and SLA

SLA (Service Level Agreement) defines the availability commitment for a service.

| Deployment approach          | Protection                                             |
| ---------------------------- | ------------------------------------------------------ |
| Single VM                    | Vulnerable to failure of its hosting infrastructure    |
| Availability Zone deployment | Protects against individual data center failures       |
| Multi-region deployment      | Supports disaster recovery across geographical regions |

The instructor discussed 99.9% availability and approximately 8 hours 46 minutes of potential downtime per year.

Note: SLA values depend on the Azure service and deployment configuration. The transcript's numerical examples are not fully consistent, so verify the specific service SLA before using them.

## 3. Azure Resources

A resource is an individual instance of an Azure service.

Examples:

| Azure service | Resource example               |
| ------------- | ------------------------------ |
| Compute       | Virtual Machine                |
| Networking    | Load Balancer, Virtual Network |
| Storage       | Storage Account                |
| Database      | Azure Database                 |
| Data          | Azure Data Factory             |

Every resource must belong to:

- One Azure subscription.
- One resource group.

## 4. Azure Resource Group

A Resource Group (RG) is a logical container used to organize and manage related Azure resources.

Example: An application may require:

- Web server
- Application server
- Database
- Load balancer

All these resources can be organized within one resource group.

### Important characteristics

1. A resource can belong to only one resource group at a time.
2. An application can have multiple resource groups.
3. Resources within one resource group can reside in different regions.
4. A resource group itself does not incur resource consumption charges.
5. Deleting a resource group deletes the resources contained within it.
6. Resource groups can be moved between subscriptions, subject to applicable constraints.

### Why use Resource Groups?

- Manage related resources together.
- Apply permissions at group level.
- Organize resources by application, environment or business unit.
- Delete multiple related resources together.

### Resource Group organization strategies

| Strategy            | Example                  |
| ------------------- | ------------------------ |
| Application-based   | Banking application RG   |
| Environment-based   | Dev RG, Test RG, Prod RG |
| Location-based      | Europe RG, Australia RG  |
| Business unit-based | HR RG, Finance RG        |

Important: Resource group location and resource location are different concepts.

The resource group's selected region stores its metadata. The actual resources can reside in different regions.

## 5. Azure Resource Manager (ARM)

Azure Resource Manager is the management layer through which Azure resource operations are handled.

It enables users to:

- Create resources.
- Update resources.
- Delete resources.
- Manage resources within subscriptions.

### How ARM works

Whenever a user requests an operation, the request goes through ARM.

\#chatgpt-mermaid-\_r_haj\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_haj\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_haj\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_haj\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_haj\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_haj\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_haj\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_haj\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_haj\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_haj\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haj\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haj\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_haj\_ p{margin:0;}#chatgpt-mermaid-\_r_haj\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_haj\_ .label text,#chatgpt-mermaid-\_r_haj\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ .node rect,#chatgpt-mermaid-\_r_haj\_ .node circle,#chatgpt-mermaid-\_r_haj\_ .node ellipse,#chatgpt-mermaid-\_r_haj\_ .node polygon,#chatgpt-mermaid-\_r_haj\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ .rough-node .label text,#chatgpt-mermaid-\_r_haj\_ .node .label text,#chatgpt-mermaid-\_r_haj\_ .image-shape .label,#chatgpt-mermaid-\_r_haj\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_haj\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ .rough-node .label,#chatgpt-mermaid-\_r_haj\_ .node .label,#chatgpt-mermaid-\_r_haj\_ .image-shape .label,#chatgpt-mermaid-\_r_haj\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_haj\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_haj\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haj\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haj\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_haj\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_haj\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_haj\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_haj\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_haj\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_haj\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haj\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_haj\_ .icon-shape,#chatgpt-mermaid-\_r_haj\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_haj\_ .icon-shape p,#chatgpt-mermaid-\_r_haj\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_haj\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_haj\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_haj\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_haj\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_haj\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_haj\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_haj\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_haj\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_haj\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haj\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_haj\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haj\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_haj\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_haj\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_haj\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_haj\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ .node rect,#chatgpt-mermaid-\_r_haj\_ .node circle,#chatgpt-mermaid-\_r_haj\_ .node ellipse,#chatgpt-mermaid-\_r_haj\_ .node polygon,#chatgpt-mermaid-\_r_haj\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_haj\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_haj\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_haj\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_haj\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haj\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}User RequestManagement InterfaceAzure PortalAzure CLIAzure PowerShellREST API / SDKAzure Resource ManagerAuthenticationAuthorizationAzure Backend ServicesResource Created / Updated /Deleted

ARM validates authentication and authorization before passing the request to backend services.

Key point: Regardless of which management interface you use, the requests go through ARM.

## 6. Azure Subscription

An Azure Subscription is a logical boundary that provides access to Azure resources.

Every resource group and resource belongs to a subscription.

### Two major purposes

| Purpose                 | Explanation                                                          |
| ----------------------- | -------------------------------------------------------------------- |
| Billing boundary        | Azure tracks and bills resource consumption against the subscription |
| Access control boundary | Permissions can be managed at subscription level                     |

### Important points

- An Azure account can contain multiple subscriptions.
- A subscription can contain multiple resource groups.
- You cannot create Azure resources without an active subscription and appropriate permissions.
- Each subscription has a unique Subscription ID.
- Cost Management and billing information are available at subscription level.

### AWS vs Azure

| AWS                                        | Azure                                                  |
| ------------------------------------------ | ------------------------------------------------------ |
| AWS Account                                | Azure Subscription (rough billing-boundary comparison) |
| Billing is associated with the AWS account | Billing is associated with the Azure subscription      |
| Account can contain multiple workloads     | Azure account can contain multiple subscriptions       |

Remember: An Azure account and Azure subscription are not the same thing.

An Azure account provides the identity context; the subscription provides the billing and resource-management boundary.

## 7. Management Groups

Management Groups help organizations manage multiple Azure subscriptions together.

They are useful when an organization has several teams, applications and subscriptions.

### Why use Management Groups?

- Apply organization-wide policies.
- Manage access across multiple subscriptions.
- Organize subscriptions according to departments or business units.
- Enforce common governance rules.

Example:

An organization wants to restrict all teams to creating resources only in India. Instead of configuring the restriction separately in every subscription, an organization-wide policy can be applied at the management group level.

### Management Group hierarchy

\#chatgpt-mermaid-\_r_haq\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_haq\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_haq\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_haq\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_haq\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_haq\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_haq\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_haq\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_haq\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_haq\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haq\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haq\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_haq\_ p{margin:0;}#chatgpt-mermaid-\_r_haq\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_haq\_ .label text,#chatgpt-mermaid-\_r_haq\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ .node rect,#chatgpt-mermaid-\_r_haq\_ .node circle,#chatgpt-mermaid-\_r_haq\_ .node ellipse,#chatgpt-mermaid-\_r_haq\_ .node polygon,#chatgpt-mermaid-\_r_haq\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ .rough-node .label text,#chatgpt-mermaid-\_r_haq\_ .node .label text,#chatgpt-mermaid-\_r_haq\_ .image-shape .label,#chatgpt-mermaid-\_r_haq\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_haq\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ .rough-node .label,#chatgpt-mermaid-\_r_haq\_ .node .label,#chatgpt-mermaid-\_r_haq\_ .image-shape .label,#chatgpt-mermaid-\_r_haq\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_haq\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_haq\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haq\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haq\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_haq\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_haq\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_haq\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_haq\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_haq\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_haq\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_haq\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_haq\_ .icon-shape,#chatgpt-mermaid-\_r_haq\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_haq\_ .icon-shape p,#chatgpt-mermaid-\_r_haq\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_haq\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_haq\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_haq\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_haq\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_haq\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_haq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_haq\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_haq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_haq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haq\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_haq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_haq\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_haq\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_haq\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_haq\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_haq\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ .node rect,#chatgpt-mermaid-\_r_haq\_ .node circle,#chatgpt-mermaid-\_r_haq\_ .node ellipse,#chatgpt-mermaid-\_r_haq\_ .node polygon,#chatgpt-mermaid-\_r_haq\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_haq\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_haq\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_haq\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_haq\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_haq\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Root Management GroupHR Management GroupEngineering ManagementGroupFinance Management GroupHR SubscriptionDev SubscriptionProduction SubscriptionFinance SubscriptionResource GroupsResource GroupsResource GroupsResource Groups

- A default root management group is created.
- Management groups can contain other management groups.
- A management group can contain multiple subscriptions.
- The hierarchy supports up to six levels of depth, excluding the root level.

## 8. Azure Management Hierarchy

This is one of the most important concepts from the session.

| Level | Component        | Purpose                           |
| ----- | ---------------- | --------------------------------- |
| 1     | Management Group | Organization-wide governance      |
| 2     | Subscription     | Billing and access control        |
| 3     | Resource Group   | Logical resource organization     |
| 4     | Resource         | Individual Azure service instance |

\#chatgpt-mermaid-\_r_hb1\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_hb1\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hb1\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hb1\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_hb1\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_hb1\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_hb1\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_hb1\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_hb1\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_hb1\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hb1\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hb1\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_hb1\_ p{margin:0;}#chatgpt-mermaid-\_r_hb1\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_hb1\_ .label text,#chatgpt-mermaid-\_r_hb1\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ .node rect,#chatgpt-mermaid-\_r_hb1\_ .node circle,#chatgpt-mermaid-\_r_hb1\_ .node ellipse,#chatgpt-mermaid-\_r_hb1\_ .node polygon,#chatgpt-mermaid-\_r_hb1\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ .rough-node .label text,#chatgpt-mermaid-\_r_hb1\_ .node .label text,#chatgpt-mermaid-\_r_hb1\_ .image-shape .label,#chatgpt-mermaid-\_r_hb1\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_hb1\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ .rough-node .label,#chatgpt-mermaid-\_r_hb1\_ .node .label,#chatgpt-mermaid-\_r_hb1\_ .image-shape .label,#chatgpt-mermaid-\_r_hb1\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_hb1\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_hb1\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hb1\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hb1\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_hb1\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hb1\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hb1\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hb1\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_hb1\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_hb1\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hb1\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_hb1\_ .icon-shape,#chatgpt-mermaid-\_r_hb1\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hb1\_ .icon-shape p,#chatgpt-mermaid-\_r_hb1\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_hb1\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_hb1\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hb1\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_hb1\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_hb1\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_hb1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_hb1\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_hb1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_hb1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hb1\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_hb1\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hb1\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hb1\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hb1\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_hb1\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_hb1\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ .node rect,#chatgpt-mermaid-\_r_hb1\_ .node circle,#chatgpt-mermaid-\_r_hb1\_ .node ellipse,#chatgpt-mermaid-\_r_hb1\_ .node polygon,#chatgpt-mermaid-\_r_hb1\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hb1\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_hb1\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_hb1\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_hb1\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hb1\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Management GroupSubscription 1Subscription 2Resource Group 1Resource Group 2Resource Group 3Virtual MachineStorage AccountLoad BalancerDatabase

Easy way to remember:

- Management Group = Manage the organization.
- Subscription = Manage billing and access.
- Resource Group = Manage related resources.
- Resource = Actual Azure service.

## 9. Azure Portal and Management Tools

Azure provides multiple ways to manage resources.

| Tool              | Description                                    |
| ----------------- | ---------------------------------------------- |
| Azure Portal      | Web-based graphical user interface (GUI)       |
| Azure Cloud Shell | Browser-based command-line environment         |
| Azure CLI         | Command-line interface                         |
| Azure PowerShell  | PowerShell-based management                    |
| Azure Mobile App  | Manage selected Azure operations from mobile   |
| REST API          | Programmatic access through HTTP APIs          |
| Azure SDK         | Integrate Azure services into application code |

### Azure Portal

- Create, update and delete resources.
- View available Azure services.
- Manage subscriptions and resource groups.
- Monitor resource operations and notifications.
- Access Azure Copilot for assistance.

### Azure Cloud Shell

- Runs directly inside the Azure Portal.
- Provides Azure CLI and Azure PowerShell.
- Does not require local installation.
- Supports file upload and download.
- Provides a browser-based code editor.

### Azure CLI vs Azure PowerShell

| Azure CLI                         | Azure PowerShell                    |
| --------------------------------- | ----------------------------------- |
| Command-line interface            | PowerShell module                   |
| Commonly preferred by Linux users | Commonly preferred by Windows users |
| Commands start with `az`          | Cmdlets use verb-noun naming        |
| Example: `az group list`          | Example: `Get-AzResourceGroup`      |

Both tools can perform the same general Azure management operations. Choose based on your comfort and scripting requirements.

## 10. Azure CLI – Practical Commands

Azure CLI commands follow a simple structure:

```
az <service> <action> <arguments>
```

For example:

```
az group list
```

Here:

- `az` – Azure CLI command.
- `group` – Service being managed.
- `list` – Action to perform.

### Important Resource Group commands

| Operation            | Azure CLI command                                  |
| -------------------- | -------------------------------------------------- |
| List resource groups | `az group list`                                    |
| List in table format | `az group list -o table`                           |
| Check if RG exists   | `az group exists -n demo-rg`                       |
| Create RG            | `az group create --name demo-rg --location eastus` |
| Delete RG            | `az group delete --name demo-rg`                   |

### Azure CLI authentication

When using Azure CLI locally:

```
az login
```

This authenticates your local machine with your Azure account.

To view subscriptions:

```
az account list -o table
```

To select a subscription:

```
az account set --subscription "<subscription-id>"
```

Important: Cloud Shell is automatically authenticated with the initial account. You generally don't need to execute `az login` again unless switching accounts.

### Azure PowerShell commands

| Operation             | PowerShell command       |
| --------------------- | ------------------------ |
| Login                 | `Connect-AzAccount`      |
| Get subscriptions     | `Get-AzSubscription`     |
| Create resource group | `New-AzResourceGroup`    |
| List resource groups  | `Get-AzResourceGroup`    |
| Delete resource group | `Remove-AzResourceGroup` |

PowerShell uses cmdlets (command-let), generally following a Verb-Noun naming convention.

Example:

```
New-AzResourceGroup -Name "demo-rg" -Location "EastUS"
```

```
Remove-AzResourceGroup -Name "demo-rg"
```

## 11. Azure CLI vs Portal – When to Use What?

Both Azure Portal and CLI can perform the same general resource management operations.

| Scenario                   | Preferred approach           |
| -------------------------- | ---------------------------- |
| Learning Azure             | Azure Portal                 |
| Creating one VM manually   | Azure Portal                 |
| Creating 100 VMs           | CLI / PowerShell automation  |
| Repeated resource creation | CLI scripts                  |
| Infrastructure automation  | Terraform or other IaC tools |

### Why is CLI useful?

Suppose you need to create 100 virtual machines.

Using the portal, you would need to repeat the creation process.

Using CLI, you can write a script with a `for` or `while` loop and automate the process.

\#chatgpt-mermaid-\_r_hcn\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_hcn\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hcn\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hcn\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_hcn\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_hcn\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_hcn\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_hcn\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_hcn\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_hcn\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcn\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcn\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_hcn\_ p{margin:0;}#chatgpt-mermaid-\_r_hcn\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_hcn\_ .label text,#chatgpt-mermaid-\_r_hcn\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ .node rect,#chatgpt-mermaid-\_r_hcn\_ .node circle,#chatgpt-mermaid-\_r_hcn\_ .node ellipse,#chatgpt-mermaid-\_r_hcn\_ .node polygon,#chatgpt-mermaid-\_r_hcn\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ .rough-node .label text,#chatgpt-mermaid-\_r_hcn\_ .node .label text,#chatgpt-mermaid-\_r_hcn\_ .image-shape .label,#chatgpt-mermaid-\_r_hcn\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_hcn\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ .rough-node .label,#chatgpt-mermaid-\_r_hcn\_ .node .label,#chatgpt-mermaid-\_r_hcn\_ .image-shape .label,#chatgpt-mermaid-\_r_hcn\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_hcn\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_hcn\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcn\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcn\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_hcn\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hcn\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hcn\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hcn\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_hcn\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_hcn\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcn\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_hcn\_ .icon-shape,#chatgpt-mermaid-\_r_hcn\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hcn\_ .icon-shape p,#chatgpt-mermaid-\_r_hcn\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_hcn\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_hcn\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hcn\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_hcn\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_hcn\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_hcn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_hcn\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_hcn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_hcn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcn\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_hcn\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcn\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hcn\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hcn\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_hcn\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_hcn\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ .node rect,#chatgpt-mermaid-\_r_hcn\_ .node circle,#chatgpt-mermaid-\_r_hcn\_ .node ellipse,#chatgpt-mermaid-\_r_hcn\_ .node polygon,#chatgpt-mermaid-\_r_hcn\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hcn\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_hcn\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_hcn\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_hcn\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcn\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Need to create 100 VMsChoose approachPortalCLI / PowerShellCreate VMs manuallyWrite automation scriptLoop through 100 VMsVMs created automatically

## 12. Region Selection – Important Factors

When choosing an Azure region, consider:

| Factor               | Explanation                                       |
| -------------------- | ------------------------------------------------- |
| User location        | Deploy close to end users to reduce latency       |
| Cost                 | Compare regional pricing                          |
| Compliance           | Follow data residency and regulatory requirements |
| Service availability | Verify that required services are available       |
| Team location        | Consider operational convenience                  |

Example: If your customers are in the US, deploying your application in a suitable US region may reduce latency.

However, compliance requirements can override geographical or cost preferences.

## 13. Moving Azure Resources Between Regions

Resources can sometimes be moved, but support depends on the service.

The instructor emphasized that moving resources between regions should be evaluated case by case.

Example:

Consider the following architecture:

\#chatgpt-mermaid-\_r_hcu\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_hcu\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hcu\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hcu\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_hcu\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_hcu\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_hcu\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_hcu\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_hcu\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_hcu\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcu\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcu\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_hcu\_ p{margin:0;}#chatgpt-mermaid-\_r_hcu\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_hcu\_ .label text,#chatgpt-mermaid-\_r_hcu\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ .node rect,#chatgpt-mermaid-\_r_hcu\_ .node circle,#chatgpt-mermaid-\_r_hcu\_ .node ellipse,#chatgpt-mermaid-\_r_hcu\_ .node polygon,#chatgpt-mermaid-\_r_hcu\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ .rough-node .label text,#chatgpt-mermaid-\_r_hcu\_ .node .label text,#chatgpt-mermaid-\_r_hcu\_ .image-shape .label,#chatgpt-mermaid-\_r_hcu\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_hcu\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ .rough-node .label,#chatgpt-mermaid-\_r_hcu\_ .node .label,#chatgpt-mermaid-\_r_hcu\_ .image-shape .label,#chatgpt-mermaid-\_r_hcu\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_hcu\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_hcu\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcu\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcu\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_hcu\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hcu\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hcu\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hcu\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_hcu\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_hcu\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hcu\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_hcu\_ .icon-shape,#chatgpt-mermaid-\_r_hcu\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hcu\_ .icon-shape p,#chatgpt-mermaid-\_r_hcu\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_hcu\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_hcu\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hcu\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_hcu\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_hcu\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_hcu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_hcu\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_hcu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_hcu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcu\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_hcu\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hcu\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hcu\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hcu\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_hcu\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_hcu\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ .node rect,#chatgpt-mermaid-\_r_hcu\_ .node circle,#chatgpt-mermaid-\_r_hcu\_ .node ellipse,#chatgpt-mermaid-\_r_hcu\_ .node polygon,#chatgpt-mermaid-\_r_hcu\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hcu\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_hcu\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_hcu\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_hcu\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hcu\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Load Balancer - East USVM 1 - East USVM 2 - East US

If you move VM 1 to West US without changing the architecture, it may affect the load balancer configuration.

Therefore:

- Check whether the particular Azure service supports regional movement.
- Consider dependencies between resources.
- Plan the migration before moving resources.
- Do not assume every Azure resource supports direct regional movement.

## 14. Permission vs Policy

These are two different concepts.

| Permission                           | Policy                                                                       |
| ------------------------------------ | ---------------------------------------------------------------------------- |
| Defines what a user is allowed to do | Defines what configurations or actions are permitted by organizational rules |
| Controls access                      | Enforces organizational compliance                                           |
| Example: User can create VMs         | Example: VMs can only be created in India                                    |

Example:

A user has permission to create virtual machines.

However, an organizational policy restricts VM creation to the India region.

The user cannot create a VM in Australia despite having the necessary permission.

## 15. Industry Best Practices – Resource Organization

The instructor discussed organizing subscriptions and resource groups based on applications, environments and business units.

### Example: Production and Non-Production separation

| Subscription                | Resource Groups |
| --------------------------- | --------------- |
| Production Subscription     | Production RG   |
| Non-Production Subscription | Dev RG          |
| Non-Production Subscription | Test RG         |
| Non-Production Subscription | Staging RG      |

### Large organization example

Consider a banking organization with different business units:

- Mobile Banking
- Net Banking
- Credit Cards
- Corporate Banking

Each business unit can have its own management group when different organizational policies are required.

Within each management group:

- Create separate subscriptions for applications.
- Separate production and non-production subscriptions.
- Organize resources into appropriate resource groups.

### Rules of thumb

| Requirement                             | Recommended approach                                                  |
| --------------------------------------- | --------------------------------------------------------------------- |
| Separate billing                        | Different subscriptions                                               |
| Apply different organizational policies | Different management groups                                           |
| Separate production and development     | Different subscriptions or resource groups, depending on requirements |
| Manage related resources together       | Same resource group                                                   |

These are organizational guidelines, not mandatory structures. Design the hierarchy according to business and operational requirements.

## 16. Important Azure vs AWS Comparisons

| Concept                        | AWS                                          | Azure              |
| ------------------------------ | -------------------------------------------- | ------------------ |
| Cloud account                  | AWS Account                                  | Azure Account      |
| Billing boundary               | AWS Account                                  | Azure Subscription |
| Logical resource grouping      | No direct equivalent to Azure Resource Group | Resource Group     |
| Organization-level grouping    | AWS Organizations                            | Management Groups  |
| Command-line tool              | AWS CLI                                      | Azure CLI          |
| PowerShell management          | AWS Tools for PowerShell                     | Azure PowerShell   |
| Dedicated private connectivity | Direct Connect                               | ExpressRoute       |

These are conceptual comparisons; the services do not have identical functionality.

## 17. Quick Revision – Interview Questions

| Question                                              | Key answer                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------------ |
| What is an Azure Region?                              | Geographical location containing one or more data centers          |
| What is an Availability Zone?                         | Physically separate location within a region for high availability |
| What is an Azure Resource?                            | An instance of an Azure service                                    |
| What is a Resource Group?                             | Logical container for related resources                            |
| Can one RG contain resources from multiple regions?   | Yes                                                                |
| Can a resource belong to multiple RGs simultaneously? | No                                                                 |
| Does an RG itself incur resource consumption charges? | No                                                                 |
| What happens when an RG is deleted?                   | Its contained resources are deleted                                |
| What is ARM?                                          | Azure's resource management layer                                  |
| What is an Azure Subscription?                        | Billing and access control boundary                                |
| Can one Azure account have multiple subscriptions?    | Yes                                                                |
| What is a Management Group?                           | Groups subscriptions for organization-wide governance              |
| What is Azure Cloud Shell?                            | Browser-based command-line environment                             |
| What is the difference between CLI and PowerShell?    | Different command interfaces for managing Azure                    |
| What is Azure Policy?                                 | Enforces organizational rules on resource configurations and usage |

## 18. Key Takeaways

Remember these 10 concepts

1. Azure, AWS and GCP share many cloud concepts.
2. Region represents geographical infrastructure.
3. Availability Zones provide protection against data center failures.
4. Every Azure service instance is a resource.
5. Resources belong to resource groups.
6. Subscriptions provide billing and access boundaries.
7. Management Groups organize multiple subscriptions.
8. ARM processes Azure resource management requests.
9. Azure Portal, CLI and PowerShell are different management interfaces.
10. Use separate subscriptions when billing or isolation requirements demand it.

Most important diagram to remember:

\#chatgpt-mermaid-\_r_hd6\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_hd6\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hd6\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_hd6\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_hd6\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_hd6\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_hd6\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_hd6\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_hd6\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_hd6\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hd6\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hd6\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_hd6\_ p{margin:0;}#chatgpt-mermaid-\_r_hd6\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_hd6\_ .label text,#chatgpt-mermaid-\_r_hd6\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ .node rect,#chatgpt-mermaid-\_r_hd6\_ .node circle,#chatgpt-mermaid-\_r_hd6\_ .node ellipse,#chatgpt-mermaid-\_r_hd6\_ .node polygon,#chatgpt-mermaid-\_r_hd6\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ .rough-node .label text,#chatgpt-mermaid-\_r_hd6\_ .node .label text,#chatgpt-mermaid-\_r_hd6\_ .image-shape .label,#chatgpt-mermaid-\_r_hd6\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_hd6\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ .rough-node .label,#chatgpt-mermaid-\_r_hd6\_ .node .label,#chatgpt-mermaid-\_r_hd6\_ .image-shape .label,#chatgpt-mermaid-\_r_hd6\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_hd6\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_hd6\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hd6\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hd6\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_hd6\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hd6\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hd6\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hd6\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_hd6\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_hd6\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_hd6\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_hd6\_ .icon-shape,#chatgpt-mermaid-\_r_hd6\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_hd6\_ .icon-shape p,#chatgpt-mermaid-\_r_hd6\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_hd6\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_hd6\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_hd6\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_hd6\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_hd6\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_hd6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_hd6\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_hd6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_hd6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hd6\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_hd6\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_hd6\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hd6\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_hd6\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_hd6\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_hd6\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ .node rect,#chatgpt-mermaid-\_r_hd6\_ .node circle,#chatgpt-mermaid-\_r_hd6\_ .node ellipse,#chatgpt-mermaid-\_r_hd6\_ .node polygon,#chatgpt-mermaid-\_r_hd6\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_hd6\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_hd6\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_hd6\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_hd6\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_hd6\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Management GroupAzure SubscriptionResource GroupVirtual MachineStorage AccountVirtual Network

End of Azure Morning Session – 2 November 2026.

Source: Uploaded class transcript. Classroom discussions about course scheduling and unrelated training queries have been excluded. Technical explanations and examples are based on the instructor's discussion, with obvious speech-to-text errors cleaned up.&#x20;

Azure(Morning) transcript_2026-11-02_09.24.16.txt

Azure(Morning) transcript_2026-11-02_09.24.16.txt
