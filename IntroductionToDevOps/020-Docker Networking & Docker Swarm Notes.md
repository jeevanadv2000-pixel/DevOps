# DevOps Session — Docker Networking & Docker Swarm Notes

Date: 29 September 2026\
Instructor: Rochak Agrawal\
Focus: Docker network drivers, container-to-container communication, custom bridge networks, IP addressing, network isolation, Docker Swarm, and container orchestration.

> Note: These notes retain the instructor's technical explanations and demonstrations, remove participant discussions, and correct a few technical oversimplifications in the transcript. The session mainly covers Docker networking and introduces Docker Swarm. [Source transcript](https://raw.githubusercontent.com/jeevanadv2000-pixel/python/main/DevOps%20Session%28Morning%29%20transcript_2026-09-29_09.21.30.txt?utm_source=chatgpt.com).

## 1. Why Docker Networking?

Previously, we learned how to expose a container application through host port mapping.

Example:

```
docker run -d --name nginx -p 8080:80 nginx
```

Traffic flow:

```
User / Browser
      |
      | http://HOST-IP:8080
      ▼
Docker Host
      |
      | Port mapping
      ▼
Container:80
      |
      ▼
NGINX
```

But what happens when multiple containers need to communicate?

Consider a typical application:

- Web container
- Application container
- Database container

The application needs to communicate with the database. The web container needs to communicate with the application.

Docker networking provides the connectivity and isolation required for this architecture.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



### Three-tier container architecture

\#chatgpt-mermaid-\_r_32c\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_32c\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_32c\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_32c\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_32c\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_32c\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_32c\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_32c\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_32c\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_32c\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_32c\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_32c\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_32c\_ p{margin:0;}#chatgpt-mermaid-\_r_32c\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_32c\_ .label text,#chatgpt-mermaid-\_r_32c\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ .node rect,#chatgpt-mermaid-\_r_32c\_ .node circle,#chatgpt-mermaid-\_r_32c\_ .node ellipse,#chatgpt-mermaid-\_r_32c\_ .node polygon,#chatgpt-mermaid-\_r_32c\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ .rough-node .label text,#chatgpt-mermaid-\_r_32c\_ .node .label text,#chatgpt-mermaid-\_r_32c\_ .image-shape .label,#chatgpt-mermaid-\_r_32c\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_32c\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ .rough-node .label,#chatgpt-mermaid-\_r_32c\_ .node .label,#chatgpt-mermaid-\_r_32c\_ .image-shape .label,#chatgpt-mermaid-\_r_32c\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_32c\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_32c\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_32c\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_32c\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_32c\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_32c\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_32c\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_32c\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_32c\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_32c\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_32c\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_32c\_ .icon-shape,#chatgpt-mermaid-\_r_32c\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_32c\_ .icon-shape p,#chatgpt-mermaid-\_r_32c\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_32c\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_32c\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_32c\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_32c\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_32c\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_32c\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_32c\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_32c\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_32c\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_32c\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_32c\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_32c\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_32c\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_32c\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_32c\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_32c\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ .node rect,#chatgpt-mermaid-\_r_32c\_ .node circle,#chatgpt-mermaid-\_r_32c\_ .node ellipse,#chatgpt-mermaid-\_r_32c\_ .node polygon,#chatgpt-mermaid-\_r_32c\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_32c\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_32c\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_32c\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_32c\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_32c\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}UserWeb ContainerApplication ContainerDatabase Container

## 2. Network Namespace

Every container gets its own network namespace.

A network namespace provides an isolated network environment with its own:

- Network interfaces
- IP addresses
- Routing information
- Network configuration

Docker uses network drivers to connect container network namespaces.

```
Docker Host
    |
    ├── Container A
    │     ├── Network interface
    │     └── Private IP
    │
    ├── Container B
    │     ├── Network interface
    │     └── Private IP
    │
    └── Container C
          ├── Network interface
          └── Private IP
```

Containers can communicate when their network configuration permits it.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



## 3. Docker Network Types

Docker provides several network drivers.

| Network driver | Purpose                                                      | Typical use                                     |
| -------------- | ------------------------------------------------------------ | ----------------------------------------------- |
| Bridge         | Connects containers on one Docker host                       | Single-host applications                        |
| Host           | Uses the host's network stack                                | Specialized performance/networking requirements |
| None           | Disables normal container networking                         | Isolated workloads                              |
| Overlay        | Connects containers across multiple Docker hosts             | Docker Swarm                                    |
| Macvlan        | Gives containers their own MAC address on a physical network | Legacy applications or direct LAN connectivity  |

These are the five networking types discussed by the instructor.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



# 4. Bridge Network

Most important

Bridge is the most commonly used Docker network driver for single-host container communication.

It creates a private network on a Docker host.

```
             Docker Host
                  |
           Docker Bridge
                  |
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Web         App        DB
   Container  Container  Container
```

### Important characteristics

- Containers receive private IP addresses.
- Containers on the same bridge network can communicate.
- Containers can access external networks if the host's networking permits it.
- Different bridge networks provide network isolation.
- Port mapping can publish container services to the host.

### Default bridge vs user-defined bridge

This is a particularly important distinction.

Docker automatically creates a default bridge network named `bridge`.

When you run:

```
docker run -d --name web nginx
```

Docker connects the container to the default bridge unless you specify another network.

Default bridge network:

- Containers can generally communicate with each other using their IP addresses.
- Automatic container-name resolution is not provided in the same way as user-defined bridge networks.
- IP addresses can change when containers are recreated.

User-defined bridge network:

- Containers can communicate using container names.
- Docker provides automatic DNS-based name resolution.
- You can create separate networks to isolate application components.

This is why user-defined bridge networks are generally preferred over the default bridge for multi-container applications.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+2



### Default bridge vs user-defined bridge

| Feature                              | Default bridge  | User-defined bridge      |
| ------------------------------------ | --------------- | ------------------------ |
| Created automatically                | Yes             | No                       |
| Container communication              | By IP           | By IP and container name |
| Automatic name resolution            | No              | Yes                      |
| Network isolation                    | Default network | Separate custom networks |
| Recommended for multi-container apps | Less preferred  | Yes                      |

## 5. Creating a User-Defined Bridge Network

Create a custom network:

```
docker network create \
  --driver bridge \
  my-net
```

Here:

- `docker network create` creates a network.
- `--driver bridge` specifies the network type.
- `my-net` is the network name.

List networks:

```
docker network ls
```

Inspect a network:

```
docker network inspect my-net
```

### Run containers in the custom network

```
docker run -dit \
  --name app1 \
  --network my-net \
  busybox
```

Create another container:

```
docker run -dit \
  --name app2 \
  --network my-net \
  busybox
```

Test connectivity:

```
docker exec app1 ping -c 3 app2
```

Because both containers are connected to the same user-defined bridge, Docker can resolve `app2` to its container IP.

```
             my-net
                |
          ┌─────┴─────┐
          ▼           ▼
        app1         app2
          |           |
          └── Name ───┘
             resolution
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

## 6. IP Address Assignment and Subnets

When Docker creates a bridge network, it allocates a private IP address range.

Example:

```
Network: 192.0.2.0/24

Container 1: 192.0.2.2
Container 2: 192.0.2.3
Container 3: 192.0.2.4
```

The addresses above are illustrative; `192.0.2.0/24` is a documentation range, not a suitable general-purpose production subnet.

### Specify a custom subnet

```
docker network create \
  --driver bridge \
  --subnet 172.30.0.0/24 \
  custom-net
```

Docker assigns addresses from the configured subnet.

Inspect:

```
docker network inspect custom-net
```

You can check:

- Subnet
- Gateway
- Connected containers
- Container IP addresses

If no subnet is specified, Docker selects an available range from its configured/default address pools.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



### Important: Docker networking vs AWS networking

Docker bridge networking is an internal Docker networking mechanism.

For example:

```
AWS VPC
   |
   ▼
EC2 Instance (Docker Host)
   |
   ▼
Docker Bridge Network
   |
   ├── Container A
   └── Container B
```

The EC2 instance participates in AWS VPC networking. Docker creates another network layer for its containers.

Therefore, Docker bridge networking works similarly whether Docker runs on:

- AWS EC2
- Azure VM
- Google Cloud VM
- On-premises Linux server

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

# 7. Host Network

The host network driver allows a container to use the host's network stack directly.

Example:

```
docker run -d \
  --name nginx-host \
  --network host \
  nginx
```

### Architecture

```
          Docker Host
          Host Network
                |
          ┌─────┴─────┐
          ▼           ▼
       Host App    Container
                   Uses host
                   network stack
```

### Characteristics

- No separate container network namespace in the usual host-network implementation.
- No Docker port mapping is required.
- Container uses the host's network interfaces and ports.
- Port conflicts can occur with other host processes.
- Network isolation is reduced.

### Bridge vs Host

| Feature                    | Bridge                               | Host                                    |
| -------------------------- | ------------------------------------ | --------------------------------------- |
| Separate network namespace | Yes                                  | No, shares host networking              |
| Separate container IP      | Yes                                  | Uses host networking                    |
| Port mapping               | Usually required for external access | Not required                            |
| Network isolation          | More                                 | Less                                    |
| Performance overhead       | Some networking overhead             | Lower networking overhead in some cases |

Interview point: Host networking can improve performance in certain workloads, but it sacrifices network isolation. It is not automatically the best choice for production.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

# 8. None Network

The `none` network driver disables normal external networking for the container.

Example:

```
docker run -dit \
  --name isolated-app \
  --network none \
  busybox
```

The container has only its loopback interface for local communication.

```
Container
    |
    ▼
Loopback (127.0.0.1)

No normal external network connectivity
```

### When is it useful?

- Isolated background processing
- Sensitive processing workloads
- Tasks that do not need network access
- Security-focused execution environments

Example: A batch process calculating employee salaries and generating a report without needing network connectivity.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

# 9. Overlay Network

Important for Swarm

A bridge network works within a single Docker host.

But what happens when containers run on different hosts?

Example:

```
Docker Host 1                 Docker Host 2
     |                             |
  Web Container               DB Container
     |                             |
     └────────────── ? ────────────┘
```

A normal local bridge network cannot directly span both hosts.

This is where the overlay network comes in.

An overlay network connects containers across multiple Docker hosts.

```
          Overlay Network
        /                 \
       ▼                   ▼
 Docker Host 1         Docker Host 2
       |                   |
 Web Container        DB Container
```

### Characteristics

- Supports multi-host container communication.
- Used with Docker Swarm.
- Provides a virtual network across hosts.
- Helps connect services running on different nodes.

### When to use it?

When an application is distributed across multiple Docker hosts and its containers need network communication.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

# 10. Macvlan Network

Macvlan allows a container to appear as a separate device on the physical network.

Normally, containers communicate through virtual networking and the host's network interfaces.

With Macvlan, each container can be assigned its own MAC address.

```
          Physical Network
                 |
          Docker Host NIC
                 |
           Macvlan Network
             /       \
            ▼         ▼
       Container A  Container B
        MAC A        MAC B
        IP A         IP B
```

### When is it useful?

- Legacy applications that depend on MAC addresses.
- Applications requiring direct LAN connectivity.
- Network environments where containers must appear as separate physical devices.

Important: Macvlan requires suitable host network-interface configuration and support from the underlying network.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



# 11. Network Driver Comparison

| Driver  | Scope          | Container IP       | Main purpose                         |
| ------- | -------------- | ------------------ | ------------------------------------ |
| Bridge  | Single host    | Private IP         | Local container communication        |
| Host    | Host network   | Host networking    | Direct host network access           |
| None    | Isolated       | Loopback only      | No external networking               |
| Overlay | Multiple hosts | Overlay network IP | Multi-host communication             |
| Macvlan | Physical LAN   | LAN IP             | Containers appear as network devices |

### Easy way to remember

```
Bridge   → One Docker host
Host     → Host network directly
None     → No external network
Overlay  → Multiple Docker hosts
Macvlan  → Separate MAC address
```

# 12. Container-to-Container Communication

Consider three containers:

- A = Web
- B = Application
- C = Database

Suppose the requirement is:

- A should communicate with B.
- B should communicate with C.
- A should NOT communicate directly with C.

We can achieve this using two separate user-defined bridge networks.

\#chatgpt-mermaid-\_r_37a\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(13, 13, 13);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_37a\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_37a\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_37a\_ .error-icon{fill:rgb(243, 243, 243);}#chatgpt-mermaid-\_r_37a\_ .error-text{fill:rgb(13, 13, 13);stroke:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_37a\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_37a\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_37a\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_37a\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_37a\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_37a\_ .marker.cross{stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_37a\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_37a\_ p{margin:0;}#chatgpt-mermaid-\_r_37a\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ .cluster-label text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ .cluster-label span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_37a\_ .label text,#chatgpt-mermaid-\_r_37a\_ span{fill:rgb(13, 13, 13);color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ .node rect,#chatgpt-mermaid-\_r_37a\_ .node circle,#chatgpt-mermaid-\_r_37a\_ .node ellipse,#chatgpt-mermaid-\_r_37a\_ .node polygon,#chatgpt-mermaid-\_r_37a\_ .node path{fill:rgb(222, 234, 251);stroke:rgb(83, 154, 248);stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ .rough-node .label text,#chatgpt-mermaid-\_r_37a\_ .node .label text,#chatgpt-mermaid-\_r_37a\_ .image-shape .label,#chatgpt-mermaid-\_r_37a\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_37a\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ .rough-node .label,#chatgpt-mermaid-\_r_37a\_ .node .label,#chatgpt-mermaid-\_r_37a\_ .image-shape .label,#chatgpt-mermaid-\_r_37a\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_37a\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_37a\_ .root .anchor path{fill:rgb(143, 143, 143)!important;stroke-width:0;stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_37a\_ .arrowheadPath{fill:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_37a\_ .edgePath .path{stroke:rgb(143, 143, 143);stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ .flowchart-link{stroke:rgb(143, 143, 143);fill:none;}#chatgpt-mermaid-\_r_37a\_ .edgeLabel{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_37a\_ .edgeLabel p{background-color:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_37a\_ .edgeLabel rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_37a\_ .labelBkg{background-color:rgba(252, 252, 252, 0.5);}#chatgpt-mermaid-\_r_37a\_ .cluster rect{fill:rgb(243, 243, 243);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ .cluster text{fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ .cluster span{color:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(243, 243, 243);border:1px solid rgba(0, 0, 0, 0.1);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_37a\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(13, 13, 13);}#chatgpt-mermaid-\_r_37a\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_37a\_ .icon-shape,#chatgpt-mermaid-\_r_37a\_ .image-shape{background-color:rgb(252, 252, 252);text-align:center;}#chatgpt-mermaid-\_r_37a\_ .icon-shape p,#chatgpt-mermaid-\_r_37a\_ .image-shape p{background-color:rgb(252, 252, 252);padding:2px;}#chatgpt-mermaid-\_r_37a\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_37a\_ .image-shape .label rect{opacity:0.5;background-color:rgb(252, 252, 252);fill:rgb(252, 252, 252);}#chatgpt-mermaid-\_r_37a\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_37a\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_37a\_ .node .neo-node{stroke:rgb(83, 154, 248);}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_37a\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_37a\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].node .neo-line path{stroke:rgb(83, 154, 248);filter:none;}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_37a\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_37a\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_37a\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_37a\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_37a\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_37a\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(0, 79, 153);}#chatgpt-mermaid-\_r_37a\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_37a\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_37a\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ .node rect,#chatgpt-mermaid-\_r_37a\_ .node circle,#chatgpt-mermaid-\_r_37a\_ .node ellipse,#chatgpt-mermaid-\_r_37a\_ .node polygon,#chatgpt-mermaid-\_r_37a\_ .node path{fill:rgb(229, 243, 255);stroke:rgba(0, 0, 0, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_37a\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_37a\_ .mermaid-decision .label-container{fill:rgb(245, 250, 255);stroke:rgb(206, 219, 229);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_37a\_ .edgePaths .flowchart-link{stroke:rgb(143, 143, 143);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_37a\_ .marker{fill:rgb(143, 143, 143);stroke:rgb(143, 143, 143);}#chatgpt-mermaid-\_r_37a\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Backend NetworkFrontend NetworkDatabase Container CWeb Container AApplication Container B

Container B is attached to both networks.

- A can communicate with B.
- B can communicate with C.
- A cannot directly communicate with C through these networks.

This is a useful way to implement network segmentation.

Practical relevance: You can isolate database access so that only the application layer can reach the database, rather than exposing it to every container.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

# 13. Connecting One Container to Multiple Networks

A container can belong to more than one network.

Create networks:

```
docker network create frontend
docker network create backend
```

Run a container in the frontend network:

```
docker run -dit \
  --name app \
  --network frontend \
  busybox
```

Connect it to another network:

```
docker network connect backend app
```

Inspect:

```
docker inspect app
```

The container now has network connectivity through both networks.

```
          frontend
              |
              |
          App Container
              |
              |
           backend
```

It can have a separate IP address on each network.

The operating system's routing table determines how traffic is sent through the available interfaces.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub



# 14. Container IP vs Container Name

The instructor explains how applications communicate using a hostname and port.

For example, a Java application connecting to MySQL needs:

```
Database hostname
Database port
Username
Password
```

In a user-defined Docker bridge network, you can use the database container name as the hostname.

Example:

```
DB_HOST=mysql-db
DB_PORT=3306
```

Application configuration:

```
docker run -d \
  --name app \
  --network backend \
  -e DB_HOST=mysql-db \
  my-app:1.0
```

Database:

```
docker run -d \
  --name mysql-db \
  --network backend \
  mysql
```

The application can resolve `mysql-db` through Docker's embedded DNS service.

### Why use names instead of IP addresses?

Container IP addresses can change when containers are recreated.

But if the container name/service name remains consistent, applications can continue using that name.

```
Not ideal:
Application → 172.30.0.3

Better:
Application → mysql-db
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

# 15. Static IP Address for Containers

Docker allows you to specify a static IP address on a user-defined network.

Example:

```
docker network create \
  --subnet 172.30.0.0/24 \
  custom-net
```

Run a container:

```
docker run -dit \
  --name app \
  --network custom-net \
  --ip 172.30.0.10 \
  busybox
```

This assigns the container the specified IP address on that network, provided the address is available and valid.

### But should we depend on static container IPs?

Usually, no.

Containers are designed to be replaceable. When a container is recreated, its identity and IP may change unless explicitly managed.

For most application communication, use:

- Container names
- Service names
- Service discovery

rather than hardcoding container IP addresses.

This becomes especially important when we move to orchestration platforms such as Kubernetes.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

# 16. Stateless vs Stateful Applications

The instructor introduces an important distinction while explaining container IP changes.

| Stateless application                                           | Stateful application                            |
| --------------------------------------------------------------- | ----------------------------------------------- |
| Does not rely on local persistent session/data between requests | Maintains or depends on persistent state        |
| Instances can often be replaced easily                          | Data/state must be preserved                    |
| Easier to scale horizontally                                    | Requires careful storage and consistency design |
| Example: stateless web frontend                                 | Example: database                               |

### Example: Stateless

A user opens a news website, reads an article, and leaves.

Another instance can serve the next request without needing the original container's local state.

### Example: Stateful

A user adds items to an online shopping cart.

The cart must remain available across requests and container replacements.

The state should be stored in a suitable persistent system, such as a database or shared storage, rather than relying only on a container's writable layer.

```
Stateless:
Request → Container A
Next request → Container B

Stateful:
Request → Application
              |
              ▼
        Persistent Database
```

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

# 17. Docker Networking Commands

These are the commands worth knowing from this session.

| Command                      | Purpose                                    |
| ---------------------------- | ------------------------------------------ |
| `docker network ls`          | List networks                              |
| `docker network create`      | Create network                             |
| `docker network inspect`     | Show network details                       |
| `docker network connect`     | Connect container to network               |
| `docker network disconnect`  | Disconnect container                       |
| `docker network rm`          | Remove network                             |
| `docker network prune`       | Remove unused networks                     |
| `docker inspect <container>` | Inspect container networking configuration |

### Example workflow

```
# List networks
docker network ls

# Create a bridge network
docker network create --driver bridge my-net

# Run container in the network
docker run -dit --name app1 --network my-net busybox

# Run another container
docker run -dit --name app2 --network my-net busybox

# Test communication
docker exec app1 ping -c 3 app2

# Inspect network
docker network inspect my-net

# Connect existing container to another network
docker network connect backend app1
```

# 18. Docker Limitations and Container Orchestration

The instructor then moves from Docker networking to container orchestration.

Docker is useful for:

- Packaging applications
- Building images
- Running containers
- Managing container networks and volumes

But production environments often require additional capabilities:

- Multiple hosts
- Automatic recovery
- Scaling
- Load balancing
- Service discovery
- Coordinated deployments

This is where container orchestration becomes important.

```
             Docker
               |
        Runs Containers
               |
               ▼
       Orchestration Layer
               |
       ┌───────┴────────┐
       ▼                ▼
  Docker Swarm      Kubernetes
```

A technical clarification: Docker Engine can run production workloads. Orchestration is needed when you want to manage applications across multiple containers and hosts reliably; it is not correct to say Docker itself cannot be used in production.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1



# 19. Docker Swarm

Docker Swarm is Docker's built-in container orchestration solution.

It allows multiple Docker hosts to be joined into a cluster.

```
             Docker Swarm Cluster
                     |
             ┌───────┴────────┐
             ▼               ▼
         Manager           Worker
             |               |
          Node 1           Node 2
             |               |
        Containers       Containers
```

### Features

- Multi-host container management
- Service scheduling
- Load balancing
- Scaling services
- Automatic recovery/rescheduling in supported failure scenarios
- Overlay networking

### Manager node

The manager is responsible for cluster management and scheduling decisions.

### Worker node

Worker nodes run tasks assigned by the manager.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

## 20. Initialize Docker Swarm

On the first Docker host:

```
docker swarm init
```

This initializes the Swarm and makes the current node a manager.

The output includes a command with a join token for adding worker nodes.

### List nodes

```
docker node ls
```

Initially:

```
ID       HOSTNAME    STATUS    AVAILABILITY    MANAGER STATUS
abc123   manager1    Ready     Active          Leader
```

### Add worker node

Run the generated `docker swarm join` command on another Docker host.

It will look similar to:

```
docker swarm join \
  --token <worker-token> \
  <manager-ip>:2377
```

The token and manager address must come from your own Swarm initialization output.

Then, on the manager:

```
docker node ls
```

You should see the worker node.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub

### Leave Swarm

```
docker swarm leave
```

A worker can leave the cluster using this command. A manager should be demoted or handled appropriately before leaving an active production cluster.

# 21. Docker Swarm vs Kubernetes

The instructor compares Docker Swarm with Kubernetes.

| Feature               | Docker Swarm                      | Kubernetes                        |
| --------------------- | --------------------------------- | --------------------------------- |
| Purpose               | Container orchestration           | Container orchestration           |
| Complexity            | Relatively simple                 | More complex                      |
| Cluster management    | Built into Docker                 | Separate Kubernetes control plane |
| Scaling               | Supported                         | Advanced scaling capabilities     |
| Networking            | Overlay networking                | Kubernetes networking model       |
| Ecosystem             | Smaller                           | Large ecosystem                   |
| Common cloud offering | Docker Swarm can run on cloud VMs | EKS, AKS, GKE                     |

### Cloud-managed Kubernetes

Cloud providers offer managed Kubernetes services:

| Cloud provider | Service    |
| -------------- | ---------- |
| AWS            | Amazon EKS |
| Azure          | Azure AKS  |
| Google Cloud   | Google GKE |

Managed Kubernetes reduces the operational burden of managing the Kubernetes control plane.

[image](https://www.google.com/s2/favicons?domain=https://raw.githubusercontent.com\&sz=32)

GitHub+1

### Important technical correction

The instructor mentions that Kubernetes runs Docker containers. More precisely:

- Kubernetes runs OCI-compatible container images.
- Modern Kubernetes commonly uses container runtimes such as `containerd` or CRI-O.
- Docker Engine is not required on every Kubernetes worker node.
- Kubernetes provides its own networking and storage abstractions.

So, Docker and Kubernetes are related technologies, but they are not the same product.

# 22. Practical DevOps Scenario — Jenkins and Tomcat Containers

This example connects the networking session to your earlier Jenkins assignment.

Suppose Jenkins and Tomcat are running as containers.

```
          Docker Host
                |
          custom network
                |
       ┌────────┴─────────┐
       ▼                  ▼
    Jenkins             Tomcat
    Container           Container
       |                  |
       └────── Deploy ────┘
```

Create a network:

```
docker network create cicd-net
```

Run Jenkins:

```
docker run -d \
  --name jenkins \
  --network cicd-net \
  -p 8080:8080 \
  jenkins/jenkins:lts
```

Run Tomcat:

```
docker run -d \
  --name tomcat \
  --network cicd-net \
  -p 8081:8080 \
  tomcat
```

Jenkins can reach Tomcat using:

```
tomcat:8080
```

The browser can reach Jenkins through:

```
http://HOST-IP:8080
```

The browser can reach Tomcat through:

```
http://HOST-IP:8081
```

Key learning: Container-to-container communication uses the Docker network. Host-to-container access is provided through published ports.

# 🔥 Interview Revision

## Important questions

1\. What is Docker networking?

Docker networking allows containers to communicate with other containers, the Docker host, and external networks.

2\. What is a network namespace?

An isolated network environment for a container, including its network interfaces, IP addresses, and routing configuration.

3\. What is a bridge network?

A private network that connects containers running on a single Docker host.

4\. Difference between default bridge and user-defined bridge?

The user-defined bridge provides automatic container-name resolution and better network separation.

5\. What is host networking?

A container uses the host's network stack directly rather than an isolated container network stack.

6\. What is the none network?

It disables normal external networking for a container.

7\. What is an overlay network?

A network that connects containers across multiple Docker hosts.

8\. What is Macvlan?

A network driver that gives containers their own MAC addresses and allows them to appear as separate devices on a physical network.

9\. How do containers communicate using names?

Through Docker's embedded DNS service on a user-defined network.

10\. Why should we avoid hardcoding container IP addresses?

Container IPs can change when containers are recreated. Names and service discovery provide more stable references.

11\. Can one container belong to multiple networks?

Yes. Use:

```
docker network connect <network> <container>
```

12\. What is Docker Swarm?

A container orchestration solution that connects multiple Docker hosts into a cluster.

13\. What is the difference between manager and worker nodes?

Manager nodes manage the cluster and scheduling; worker nodes execute assigned workloads.

14\. What is the difference between Docker Swarm and Kubernetes?

Swarm is simpler and integrated with Docker, while Kubernetes offers a broader, more powerful orchestration platform and ecosystem.

15\. What is a managed Kubernetes service?

A cloud provider manages much of the Kubernetes control-plane infrastructure so teams can focus more on workloads.

# 🧠 Final Mental Model

Docker networking

```
Docker Host
    |
    ├── Bridge
    │     └── Single-host containers
    │
    ├── Host
    │     └── Host network stack
    │
    ├── None
    │     └── Isolated container
    │
    ├── Overlay
    │     └── Multi-host containers
    │
    └── Macvlan
          └── Separate MAC address
```

Container communication

```
Default bridge → IP-based communication
Custom bridge  → IP + container-name resolution
Multiple nets  → Network segmentation
```

Container orchestration

```
Docker Engine
     |
     ▼
Docker Swarm
     |
     ▼
Multi-host cluster
     |
     ▼
Kubernetes
     |
     ▼
Advanced orchestration
```

## What to prioritize for interviews

1. Bridge vs host vs none vs overlay vs Macvlan.
2. Default bridge vs user-defined bridge.
3. Container-to-container communication using names.
4. Docker subnet and IP allocation.
5. Connecting a container to multiple networks.
6. Network isolation using multiple bridge networks.
7. Stateless vs stateful applications.
8. Docker Swarm manager and worker.
9. Docker Swarm vs Kubernetes.
10. Jenkins-to-Tomcat communication through a custom Docker network.

Next learning topic: Continue with Docker Swarm services, replicas, scaling, and orchestration, then move into Kubernetes.