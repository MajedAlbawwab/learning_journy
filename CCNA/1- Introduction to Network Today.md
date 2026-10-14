# Module 1: Networking Today

_Introduction to Networks v7.0 (ITN)_

---

## 📌 Overview

- Module 1 introduces what networks are made of (hosts, intermediary devices, media), how they are drawn (topology diagrams), and how they are classified (LAN, WAN, internet, intranet, extranet).
- It also covers how networks connect to the internet, the four requirements of a reliable network, current trends (BYOD, cloud, etc.), and basic network security.
- It is the foundation for everything later in CCNA: terminology, device roles, and the vocabulary used in all topology diagrams.

### Module Objective

**Explain the advances in modern technologies.**

|Topic|Topic Objective|
|---|---|
|Networks Affect Our Lives|Explain how networks affect our daily lives.|
|Network Components|Explain how host and network devices are used.|
|Network Representations and Topologies|Explain network representations and how they are used in network topologies.|
|Common Types of Networks|Compare the characteristics of common types of networks.|
|Internet Connections|Explain how LANs and WANs interconnect to the internet.|
|Reliable Networks|Describe the four basic requirements of a reliable network.|
|Network Trends|Explain how trends such as BYOD, online collaboration, video, and cloud computing are changing the way we interact.|
|Network Security|Identify some basic security threats and solutions for all networks.|
|The IT Professional|Explain employment opportunities in the networking field.|

> ⚠️ **Check:** The objectives table lists a topic "The IT Professional", but the PDF contains **no slides for it** (numbering jumps from 1.8 Network Security to 1.10 Module Practice and Quiz; 1.9 is missing). Nothing to note for that topic from this PDF.

---

## 📖 Core Concepts

## 1.1 Networks Affect Our Lives

- The PDF contains only a promotional video ("The Cisco Networking Academy Learning Experience": _learn how we use technology to make the world a better place_).
- No technical content.

---

## 1.2 Network Components

### Host Roles

- **Host (end device)**: every computer on a network.
- **Server**: a computer that **provides information** to end devices (e.g., email servers, web servers, file servers).
- **Client**: a computer that **sends requests** to servers to retrieve information (e.g., a web page from a web server, email from an email server).

```
 Client                                          Server
 [PC] ------------ ( Internet ) ------------- [Server]
```

|Server Type|Description|
|---|---|
|Email|Email server runs email server software. Clients use client software to access email.|
|Web|Web server runs web server software. Clients use browser software to access web pages.|
|File|File server stores corporate and user files. The client devices access these files.|

### Peer-to-Peer (P2P)

- A device can be **both a client and a server** in a **Peer-to-Peer network**.
- Recommended **only for very small networks**.
- Example in the figure: one PC shares a printer ("Print Sharing"), another shares files ("File Sharing").

```
 [Printer]----------[PC: "I have a printer to share"]----------[PC: "I have files to share"]
                         (Print Sharing)                            (File Sharing)
```

|Advantages|Disadvantages|
|---|---|
|Easy to set up|No centralized administration|
|Less complex|Not as secure|
|Lower cost|Not scalable|
|Used for simple tasks: transferring files and sharing printers|Slower performance|

> 🎯 **Exam Focus:** Client vs. server roles, and P2P pros/cons (cheap/simple vs. no central admin, not secure, not scalable, slower).

### End Devices

- **End device**: where a message **originates from** or where it is **received**.
- Data originates at an end device, flows through the network, and arrives at an end device.
- Figure: two LANs (each with PCs, a server, an IP phone, and a switch) connected through an **internetwork** of routers. A message (envelope with "?") leaving the left LAN can take **alternate routes** through the routers to reach the right LAN.

```
  LAN                      Internetwork                     LAN
 [PC][Server][IP Phone]   +----------------+   [PC][Server][IP Phone]
      \   |   /           |   (R)----(R)   |         \   |   /
       [Switch]--(msg ?)--|    |  \  /     |--(R)----[Switch]
                          |   (R)----      |
                          +----------------+
        Messages can take alternate routes.
```

### Intermediary Network Devices

- **Intermediary device**: interconnects end devices.
- Examples: **switches, wireless access points, routers, firewalls**.
- Figure shows: Wireless Router, LAN Switch, Router, Multilayer Switch, Firewall Appliance.
- Roles in managing data as it flows through the network:
    - **Regenerate and retransmit** data signals.
    - **Maintain information** about what pathways exist in the network.
    - **Notify other devices** of errors and communication failures.

### Network Media

- Communication across a network is carried through a **medium**, which allows a message to travel from source to destination.

|Media Type|Description|
|---|---|
|**Metal wires within cables** (copper)|Uses **electrical impulses**|
|**Glass or plastic fibers within cables** (fiber-optic cable)|Uses **pulses of light**|
|**Wireless transmission**|Uses **modulation of specific frequencies of electromagnetic waves**|

> 🧠 **Memory Aid:** Copper = **E**lectricity, Fiber = **L**ight, Wireless = **W**aves (electromagnetic).

---

## 1.3 Network Representations and Topologies

### Network Representations

- **Network diagrams** (often called **topology diagrams**) use **symbols** to represent devices within the network.
- Important terms:
    - **Network Interface Card (NIC)**
    - **Physical Port**
    - **Interface**
- > **Note:** The terms _port_ and _interface_ are often used **interchangeably**.
    

**Symbols (from the figure):**

|Category|Symbols shown|
|---|---|
|End Devices|Desktop Computer, Laptop, Printer, IP Phone, Wireless Tablet, TelePresence Endpoint|
|Intermediary Devices|Wireless Router, LAN Switch, Router, Multilayer Switch, Firewall Appliance|
|Network Media|**Wireless Media** = wavy line; **LAN Media** = straight black line; **WAN Media** = red zig-zag (lightning-bolt) line|

> 🎯 **Exam Focus:** Know the media line styles: wavy = wireless, straight line = LAN, red zig-zag = WAN.

### Topology Diagrams

- **Physical topology diagrams**: show the **physical location** of intermediary devices and **cable installation**.
- **Logical topology diagrams**: show **devices, ports, and the addressing scheme** of the network.

**Physical topology (figure):**

```
                              (Internet)
                                  |
 Server Room: Rm 2158             |                      Rm 2124
 +--------------------------+     |        +--------------------------------------+
 | Web Server  (Rack2/Sh1)--+     |        | S3 (Rack1/Shelf1) ---- Class 1: Rm 2125 (laptop)
 | Email Server(Rack2/Sh2)--S1----R1 ------ S4 (Rack1/Shelf2) ---- Class 2: Rm 2126 (laptop)
 | File Server (Rack2/Sh3)--+ (Rack1/Shelf2) (Rack1/Shelf1)
 +--------------------------+     |        | S5 (Rack1/Shelf3) ---- Class 3: Rm 2127 (laptop)
                                  |        +--------------------------------------+
                                  S2 (Rack1/Shelf3)
                                  |
                            IT Office: Rm 2159 (printer + PC)
```

- R1 = Rack 1, Shelf 1; S1 = Rack 1, Shelf 2; S2 = Rack 1, Shelf 3.
- R1 connects to: Internet, S1, S2, S3, S4, S5.
- S1 connects to the Web (Rack 2 Shelf 1), Email (Rack 2 Shelf 2), and File (Rack 2 Shelf 3) servers.

**Logical topology (figure):**

```
                          (Internet)
                              | G0/0
 Network 192.168.10.0         |                    Network 192.168.100.0
 Web Server  --F0/1--+        |                    S3 --F0/1-- laptop
 Email Server--F0/2--S1--G0/1-R1--(G1/0)--G0/1--S3
 File Server --F0/3--+        |  Network 192.168.101.0
                              +--(G1/1)--G0/1--S4--F0/1-- laptop
                              |  Network 192.168.102.0
                              +--(G0/2 / G1/2)--G0/1--S5--F0/1-- laptop
                              |
                              +--(F0/1 / G0/1)--S2--F0/2-- printer + PC
                                                Network 192.168.11.0
```

|Logical Network|Devices|
|---|---|
|192.168.10.0|Web Server (S1 F0/1), Email Server (S1 F0/2), File Server (S1 F0/3), switch S1|
|192.168.11.0|Printer + PC on S2 (PC on F0/2)|
|192.168.100.0|S3 + laptop (laptop on F0/1)|
|192.168.101.0|S4 + laptop (laptop on F0/1)|
|192.168.102.0|S5 + laptop (laptop on F0/1)|

> ⚠️ **Check:** The exact interface-to-link mapping on R1 (labels G0/0, G0/1, G1/0, G1/1, G0/2, G1/2, F0/1) is hard to read from the figure; labels shown on S1/S3/S4/S5 uplinks are G0/1. Verify exact R1 interface assignments in the original slide 13 if needed.

> 💡 **Tip:** Physical = _where things are_ (rooms, racks, shelves, cables). Logical = _how things are addressed_ (networks, ports, IPs).

---

## 1.4 Common Types of Networks

### Networks of Many Sizes

|Network Type|Description|
|---|---|
|**Small Home Networks**|Connect a few computers to each other and the internet|
|**Small Office/Home Office (SOHO)**|Enables computers within a home or remote office to connect to a corporate network|
|**Medium to Large Networks**|Many locations with hundreds or thousands of interconnected computers|
|**World Wide Area Networks (WAN)**|Connects hundreds of millions of computers worldwide – such as the internet|

> ⚠️ **Check:** The slide says "World **Wide** Area Networks (WAN)". The standard term is **Wide Area Network (WAN)**; the "world wide" example (the internet) is the global-scale case.

### LANs and WANs

- Network infrastructures vary greatly in:
    - **Size of the area covered**
    - **Number of users connected**
    - **Number and types of services available**
    - **Area of responsibility**
- Two most common types: **Local Area Network (LAN)** and **Wide Area Network (WAN)**.
- Figure: Home Office, Central, and Branch **LANs** connected through a **WAN cloud** (WAN links shown as red zig-zag lines).
- **LAN**: network infrastructure that spans a **small geographical area**.
- **WAN**: network infrastructure that spans a **wide geographical area**.

|LAN|WAN|
|---|---|
|Interconnect end devices in a limited area|Interconnect LANs over wide geographical areas|
|Administered by a single organization or individual|Typically administered by one or more service providers|
|Provide high-speed bandwidth to internal devices|Typically provide slower speed links between LANs|

```
 Branch 1 LAN                                   Branch 2 LAN
 [PCs][Server][IP Phone]--[Switch]--[Router]~~~WAN~~~[Router]--[Switch]--[PCs][Server][IP Phone]
```

> 🎯 **Exam Focus:** LAN = small area, single administrator, high speed. WAN = wide area, service providers, slower.

### The Internet

- **The internet**: a worldwide collection of interconnected LANs and WANs.
    - LANs are connected to each other using WANs.
    - WANs may use **copper wires, fiber-optic cables, and wireless transmissions**.
- The internet is **not owned by any individual or group**.
- Groups developed to help maintain structure on the internet: **IETF, ICANN, IAB**.

> 🆕 Added: IETF = Internet Engineering Task Force; ICANN = Internet Corporation for Assigned Names and Numbers; IAB = Internet Architecture Board (the PDF gives only the acronyms).

- Figure: Home, Hospital, Government, School, and Business LANs connect via routers into a cloud of interconnected (multilayer) switches/WAN links.

### Intranets and Extranets

- **Intranet**: a **private** collection of LANs and WANs **internal to an organization**, meant to be accessible only to the organization's members or others with authorization.
- **Extranet**: used by an organization to provide **secure access** to its network for individuals who work for a **different organization** and need access to its data.

```
 +----------------------------------------------+
 |  The Internet  (The World)                   |
 |   +--------------------------------------+   |
 |   | Extranet (Suppliers, Customers,      |   |
 |   |           Collaborators)             |   |
 |   |   +------------------------------+   |   |
 |   |   | Intranet (Company Only)      |   |   |
 |   |   +------------------------------+   |   |
 |   +--------------------------------------+   |
 +----------------------------------------------+
```

---

## 1.5 Internet Connections

### Internet Access Technologies

- Popular services for **home users and small offices**: **broadband cable, broadband digital subscriber line (DSL), wireless WANs, mobile services**.
- **Organizations** need faster connections to support **IP phones, video conferencing, and data center storage**.
- **Business-class interconnections** are usually provided by **service providers (SP)** and may include: **business DSL, leased lines, Metro Ethernet**.

### Home and Small Office Internet Connections

- Figure: Home User, Teleworker, and Small Office connect to an **Internet Service Provider** via DSL, Cable, Cellular, Satellite, or Dial-Up Telephone; the ISP connects to the Internet.

|Connection|Description|
|---|---|
|**Cable**|High bandwidth, always on, internet offered by cable television service providers|
|**DSL**|High bandwidth, always on, internet connection that runs over a telephone line|
|**Cellular**|Uses a cell phone network to connect to the internet|
|**Satellite**|Major benefit to rural areas without Internet Service Providers|
|**Dial-up telephone**|An inexpensive, low-bandwidth option using a modem|

### Businesses Internet Connections

- Corporate business connections may require: **higher bandwidth, dedicated connections, managed services**.
- Figure: Organization connects to ISP through Dedicated Leased Lines, Metro Ethernet, Business DSL, or Satellite.

|Type of Connection|Description|
|---|---|
|**Dedicated Leased Line**|Reserved circuits within the service provider's network that connect distant offices with private voice and/or data networking|
|**Ethernet WAN**|Extends LAN access technology into the WAN|
|**DSL**|Business DSL is available in various formats including **Symmetric Digital Subscriber Lines (SDSL)**|
|**Satellite**|Can provide a connection when a wired solution is not available|

### The Converging Network

- **Before converged networks**: an organization was **separately cabled** for telephone, video, and data. Each network used **different technologies** to carry the signal, and each used a **different set of rules and standards**.
    - Figure: separate Computer, Telephone, and Broadcast networks, each with its own medium, message, and rule/agreement/standard.
- **Converged data networks** carry multiple services on **one link**: **data, voice, video**.
- Converged networks deliver data, voice, and video over the **same network infrastructure**, which uses the **same set of rules and standards**.
    - Figure: "One Network – Multiple Devices".

> 🎯 **Exam Focus:** Convergence = data + voice + video on one infrastructure using one set of rules/standards.

### Labs / Videos in this Topic (no technical content)

- Video – Download and Install Packet Tracer.
- Video – Getting Started in Cisco Packet Tracer: navigate the Packet Tracer interface; customize the Packet Tracer interface.
- **Packet Tracer – Network Representation**: the network model incorporates many technologies you will master in CCNA. _Note: it is not important to understand everything you see and do in this activity._

---

## 1.6 Reliable Networks

### Network Architecture

- **Network architecture**: the technologies that support the infrastructure that moves data across the network.
- **Four basic characteristics** the underlying architectures must address to meet user expectations:
    1. **Fault Tolerance**
    2. **Scalability**
    3. **Quality of Service (QoS)**
    4. **Security**

> 🧠 **Memory Aid:** **F**aults, **S**cale, **Q**oS, **S**ecurity → "**F**ive **S**ervers **Q**uietly **S**tay" (F-S-Q-S).

### Fault Tolerance

- A **fault tolerant network** limits the impact of a failure by **limiting the number of affected devices**.
- **Multiple paths** are required for fault tolerance.
- Reliable networks provide **redundancy** by implementing a **packet-switched network**:
    - Packet switching **splits traffic into packets** that are routed over a network.
    - Each packet could theoretically take a **different path** to the destination.
- This is **not possible with circuit-switched networks**, which establish **dedicated circuits**.
- Figure: when one router fails, redundant connections allow alternative paths; the user experience is unaffected.

### Scalability

- A **scalable network** can expand quickly and easily to support **new users and applications** without impacting the performance of services to existing users.
- Network designers follow **accepted standards and protocols** to make networks scalable.
- Figure: additional users and whole networks can be connected to the internet without degrading performance for existing users.

### Quality of Service (QoS)

- Voice and live video transmissions require higher expectations for the services being delivered.
- Choppy live video (constant breaks and pauses) is caused when **demand for bandwidth is higher than available** and **QoS isn't configured**.
- **QoS** is the **primary mechanism** used to ensure reliable delivery of content for all users.
- With a QoS policy in place, the **router** can more easily manage the flow of data and voice traffic.
- Figure: QoS, managed by the router, ensures priorities match the type of communication and its importance to the organization. **Web pages can usually receive a lower priority.** A **Voice over IP (VoIP) call** needs priority to maintain a smooth, uninterrupted user experience.

### Network Security

- **Two main types of network security** that must be addressed:
    - **Network infrastructure security**
        - Physical security of network devices
        - Preventing unauthorized access to the devices
    - **Information security**
        - Protection of the information or data transmitted over the network
- **Three goals of network security:**

|Goal|Meaning|
|---|---|
|**Confidentiality**|Only intended recipients can read the data|
|**Integrity**|Assurance that the data has not been altered during transmission|
|**Availability**|Assurance of timely and reliable access to data for authorized users|

> 🆕 Added: These three goals are commonly known as the **CIA triad** (the PDF does not use this name).

- Figure: administrators protect the network with software and hardware security and by preventing physical access to network devices (login/password prompt on a router).

> 🎯 **Exam Focus:** Four reliable-network characteristics, the difference between packet-switched and circuit-switched, and the three security goals.

---

## 1.7 Network Trends

### Recent Trends

- The role of the network must adjust and continually transform to keep up with new technologies and end-user devices.
- Trends affecting organizations and consumers: **Bring Your Own Device (BYOD)**, **Online collaboration**, **Video communications**, **Cloud computing**.

### Bring Your Own Device (BYOD)

- Allows users to use their **own devices**, giving them more opportunities and greater flexibility.
- Users can use personal tools to access information and communicate using their: **laptops, netbooks, tablets, smartphones, e-readers**.
- **BYOD means any device, with any ownership, used anywhere.**

### Online Collaboration

- Collaborate and work with others over the network on **joint projects**.
- Collaboration tools including **Cisco WebEx** give users a way to **instantly connect and interact**.
- Collaboration is a very high priority for businesses and education.
- **Cisco Webex Teams**: multifunctional collaboration tool – send instant messages, post images, post videos and links.

### Video Communication

- Video calls can be made to anyone regardless of location.
- Video conferencing is a powerful tool for communicating with others.
- Video is becoming a **critical requirement** for effective collaboration.
- **Cisco TelePresence** is one way of working where everyone, everywhere.
- (Video – Cisco WebEx for Huddles: no technical content.)

> ⚠️ **Check:** Slide text reads "Cisco TelePresence powers is one way of working where everyone, everywhere" – wording is garbled in the original.

### Cloud Computing

- Allows us to **store personal files or back up data** on servers over the internet.
    - Applications can also be accessed using the cloud.
    - Allows businesses to deliver to **any device anywhere in the world**.
- Cloud computing is made possible by **data centers**.
    - Smaller companies that can't afford their own data centers **lease server and storage services** from larger data-center organizations in the cloud.

**Four types of clouds:**

|Cloud Type|Description|
|---|---|
|**Public**|Available to the general public through a **pay-per-use** model or for free|
|**Private**|Intended for a **specific organization or entity**, such as the government|
|**Hybrid**|Made up of **two or more cloud types** (e.g., part custom and part public). Each part remains a distinctive object, but both are connected using the same architecture|
|**Custom**|Built to meet the needs of a **specific industry**, such as healthcare or media. Can be private or public|

### Technology Trends in the Home

- **Smart home technology**: integrates technology into everyday appliances so they can **interconnect with other devices**.
- Example: ovens might know what time to cook a meal by communicating with your calendar about what time you are scheduled to be home.
- Smart home technology is currently being developed for **all rooms** within a house.

### Powerline Networking

- Allows devices to connect to a **LAN** where data network cables or wireless communications are **not a viable option**.
- Using a **standard powerline adapter**, devices can connect to the LAN **wherever there is an electrical outlet**, by **sending data on certain frequencies**.
- Especially useful when **wireless access points cannot reach all the devices** in the home.
- Figure: Wireless-N Router connected by wired link to powerline adapters (PLEK400 4-Port Powerline Adapter, PLE400, PLSK400 4-Port Powerline Adapter) over the home's electrical wiring (powerline connection = orange; wired connection = purple).

### Wireless Broadband

- In addition to DSL and cable, **wireless** is another option to connect homes and small businesses to the internet.
- **Wireless Internet Service Provider (WISP)**: an ISP that connects subscribers to designated **access points or hotspots**; more commonly found in **rural environments**.
- **Wireless broadband** is another solution for the home and small businesses:
    - Uses the **same cellular technology** used by a smartphone.
    - An **antenna is installed outside the house**, providing wireless or wired connectivity for devices in the home.

---

## 1.8 Network Security

### Security Threats

- Network security is an **integral part of networking regardless of network size**.
- Implemented security must take into account the environment while securing data, **but still allow the quality of service expected of the network**.
- Securing a network involves many **protocols, technologies, devices, tools, and techniques** to secure data and mitigate threats.
- **Threat vectors** might be **external or internal**.
- Figure: external threat comes from the Internet through the firewall; internal threat originates inside the LAN; both can lead to a **compromised host**.

|External Threats|Internal Threats|
|---|---|
|Viruses, worms, and Trojan horses|Lost or stolen devices|
|Spyware and adware|Accidental misuse by employees|
|Zero-day attacks|Malicious employees|
|Threat actor attacks||
|Denial of service attacks||
|Data interception and theft||
|Identity theft||

### Security Solutions

- Security must be implemented in **multiple layers** using **more than one** security solution.
- **Home or small office** network security components:
    - **Antivirus and antispyware software** installed on end devices.
    - **Firewall filtering** to block unauthorized access to the network.
- **Larger networks** have additional security requirements:
    - **Dedicated firewall system**
    - **Access control lists (ACL)**
    - **Intrusion prevention systems (IPS)**
    - **Virtual private networks (VPN)**
- The study of network security starts with a clear understanding of the underlying **switching and routing infrastructure**.
- Figure: Internet → routers with Access Lists/ACLs → firewalls → LANs with servers and PCs.

> 💡 **Tip:** "Multiple layers" = defense in depth: endpoint software (antivirus/antispyware) + firewall for small networks; add dedicated firewall, ACL, IPS, VPN for larger ones.

---

## 1.9 / 1.10 Module Practice and Quiz

- The PDF only has a section divider and a Cisco logo slide. No technical content.

---

## 📊 Quick Reference

### Device Roles

|Term|Role|
|---|---|
|Host / End device|Any computer on the network; where messages originate or are received|
|Server|Provides information (email, web, file)|
|Client|Sends requests to servers|
|Intermediary device|Interconnects end devices: switch, wireless AP, router, firewall|

### Media

|Medium|Signal|
|---|---|
|Copper (metal wires)|Electrical impulses|
|Fiber-optic (glass/plastic)|Pulses of light|
|Wireless|Modulated electromagnetic waves|

### Topology Diagram Types

|Type|Shows|
|---|---|
|Physical|Physical location of intermediary devices and cable installation|
|Logical|Devices, ports, and addressing scheme|

### Network Types

|Type|Key Point|
|---|---|
|Small Home|A few computers + internet|
|SOHO|Home/remote office to corporate network|
|Medium/Large|Many locations, hundreds/thousands of computers|
|WAN (worldwide)|Hundreds of millions of computers, e.g., the internet|
|LAN|Small area, single org, high speed|
|WAN|Wide area, service providers, slower|
|Internet|Worldwide collection of LANs and WANs; not owned by anyone (IETF, ICANN, IAB)|
|Intranet|Private, organization only|
|Extranet|Secure access for outside organizations (suppliers, customers, collaborators)|

### Internet Connection Options

|Environment|Options|
|---|---|
|Home / small office|Cable, DSL, cellular, satellite, dial-up, wireless WAN/WISP, mobile|
|Business|Dedicated leased line, Ethernet WAN (Metro Ethernet), business DSL (e.g., SDSL), satellite|

### Reliable Network Characteristics

|Characteristic|Summary|
|---|---|
|Fault tolerance|Limits impact of failure; multiple paths; packet switching (vs. circuit switching)|
|Scalability|Expands without hurting existing users; standards and protocols|
|QoS|Prioritizes traffic (e.g., VoIP over web pages); router manages|
|Security|Infrastructure security + information security; Confidentiality, Integrity, Availability|

### Cloud Types

|Public|Private|Hybrid|Custom|
|---|---|---|---|
|General public, pay-per-use or free|Specific organization (e.g., government)|Two or more cloud types|Specific industry; private or public|

### Security Solutions

|Network Size|Solutions|
|---|---|
|Home / small office|Antivirus + antispyware, firewall filtering|
|Larger networks|Dedicated firewall, ACLs, IPS, VPN|

---

## ✅ Key Takeaways

- Every computer on a network is a **host/end device**; **servers** provide information, **clients** request it. In a **P2P** network a device is both.
- **Intermediary devices** (switches, wireless APs, routers, firewalls) interconnect end devices, regenerate/retransmit signals, maintain path information, and notify of errors.
- Media: **copper = electrical impulses, fiber = light pulses, wireless = electromagnetic waves**.
- **Physical** topology diagrams show location/cabling; **logical** diagrams show ports and addressing.
- **LAN** = small area, single administrator, high speed; **WAN** = wide area, service providers, slower. The **internet** is a worldwide collection of LANs and WANs not owned by anyone (IETF, ICANN, IAB).
- **Intranet** = private to the organization; **extranet** = secure access for external partners.
- **Converged networks** carry data, voice, and video on one infrastructure with one set of rules/standards.
- A reliable network needs **fault tolerance, scalability, QoS, and security**; fault tolerance relies on **packet switching** (not circuit switching).
- Security goals: **confidentiality, integrity, availability**. Use **layered** security (antivirus/antispyware + firewall; larger networks add ACLs, IPS, VPN).
- Trends: **BYOD, online collaboration, video, cloud computing** (public/private/hybrid/custom), smart home, powerline networking, wireless broadband (WISP).

---

## 🔎 Coverage Check

- [x] Module objectives table (all 9 topics; "The IT Professional" has no slides in PDF – flagged)
- [x] 1.1 Networks Affect Our Lives (video only)
- [x] Host roles: host/end device, servers, clients, server-type table
- [x] Peer-to-peer: definition, advantages/disadvantages table, figure
- [x] End devices and figure with alternate routes
- [x] Intermediary devices: examples, three management roles
- [x] Network media: three types and signal types
- [x] Network representations: topology diagrams, NIC, physical port, interface, symbols, media line styles
- [x] Physical vs. logical topology diagrams (both figures recreated; interface labels flagged)
- [x] Networks of many sizes (home, SOHO, medium/large, WAN)
- [x] LANs vs. WANs (factors, definitions, comparison table)
- [x] The internet (IETF, ICANN, IAB)
- [x] Intranets and extranets
- [x] Internet access technologies; home/small office connections; business connections
- [x] Converging network (before/after)
- [x] Videos and Packet Tracer activity (Network Representation)
- [x] Reliable networks: four characteristics, fault tolerance, scalability, QoS, security (two types, three goals)
- [x] Trends: BYOD, collaboration (WebEx, Webex Teams), video (TelePresence), cloud computing (4 types), smart home, powerline, wireless broadband/WISP
- [x] Network security: threats (external/internal), solutions (home vs. larger networks)
- [x] 1.10 Module Practice and Quiz (no content)
- [x] No CLI commands, configuration, or verification content exists in this PDF section, so those sections are omitted




