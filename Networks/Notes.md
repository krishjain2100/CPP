A protocol defines the format and the order of messages exchanged between two or more communicating entities, as well as the actions taken on the transmission and/or receipt of a message or other event.

---
#### End Systems

The computers and other devices connected to the Internet are often referred to as end systems. E.g., Desktop PCs, Macs, and Linux boxes), servers (e.g., web and e-mail servers),
mobile devices (e.g., laptops, smartphones, and tablets)

End systems are also referred to as hosts because they host (i.e,, run) application programs such as a Web browser program, a Web server program, an e-mail client program, or an e-mail server programs. Hosts further divided: clients and servers. Clients tend to be desktops, laptops, smartphones, whereas servers tend to be more powerful machines that store and distribute web pages, stream video, relay e-mail, and so on. Most of the servers from which we receive search results reside in large data centres.

---
### Access Networks

The network that physically connects an end system to the first router (also known as the edge router) on a path from the end system to any other distant end system.

The two most prevalent types of broadband residential access are:
- Digital Subscriber Line (DSL)
- Cable

A residence typically obtains DSL Internet access from the same local telephone company (telco) that provides its wired local phone access. Thus, when DSL is used, a customer’s telco is also its ISP. Each customer’s DSL modem uses the existing telephone line exchange data with a **digital subscriber line access multiplexer (DSLAM)** located in the telco’s local central office (CO). The home’s DSL modem takes digital data and translates it to high frequency tones for transmission over telephone wires to the CO, the analog signals from many such houses are translated back into digital format at the DSLAM.


![[DSL.png]]


The residential telephone line carries both data and traditional telephone signals simultaneously, which are encoded at different frequencies:
- A high-speed downstream channel (internet to you), in the 50 kHz to 1 MHz band
- A medium-speed upstream channel (you to internet), in the 4 kHz to 50 kHz band
- An ordinary two-way telephone channel, in the 0 to 4 kHz band

This approach makes the single DSL link appear as if there were three separate links, so that a telephone call and an Internet connection can share the DSL link at the same time. On the customer side, a splitter separates the data and telephone signals arriving to the home and forwards the data signal to the DSL modem. On the telco side, in the CO, the DSLAM separates the data and phone signals and sends the data into the Internet. Hundreds or even thousands of households connect to a single DSLAM.

Cable Internet access makes use of the cable television company’s existing cable television infrastructure. Fiber optics connect the cable head end to neighbourhood junctions, from which traditional coaxial cable is then used to reach individual houses and apartments. Each neighbourhood junction typically supports 500 to 5,000 homes. Because both fiber and coaxial cable are employed in this system, it is often referred to as **hybrid fiber coax (HFC)**.

![[Cable.png]]

As with a DSL modem, the cable modem is typically an external device and connects to the home PC through an Ethernet port. At the cable head end, the **Cable Modem Termination System (CMTS)** serves a similar function as the DSL network’s DSLAM, turning the analog signal sent from the cable modems in many downstream homes back into digital format. Cable modems divide the HFC network into two channels, a downstream and an upstream channel. As with DSL, access is typically asymmetric, with the downstream channel typically allocated a higher transmission rate than the upstream channel. 

One important characteristic of cable Internet access is that it is a shared broadcast medium. For this reason, if several users are simultaneously downloading a video file on the downstream channel, the actual rate would be lowered. Because the upstream channel is also shared, a distributed multiple access protocol is needed to coordinate transmissions and avoid collisions. 

An up-and-coming technology that provides even higher speeds is **fiber to the home (FTTH)**. 
This provide an optical fiber path from the CO directly to the home. The simplest optical distribution network is called direct fiber, with one fiber leaving the CO for each home. More commonly, each fiber leaving the central office is actually shared by many homes, it is not until the fiber gets relatively close to the homes that it is split into individual customer-specific fibers.

---

![[Screenshot 2026-08-10 at 11.07.52 PM.png]]

**Cable or DSL Modem**: The modem serves as the physical bridge between your home network and your Internet Service Provider (ISP). Its primary function is modulation and demodulation, translating the analog frequencies traveling over the ISP's external wiring (like copper phone lines or coaxial cables) into the digital data packets used by your home networking equipment, and vice versa.

**Router (with Firewall and NAT)**: The router acts as the central hub that directs data traffic between your local devices and the external internet. It utilises Network Address Translation (NAT) to allow all your local devices to share a single public IP address assigned by the ISP. Furthermore, its built-in firewall provides perimeter security by inspecting and blocking unauthorised incoming traffic from reaching your local network.

**WiFi Wireless Access Point**: The access point operates as a wireless transmitter and receiver that bridges wireless devices to your wired local area network (LAN). It converts digital network data into radio waves, broadcasting the Wi-Fi signal so that smartphones, laptops, and smart appliances can communicate with the router without requiring physical Ethernet cables.

---
### Wireless access networks:

- **Wireless Local Area Networks (WLANs)**
	- **Range & Use Case:** Designed for short-range coverage, typically spanning about 100 feet. This is the standard technology used inside and immediately around a single building, such as a house or an office.
	- **Technology & Speeds:** Operates using the 802.11 family of standards (which we commonly call Wi-Fi). Depending on the specific standard used (such as 802.11b, g, or n), transmission rates historically sit at 11, 54, or 450 Mbps.
	- **Base Station:** The base station in this setup is the local **wireless access point** (often built into your home router).
	
- **Wide-Area Cellular Access Networks**
	- **Range & Use Case:** Designed for long-range, widespread coverage spanning tens of kilometers. This allows devices to stay connected while moving across cities or highways.
	- **Technology & Speeds:** Provided and managed by mobile network operators (telecom companies) using infrastructure like 4G and 5G networks. The typical speeds are in the 10's of Mbps, though modern 5G can peak much higher.
	- **Base Station:** The base station in this setup is the physical **cellular tower**.

---
### Enterprise Network

Unlike a simple home network where all functions are crammed into a single box, an enterprise network (used by companies, universities, and large campuses) uses a hierarchy of dedicated, specialized devices to handle a massive amount of traffic and users.

![[Screenshot 2026-08-10 at 11.24.25 PM.png]]

The diagram illustrates the flow of data from the end-user up to the internet, using a mix of interconnected devices:

- **End-User Devices:** Desktops connect via physical cables, while laptops connect wirelessly.
- **Ethernet Switch:**  Instead of plugging every single desktop directly into a router, they all plug into a local switch. The wireless access point also plugs into this switch. The switch connects all these local devices together so they can communicate.
- **Institutional Servers:** Large organisations often host their own local services, such as internal email servers or web servers. In the diagram, these are connected to their own switch, keeping their traffic organised.
- **Institutional Router:** It connects the various internal switches (the user switch and the server switch) together, and provides the main outbound link to the external Internet Service Provider (ISP).

---
### Data Centres and Cloud Computing

Data center networks are designed to host the massive amounts of _content and services_ that those people are trying to access. Because these servers are hosting services used by thousands or millions of people simultaneously, the connections between the servers and to the outside internet must be incredibly fast.  The previously discussed networks (mobile, home, and enterprise) on the left side of the diagram. They represent the "edges" where users live. The data centres represent the core destinations.

 Because of their massive traffic demands, they connect directly to **national or global ISPs** or operate within their own dedicated **content provider networks** (like the private networks run by Google or Netflix).

![[Screenshot 2026-08-10 at 11.30.46 PM.png]]

Internet companies such as Google, Microsoft, Amazon have built massive data centers, each housing tens to hundreds of thousands of hosts. These data connected to the Internet, as well as internally. Amazon data centres serve three purposes:
- They serve Amazon e-commerce pages to users, for example, pages describing products and purchase information. 
- They serve as massively parallel computing infrastructures for Amazon-specific data processing tasks. 
- They provide cloud computing to other companies. Today a major trend in computing is for companies to use a cloud provider such as Amazon to handle all of their IT needs. For example, Airbnb edo not own and manage their own data centres but instead run their entire web-based services in the Amazon cloud, called Amazon Web Services (AWS).

The hosts in data centres, called blades and resembling pizza boxes, are generally commodity hosts that include CPU, memory, and disk storage. The hosts are stacked in racks, with each rack typically having 20 to 40 blades. The racks are then interconnected using sophisticated and evolving data centre network designs.

---
### The Network Core

The network core is a global mesh of interconnected routers. Itrepresents the regional, national, and global ISPs that act as the backbone connecting all the "edge" access networks (homes, mobile networks, enterprises, and data centres) together.

The core operates using packet-switching. End-system hosts break messages down into smaller data chunks called **packets**. The network then forwards these packets hop-by-hop from one router to the next across links until they reach their final destination.

**Routing (The Global Action)**
- Routing is the high-level, network-wide planning phase.
- It utilises **routing algorithms** that communicate across multiple routers to determine the most efficient end-to-end path a packet should take from its source to its destination.

**Forwarding (The Local Action)**
- Forwarding (also known as "switching") is the localized, physical action that happens inside a single, specific router.
- When a packet arrives at a router's input link, the router inspects the destination address in the packet's header. It then consults its **local forwarding table** to find the corresponding output link (link "32" in the table maps to output link "2") and physically moves the packet to that output link.

In standard internet packet-switching, the complete path is **not** determined before the host sends the packet. Instead, the path is navigated step-by-step, dynamically at every single router along the way.

---
### Packet Switching

Routers do not forward data bit-by-bit as it arrives. They use a **store-and-forward** method, meaning the router must receive and store the _entire_ packet before it can begin transmitting the first bit of that packet onto the next link.

**Transmission Delay:** The time it takes to push a packet out onto a link is calculated using the formula $L/R$. $L$ represents the packet size (in bits) and $R$ represents the transmission rate of the link (in bits per second, or bps).

**Queueing:** This happens when packets arrive at a router faster than the router can send them out. Assume, Hosts A and B are connected to the router with very fast links (100 Mb/s). However, the router's output link to the rest of the network is much slower (1.5 Mb/s).

A router's memory buffer is finite; it can only hold so many queued packets at once. If the high-speed incoming traffic continues for too long, the router's memory buffer will eventually fill up entirely. When a new packet arrives and there is no room left in the buffer, the router has no choice but to drop (delete) it. This is known as **packet loss**, and it is the primary reason why video streams buffer or downloads stall during heavy network congestion.

---
### Circuit Switching

Unlike packet switching (which dynamically shares network links), circuit switching works by allocating and **reserving dedicated end-to-end resources** for a specific session or "call" between a source and a destination.

Because a dedicated path is carved out just for that connection, there is no sharing with other users, which guarantees a consistent, predictable level of performance. This is the model historically used by traditional telephone networks.

But if a user has a reserved circuit but isn't actively sending data at a given moment, that circuit segment remains idle. Because it is reserved, no other user's data can jump in and use that idle bandwidth.

---
### Comparing Packet and Circuit Switching

Imagine a link that can handle **1 Gb/s (1,000 Mb/s)**. Each user requires **100 Mb/s** when they are actively transmitting data, but they only actually transmit data **10% of the time**.

- **Under Circuit Switching:** The network must reserve 100 Mb/s for every user, permanently. Therefore, the link can only support exactly **10 users** (1,000 / 100), even though 90% of the time, those reserved links are sitting idle.

- **Under Packet Switching:** Because packet switching shares the link on-demand, it can support many more people. The math shows that if you put **35 users** on this same link, the probability of more than 10 of them trying to transmit data at the exact same millisecond is extremely low (less than 0.0004). By letting users share the pipe, packet switching supports over three times as many users on the exact same hardware.

While packet switching is the foundation of the internet, it isn't perfect.

- **The Pros:** It is vastly superior for "bursty" data (like web browsing, where you request a page and then sit reading it), it allows for massive resource sharing, and it avoids the complex setup required to establish a dedicated circuit.

- **The Cons:** Because resources aren't reserved, the network can suffer from **excessive congestion**. If too many people try to send data at once, it leads to the queueing delays and packet loss (buffer overflow) discussed in previous slides. Because of this, packet-switched networks require complex protocols layered on top to manage reliable data transfer and control congestion.

---
### How Circuits are Divided (FDM vs TDM)

When a physical link is split into multiple circuits, it is typically done using one of two multiplexing methods:

![[Screenshot 2026-08-11 at 12.05.21 AM.png]]

- **Frequency Division Multiplexing (FDM):** The link's total capacity is divided into narrow, individual frequency bands (similar to different stations on a radio dial). A call is allocated its own specific frequency band and can transmit constantly, but only at the maximum rate of that narrow band.

- **Time Division Multiplexing (TDM):** The link is divided into periodic time slots. A call is allocated specific, repeating time slots. During its turn, the call can transmit at the maximum rate of the _entire_ wider frequency band, but it must wait in between its assigned slots.

---
### The Network of Networks

- End systems connect to the Internet via access ISPs, which can be traditional telecom companies, cable companies, universities, or corporate enterprise networks.
- To ensure that any host anywhere in the world can send packets to any other host, these access ISPs must be interconnected.
- The resulting global structure is incredibly complex and has evolved primarily due to economics and national policies, rather than being optimized solely for performance.

![[Screenshot 2026-08-11 at 12.39.04 AM.png]]

#### Structural Evolution

- **The Mesh Failure:** Attempting to connect millions of access ISPs directly to one another creates an O(N^2) scaling problem, making a direct mesh design far too costly to build.

- **Global Transit ISPs:** The initial solution requires access ISPs to connect to a centralised global transit ISP, establishing a customer-provider economic relationship where the lower-tier access ISP pays for connectivity.

- **The Tiered Hierarchy:** Profitability leads to competition, resulting in multiple global networks known as Tier-1 commercial ISPs (e.g., Level 3, Sprint, AT&T, NTT) that provide national and international coverage.

- **Regional ISPs:** Because Tier-1 ISPs cannot maintain a physical presence in every single city, regional networks emerge as intermediaries to connect local access ISPs to the global higher-tier networks.

#### Interconnection Mechanisms

- **Customer-Provider Relationship**: In such a hierarchy, each access ISP pays the regional ISP to which it connects, and each regional ISP pays the tier-1 ISP to which it connects. (An access ISP can also connect directly to a tier-1 ISP, in which case it pays the tier-1 ISP). Thus, there is customer-provider relationship at each level of the hierarchy. Note that the tier-1 ISPs do not pay anyone as they are at the top of the hierarchy

- **Peering Links:** The amount that a customer ISP pays a provider ISP reflects the amount of traffic it exchanges with the provider. To reduce these costs, a pair of nearby ISPs at the same level of the hierarchy can peer, that is, they can directly connect their networks together so that all the traffic between them passes over the direct connection rather than through upstream intermediaries. This is typically a settlement-free (cost-free) agreement.

- **Internet Exchange Points (IXPs):** These are stand-alone physical facilities and meeting points where multiple ISPs can peer their networks together.

- **Points of Presence (PoPs):** A localised group of routers within a provider's network that gives customer ISPs a physical location to connect into the broader network.

- **Multi-homing:** A reliability practice where an ISP connects to two or more upstream provider ISPs simultaneously, ensuring their network stays online even if one provider experiences a failure.

#### Content Provider Networks

- Massive technology companies like Google, Microsoft, and Akamai operate their own private networks to bring content and services physically closer to end users.

- These private networks link thousands of their distributed data centres globally and connect them directly into the public Internet.

- To reduce payments to upper-tier ISPs and maintain strict control over service delivery, content provider networks actively attempt to bypass Tier-1 and regional ISPs by peering directly with lower-tier access ISPs or connecting at local IXPs.

---
### Network Models

Designing the Internet involves managing a massively complex system:

- It must support billions of users and span the entire globe.
- Different applications need the network to behave in completely different ways (e.g., some need highly reliable data transfer, while others need cheap, real-time interactive speeds or multicasting capabilities).
- The network must seamlessly integrate completely different physical technologies, such as wired Ethernet, WiFi, Bluetooth, and Cellular networks.

To handle this massive complexity, network designers use an object-oriented, layered approach.

- Instead of building one massive, monolithic network program, the system is broken down into smaller, manageable pieces (objects or layers).
- Each layer has a specific job and communicates with the layers directly above and below it using clearly defined **interfaces**. The layers do not need to know _how_ the other layers work, only _how to talk to them_ via the interface.


Using a layered architecture provides several critical benefits to network design:

- **Modular Design:** Breaking the system into distinct layers drastically reduces the overall complexity of the network.

- **Software Reuse:** Upper layers can easily share and reuse the functionality of the layers below them. For example, both web browsing (HTTP) and email (SMTP) applications can just use the exact same underlying TCP layer to handle their data transfer.

- **Abstraction & Flexibility:** Because the internal implementation of a layer is hidden, you can easily update, improve, or change specific parts of a layer to adopt new technologies. As long as the interface (the way it talks to the other layers) remains the exact same, the rest of the network won't even notice the change.

Historically, there were different ways to conceptualize network layers.

- **The OSI Reference Model:** Developed by the ISO in the late 1970s, this is a theoretical 7-layer standard (Application, Presentation, Session, Transport, Network, Link, Physical). It was designed to specify the exact functionality and interfaces between layers.

- **The Internet Protocol Stack:** This is the practical, **5-layer model** that the modern Internet actually uses. It simplifies the OSI model by combining the top three layers into a single Application layer.

---
### The Internet Protocol Stack

**Application Layer:** Supports network applications and generates the actual data. Examples: HTTP (Web), IMAP/SMTP (Email), DNS, File-transfer.

**Transport Layer:** Handles process-to-process data transfer between the source and destination devices. It deals with multiplexing/demultiplexing and reliability. Examples: TCP (reliable), UDP (unreliable, fast).

**Network Layer:** Responsible for routing the data across the global network from the source host to the destination host. Examples: IP (Internet Protocol), routing protocols.

**Link Layer:** Manages the actual transfer of data between neighboring network elements (like from your computer to your home router). Examples: Ethernet, 802.11 (WiFi), PPP.

**Physical Layer:** Handles the physical transmission of the raw bits (1s and 0s) "on the wire" or through the air.

---
### Encapsulation Process

When a host sends data, it travels top-down through the Internet protocol stack. At each step, the current layer takes the entire package from the layer above it and _encapsulates_ it by adding its own specific header.

1. **The Message (Application Layer):** The process begins when a network application (like a web browser or email client) creates the raw data. At this top level, the data is called a **Message** ($M$).

2. **The Segment (Transport Layer):** The Application layer passes the Message down to the Transport layer. The Transport layer encapsulates the Message by adding its own transport-layer header ($H_t$), which contains instructions for process-to-process delivery and reliability. This combined package ($H_t$ + $M$) is now called a **Segment**.

3. **The Datagram (Network Layer):** The Transport layer passes the Segment down to the Network layer. The Network layer then encapsulates the _entire_ Segment by adding its own network-layer header ($H_n$), which contains the IP routing instructions needed to move the data across the global internet. This new package ($H_n$ + $H_t$ + $M$) is called a **Datagram**.

4. **The Frame (Link Layer):** The encapsulation process finishes at the Link layer. The Link-layer protocol takes the entire Network-layer Datagram and encapsulates it by adding its own link-layer header ($H_l$). This header contains the information needed to transfer the data to the very next physical node (like from your computer to your local router). This final, fully packaged unit of data ($H_l$ + $H_n$ + $H_t$ + $M$) is called a **Frame**.

5. **Physical Transmission:** Finally, the Link layer passes the Frame down to the Physical layer, where the entire package is converted into raw bits (1s and 0s) and transmitted over the wire or through the air.

When this package reaches its final destination, the process happens in reverse (decapsulation), with each layer stripping off its corresponding header as the data moves back up the stack to the receiving application.

Not every device processes all five layers:

1. **The Source:** The data travels top-down, starting at the Application layer and moving down to the Physical layer. A new header ($H_t$, $H_n$, $H_l$) is added at each step before it is sent over the wire.

2. **Switches:** A switch is a simpler device that only operates up to the Link layer. It reads the outermost Link header to forward the frame locally, but it does not look any further to look at the IP routing information inside.

3. **Routers:** A router is more complex and operates up to the Network layer. When a router receives a frame, it strips off the old Link header, reads the Network header ($H_n$) to determine the next global hop, packages it inside a brand new Link header, and sends it on.

4. **The Destination:** Once the frame reaches its final destination, the data moves bottom-up through the stack. Each layer strips off its corresponding header (decapsulation) until only the original message is handed to the receiving application.

![[Screenshot 2026-08-11 at 12.52.45 AM.png]]

#### Where are they implemented?

- **Software:** The Application and Transport layers are almost exclusively implemented in software on the end systems.

- **Hardware:** Because the Physical and Link layers handle communication over specific physical mediums, they are typically hardware-based, built right into a device's network interface card (like a WiFi or Ethernet card).

- **Mixed:** The Network layer is often a mixed implementation of both hardware and software.

---
### Packet Delay

- The concept of traffic intensity is calculated as $La/R$, where $L$ is the packet length, $a$ is the average arrival rate, and $R$ is the transmission rate.
- If the traffic intensity exceeds 1, the average rate of arriving bits is faster than the transmission rate.

As a packet travels from one node to the next, it accumulates four distinct types of delays, forming the total nodal delay equation: $d_{nodal} = d_{proc} + d_{queue} + d_{trans} + d_{prop}$.

- **Nodal Processing Delay ($d_{proc}$):** This is the time a router takes to check for bit-level errors and determine the correct output link. This delay is typically less than a few microseconds.

- **Queueing Delay ($d_{queue}$):** This represents the time a packet spends waiting at the output link for transmission. The length of this delay depends heavily on the router's congestion level and traffic intensity.

- **Transmission Delay ($d_{trans}$):** This is the time required to push all of the packet's bits into the physical link. It is calculated using the formula $L/R$, where $L$ is the packet length in bits and $R$ is the link transmission rate in bps.

- **Propagation Delay ($d_{prop}$):** This is the time it takes a bit to physically travel from the beginning of the link to the next router. It is calculated using the formula $d/s$, where $d$ is the length of the physical link and $s$ is the propagation speed (approximately $2 \times 10^8$ m/sec).

**Important Distinction:** Transmission delay and propagation delay are very different concepts. Transmission delay relies entirely on the packet's size and the link's transmission rate, whereas propagation delay relies entirely on the physical distance between the two routers.

One important aspect is the fact that as the traffic intensity approaches 1, the average queuing delay increases rapidly. A small percentage increase in the intensity will result in a much larger percentage-wise increase in delay. 

The fraction of lost packets increases as the traffic intensity increases. Therefore, performance at a node is often measured not only in terms of delay, but also in terms of the probability of packet loss.

![[Screenshot 2026-08-11 at 1.43.29 AM.png]]

---
### Measuring Real Internet Delays (`Traceroute`)

- `Traceroute` is a software program that provides delay measurements from a source to each router along an end-to-end path toward a destination.
- The source sends three special probe packets to each router $i$ along the path.
- Upon receiving these packets, router $i$ returns a short message back to the sender.
- The sender then measures the time interval between transmission and reply to determine the round-trip delay for that specific hop.

---
### Throughput

- Throughput is defined as the rate at which data is transferred between two end systems.

- In a network, data flows through a series of links much like fluid flowing through a series of pipes. The end-to-end throughput is always constrained by the slowest link in the path, known as the **bottleneck link**.

- Once you determine the bottleneck link, you can calculate a rough approximation of how long a file transfer will take. (This approximation ignores external factors like processing delays, propagation delays, and protocol overhead). The time to transfer a file of $F$ bits is $F / \min\{R_s, R_c\}$.

- The **core of the Internet** is generally over-provisioned with massive, high-speed fiber links that experience very little congestion.  Because the core acts as an incredibly wide pipe, the constraining factor for a single connection is almost always the **access network** at the edges (either the server's $R_s$ or the client's $R_c$).

- Throughput is not dictated solely by the physical speed of the links; it is heavily impacted by other traffic actively sharing those links. A high-speed link can quickly become the bottleneck if it is shared by too many users. Imagine 10 clients downloading from 10 servers simultaneously. They all route their traffic through a single shared core link with a transmission rate ($R$) of $5$ Mbps. The router divides that $5$ Mbps link equally among the 10 active downloads, leaving only $500$ kbps per connection. Even if a client has a fast access link ($R_c = 1$ Mbps) and the server has a fast access link ($R_s = 2$ Mbps), their actual end-to-end throughput drops to $500$ kbps. The high-speed core link has become the new bottleneck due to the volume of intervening traffic.

---
### Security

The Internet was originally designed based on a model of mutually trusting users attached to a transparent network, meaning strict security was not fundamentally built-in. Because user identity was historically taken at face value, modern network protocol designers are now playing "catch-up" to implement security considerations across all layers of the network architecture.

Malware can infect connected devices to delete files or install spyware, which secretly collects and transmits private information like passwords and social security numbers back to attackers. Much of today's malware is self-replicating, allowing it to spread exponentially from one infected host to many others.

Attackers can secretly enroll compromised hosts into a massive network of infected devices, known as a botnet, which they then control remotely.

---
###  Denial of Service (DoS) & DDoS Attacks

DoS attacks attempt to make network resources (like servers or bandwidth) completely unavailable to legitimate users by overwhelming the target with bogus traffic.

There are three main categories of DoS attacks: 
- vulnerability attacks: crashing an application or OS with specific packets
- bandwidth flooding: clogging the target's access link
- connection flooding: bogging down a host with bogus half-open or fully open TCP connections.

In a Distributed DoS (DDoS) attack, attackers select a target and leverage their botnet to send a massive aggregate blast of packets from multiple compromised sources simultaneously. Because the malicious traffic originates from many different distributed hosts, DDoS attacks are significantly harder to detect and defend against than single-source DoS attacks.

---
### Packet Interception and Injection

- **Packet Sniffing:** Attackers can deploy passive receivers with promiscuous network interfaces in broadcast environments (like shared Ethernet or wireless networks) to read and record copies of all packets flying by, which often contain unencrypted sensitive data like passwords. Because packet sniffers operate entirely passively without injecting their own packets into the channel, their presence is extremely difficult to detect.

- **IP Spoofing:** Attackers can craft and inject packets containing a fake source address into the network to successfully masquerade as a trusted user or system.

---
### Network Defense

- **Authentication:** Mechanisms are required to prove the true identity of a user or system; while cellular networks use hardware like SIM cards for this, the traditional Internet lacks this built-in hardware assistance.

- **Confidentiality:** Network designers use encryption, such as cryptography, to ensure that even if data is sniffed, it remains completely unreadable to the attacker.

- **Integrity Checks:** Digital signatures are utilised to prevent tampering and detect if a packet's payload has been altered in transit.

- **Firewalls:** These are specialized middle-boxes placed in access and core networks that operate on a "off-by-default" basis, i.e., it assumes _all_ incoming traffic is malicious unless proven otherwise, actively filtering incoming packets to restrict specific senders, receivers, or applications, and detecting or reacting to DoS attacks.

---
### Application Layer

Network applications generally follow one of two structural models: **Client-server** or **Peer-to-peer (P2P)**.

#### Client-Server Architecture

In this model, the roles of devices are strictly separated into servers and clients.

- Server:
    - Acts as an **always-on host**.
    - Has a **permanent IP address** so clients can always find it.
    - Is often housed in massive data centers to allow for scaling when traffic increases.

- Clients:
    - Initiate contact and communicate exclusively with the server.
    - Do not communicate directly with other clients.
    - May only be intermittently connected to the network.
    - Often have dynamic (changing) IP addresses.

- Common Examples: HTTP (Web), IMAP (Email), and FTP (File Transfer).

#### Peer-to-Peer (P2P) Architecture

This model completely removes the central server, relying instead on direct communication between user devices.

- There is no  server managing the network.
- Arbitrary end systems (peers) communicate directly with one another.
- Peers request services from other peers, and in return, they provide services to others.
- **Self-Scalability**: When a new peer joins the network, they bring new service demands, but they also bring _new service capacity_ (like bandwidth and storage) to help support the network.
- Because peers are intermittently connected and constantly change IP addresses, managing the network is highly complex.
- Example: P2P file sharing (such as BitTorrent).

---
### Communicating Processes

- A process is defined as a program that is running within a host device.
- When two processes are running on the exact same host, they communicate locally using inter-process communication, which is defined and managed by the operating system.
- When processes are located on different hosts, they communicate over the network by exchanging messages.
- The client process is the one that initiates the communication, while the server process is the one that waits to be contacted.
- Even Peer-to-Peer (P2P) architectures utilize both client and server processes depending on which peer is initiating the request.

---
### Sockets

- A process sends and receives all of its messages to and from the network through its socket.
- A sending process shoves a message out its socket and relies completely on the transport infrastructure on the other side to deliver it to the receiving process.
- There are two sockets: one on the sending side and one on the receiving side.
- A socket is the interface between the application layer and the transport layer within a host. It is also referred to as the **Application Programming Interface (API)** between the application and the network.
- The application developer has control of everything on the application-layer side of the socket. The only control that the application developer has on the transport-layer side is the choice of transport protocol and perhaps the ability to fix a few transport-layer parameters such as maximum buffer and maximum segment sizes (to be covered in Chapter 3)

---
### Addressing Processes 

- To identify the receiving process, two pieces of information need to be specified: the address of the host and an identifier that specifies the receiving process in the destination host.
- In the Internet, every host device has a unique 32-bit IP address. A destination **port number** serves the other requirement.
- Popular applications have been assigned specific port numbers. For example, a Web server is identified by port number 80. A mail server process (using the SMTP protocol) is identified by port number 25.
- To send a message to a specific web server (like iitd.ac.in), the sender must address it to the correct IP address (e.g., 103.27.9.24) and the correct port number (such as 80 or 443).

---
### Transport Service Requirements

When you develop an application, you must choose one of the available transport-layer protocols. We can broadly classify the possible services along four dimensions:

- **Data Reliability:** Apps (like file transfers and web transactions) strictly require 100% reliable data transfer, while others (like streaming audio) can tolerate some data loss.

- **Throughput:** Apps (like multimedia) require a minimum guaranteed amount of throughput to be effective, whereas _elastic apps_ simply make use of whatever throughput they can get (like Email, File Transfer).

- **Timing:** Certain apps (like Internet telephony and interactive games) require extremely low delay to be effective. For non-real-time applications, lower delay is always preferable to higher delay, but no tight constraint is placed on the end-to-end delays.
- **Security:** Apps may require the network to provide encryption and data integrity.

---
###  TCP vs UDP 

The Internet provides two primary transport protocols to handle these application demands, each offering different services.

**TCP Service**:

- It guarantees reliable data delivery between the sending and receiving process int he correct order.
- It ensures the sender won't overwhelm the receiver with data.
- It automatically throttles the sender when the network is overloaded.
- It requires a strict setup phase between the client and server processes before any data can be exchanged.
- It does not guarantee timing, minimum throughput, or security.

**UDP Service**:

- It provides a lightweight service with no delivery or order guarantees between the sending and receiving process.
- It does not provide reliability, flow control, congestion control, timing, throughput guarantees, security, or connection setup.

---
### Mapping Applications to Protocols

This table illustrates which transport protocols and application-layer protocols standard internet applications choose to use.

| **Application**            | **Application-Layer Protocol**                 | **Transport Protocol** |
| -------------------------- | ---------------------------------------------- | ---------------------- |
| **File transfer/download** | FTP [RFC 959]                                  | TCP                    |
| **E-mail**                 | SMTP [RFC 5321]                                | TCP                    |
| **Web documents**          | HTTP [RFC 7230, 9110]                          | TCP                    |
| **Internet telephony**     | SIP [RFC 3261], RTP [RFC 3550], or proprietary | TCP or UDP             |
| **Streaming audio/video**  | HTTP [RFC 7230], DASH                          | TCP                    |
| **Interactive games**      | WOW, FPS (proprietary)                         | UDP or TCP             |

---
### Securing TCP

Standard TCP and UDP sockets provide absolutely **no encryption**. If a user types a password and the application sends it into a vanilla socket, that password traverses the entire global Internet in raw **cleartext**. Any packet sniffer on the path can easily read it.

Transport Layer Security (TLS): To fix this, networks use TLS. TLS enhances standard TCP by providing three critical security features:
    1. Encrypted TCP connections
    2. Data integrity (ensuring the data wasn't altered in transit)
    3. End-point authentication (proving the server is who it claims to be)

TLS is not a third Internet transport protocol, on the same level as TCP and UDP, but instead is an enhancement of TCP, with the enhancements being implemented in the application layer. 

- In particular, if an application wants to use the services of TLS, it needs to include TLS code (existing, highly optimized libraries and classes) in both the client and server sides of the application. 
- TLS has its own socket API that is similar to the traditional TCP socket API. When an application uses TLS, the sending process passes cleartext data to the TLS socket; TLS in the sending host then encrypts the data and passes the encrypted data to the TCP socket.
- The encrypted data travels over the Internet to the TCP socket in the receiving process. The receiving socket passes the encrypted data to TLS, which decrypts the data. Finally, TLS passes the cleartext data through its TLS socket to the receiving process.

---
### Final Comments on TCP/UDP

Throughput or timing guarantees are services not provided by today’s Internet transport protocols. But time-sensitive applications often work fairly well because they have been designed to cope, to the greatest extent possible, with this lack of guarantee. Nevertheless, clever design has its limitations when delay is excessive, or the end-to-end throughput is limited. In summary, today’s Internet can often provide satisfactory service to time-sensitive applications, but it cannot provide any timing or throughput guarantee.

Applications choose TCP primarily because TCP provides reliable data transfer but because Internet telephony applications (such as Skype) can often tolerate some loss but require a minimal rate to be effective, thryprefer to run their applications over UDP, thereby circumventing TCP’s congestion control mechanism and packet overheads. But because many firewalls are configured to block (most types of) UDP traffic, Internet telephony applications often are designed to use TCP as a backup if UDP communication fails.

---
### Application Layer Protocols

An application-layer protocol sets the exact rules for how processes on different hosts communicate. Specifically, it defines:

- **Types of messages exchanged:** Specifying whether a message is a request or a response.
- **Message syntax:** What fields are included in the messages and how these are structured.
- **Message semantics:** meaning of information in the fields.
- **Rules:** This determines when and how processes should send and respond to messages.

These protocols generally fall into two categories:

- **Open protocols:** These are defined in RFCs, meaning everyone has access to the protocol definition, which allows for broad interoperability (e.g., HTTP, SMTP).
- **Proprietary protocols:** These are privately owned and controlled by specific organisations (e.g., Zoom, Skype).

It is important to distinguish between network applications and application-layer protocols. An application-layer protocol is only one piece of a network application.

The Web is a client-server application that allows users to obtain documents from Web servers on demand. The Web application consists of many components, including a standard for document formats (that is, HTML), Web browsers (for example, Chrome and Microsoft Internet Explorer), Web servers (for example, Apache and Microsoft servers), and an application-layer protocol. The Web’s application-layer protocol, HTTP, defines the format and sequence of messages exchanged between browser and Web server. Thus, HTTP is only one piece of the Web application.

- A web page is composed of objects, such as HTML files, JPEG images, or audio files. These objects can be stored on different Web servers.
- A typical web page consists of a base HTML-file that includes several referenced objects.
- Each object is addressable by a URL, which consists of a host name and a path name.

---
### HTTP

- HTTP stands for **Hypertext Transfer Protocol** and serves as the Web's application-layer protocol. It operates on a client/server model. The client (e.g., a browser) requests, receives, and displays Web objects using the HTTP protocol. The server (e.g., a Web server) sends objects in response to these requests using the HTTP protocol.
- HTTP relies on TCP for its transport layer. The connection process involves the client initiating a TCP connection (creating a socket) to the server on port 80, the server accepting the connection, the exchange of application-layer HTTP messages, and finally, the closing of the TCP connection.
- HTTP is a "stateless" protocol, meaning the server maintains no information about past client requests. If a particular client asks for the same object twice in a period of a few seconds, the server does not respond by saying that it just served the object to the client; instead, the server resends the object, as it has completely forgotten what it did earlier.
- Maintaining state is actively avoided because protocols that do so are complex; they require maintaining past history and reconciling inconsistent views if a server or client crashes.

---
### Persistent and Non-Persistent Connections

#### Non-Persistent HTTP

- A TCP connection is opened, at most one object is sent over that connection, and then the TCP connection is closed.
- Downloading multiple objects requires opening multiple separate connections.
- For example, if a client requests an HTML file that references 10 JPEG images, the cycle of opening a TCP connection, requesting the object, receiving the object, and closing the connection is repeated for the HTML file and then again for each of the 10 images.
- Performance is measured using Round Trip Time (RTT), defined as the time it takes for a small packet to travel from the client to the server and back.
- Each object requires one RTT to initiate the TCP connection and one RTT for the HTTP request and the first few bytes of the response to return, in addition to the time it takes to transmit the full file. This makes the total non-persistent HTTP response time equal to 2RTT + file transmission time.
- The primary issues with this approach are the mandatory 2 RTTs per object, the OS overhead required for every single TCP connection, and the fact that browsers often have to open multiple parallel TCP connections to fetch objects efficiently.

![[Screenshot 2026-08-11 at 10.16.09 PM.png]]

(First three arrows are the 3-way handshake)

#### Persistent HTTP (HTTP 1.1)

- A TCP connection is opened to a server, multiple objects can be sent over that single connection, and then the connection is closed.
- The server intentionally leaves the connection open after sending a response, allowing subsequent HTTP messages between the same client and server to utilize the already open connection. The client is able to send requests as soon as it encounters a referenced object in the base file.
- This efficiency means it can take as little as one RTT for all referenced objects, effectively cutting the response time in half compared to the non-persistent method.

---
### HTTP Message Format

```HTTP
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

These messages are written in ordinary ASCII text, meaning they are formatted to be human-readable. While a request message can have a single line or many lines, every line must end with a carriage return and a line feed character.

The HTTP specifications outline two primary types of HTTP messages: request messages and response messages.

---
### Request Message 

An HTTP request message is divided into four main sections:

- **The Request Line:** This is always the first line of the message. It contains three specific fields separated by spaces: the method field, the URL field, and the HTTP version field.
- **The Header Lines:** Directly following the request line, these lines contain a header field name and a corresponding value.
- **The Blank Line:** A specific carriage return and line feed are used to create a blank line that explicitly indicates the end of the header lines.
- **The Entity Body:** This sits at the very bottom of the message. It remains empty for certain methods like GET, but holds user-generated data for methods like POST.

#### Method Field

The method field in the request line dictates what action the client wants the server to take.

- **GET Method:** While its entity body is empty, it can still send inputted form data directly within the requested URL itself.

- **POST Method:** This is typically used when a user fills out a web form (like a search engine query); the specific inputted data is sent from the client to the server inside the entity body.

- **HEAD Method:** This functions similarly to a GET request, but it asks the server to respond with only the HTTP headers and leave out the requested object itself. Application developers often use this for debugging.

- **PUT Method:** This is used by users or applications to upload a new file to a specific path on a Web server, or to completely replace an existing file at a specified URL.

- **DELETE Method:** This allows an application or user to delete an object residing on a Web server.

#### Headers

Header lines provide essential context and instructions to the receiving server.

- **Host:** Specifies the exact host where the requested object resides, which is explicitly required by Web proxy caches.

- **Connection:** Tells the server whether to maintain persistent connections or to close the connection after the object is sent (e.g., `Connection: close`).

- **User-agent:** Identifies the specific browser type making the request (like Mozilla Firefox), allowing the server to send customized versions of the same object if necessary.

- **Accept-language:** A content negotiation header that tells the server which language version of the object the user prefers to receive.

---
### Response Message

```HTTP
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html

(data data data data data ...)
```

An HTTP response message is divided into three distinct sections: an initial status line, header lines, and an entity body.

- **The Status Line:** This always appears as the first line in a server-to-client response message. It consists of three specific fields: the protocol version, a status code, and a corresponding status phrase.
- **The Header Lines:** Directly following the status line, these provide essential metadata about the server and the data being transmitted.
- **The Entity Body:** This is the core of the message, containing the actual data of the requested object.

#### Header Lines

The header lines provide instructions and context to the client receiving the message.

- **Connection:** A value such as `close` informs the client that the server plans to close the TCP connection immediately after sending the response message.

- **Date:** This represents the exact date and time the server created and sent the response message.

- **Server:** This field identifies the software that generated the message (such as an Apache Web server), which acts as the server-side equivalent to a client's `User-agent` header.

- **Last-Modified:** This indicates the date and time the requested object was either created or last modified. This specific header is critical for object caching by local clients and network proxy servers.

- **Content-Length:** This indicates the total size of the sent object in bytes.

- **Content-Type:** This header officially dictates the object type contained in the entity body (such as HTML text), which is used instead of relying on file extensions.

#### Status Codes

The status code and its associated phrase instantly indicate the result of the client's request.

- **200 OK:** The request succeeded, and the requested object is included later within the message.

- **301 Moved Permanently:** The requested object has been permanently moved to a new location. The response specifies the new URL in a `Location:` header, which client software will typically retrieve automatically.

- **400 Bad Request:** This is a generic error code indicating that the server could not understand the request message.

- **404 Not Found:** This signifies that the requested document does not currently exist on the server.

- **505 HTTP Version Not Supported:** This indicates that the server does not support the specific HTTP protocol version that was requested by the client.

---
### Cookies

While HTTP is inherently stateless, cookies  allow websites to track and identify users over time. This ability to maintain state across multiple transactions is heavily used for user authorisation, and maintaining user session state (such as staying logged into Web e-mail) or because the server wishes to restrict user access or because it wants to serve content as a function of the user identity. Cookie technology relies on four interacting components to function:

- A **`Set-cookie:` header line** generated by the server and included in the HTTP response message to assign a unique ID to the user.
- A **`Cookie:` header line** included by the browser in subsequent HTTP request messages to identify the user to the server.
- A **cookie file** kept locally on the user’s end system and actively managed by the user’s web browser.
- A **back-end database** located at the Web site that links the assigned identification number to the user's specific activity or account information.

![[Screenshot 2026-08-11 at 10.46.22 PM.png]]

When a user (e.g., Susan) visits a site like Amazon for the first time:

1. The Amazon server generates a unique identification number (e.g., 1678) for Susan. It sends back the requested web page with an HTTP response message that includes the header line: `Set-cookie: 1678`.

2. When Susan’s browser receives this response, it reads the `Set-cookie` header. The browser actively manages a special local cookie file on her computer. It appends a new line to this file, mapping the server's hostname (Amazon) to her new ID (1678). _(Note: This file stores entries for all sites that use cookies, like a prior entry for eBay)._

3. As Susan continues to browse Amazon and clicks on a new page, her browser automatically consults its local cookie file before sending the next HTTP request. The browser extracts the ID associated with Amazon's hostname and embeds it into the new HTTP request using the header line: `Cookie: 1678`.

4. By receiving this `Cookie:` header with every subsequent request, the Amazon server knows exactly which pages user 1678 is visiting, allowing it to maintain a shopping cart or provide one-click shopping, even without knowing Susan's actual name.

---
### First-Party vs. Third-Party Cookies

- **First-party cookies:** These track user behavior strictly on the specific website the user intentionally chose to visit (e.g., visiting NYTimes.com and receiving a cookie from NYTimes.com).

- **Third-party cookies:** These are generated by a website that the user did not choose to visit, such as an advertising network embedded within the host site. More dangerous is when there is a hidden link embedded instead of an advertisement.

- Because third-party cookies can track a user's common identity across many different websites, often invisibly, they have been disabled by default in browsers like Firefox and Safari, and in Chrome as of 2023.

This example illustrates what happens when multiple independent websites use the same third-party advertising network (say **AdX**).

A user decides to visit `nytimes.com` to read the sports section. 
The `nytimes.com` server creates a **first-party cookie** (ID: 1634) to remember the user for their own site's purposes. However, the NY Times webpage also contains an embedded advertisement provided by **AdX.com**.
To display the ad, the user's browser automatically sends an HTTP request to the AdX server. Because AdX has never seen this user before, it creates its own **third-party cookie** (ID: 7493) and sends it back to the user's browser. The AdX database now has a profile for user 7493, noting that they like reading NY Times sports.

A day later, the user browses to a completely unrelated website: `socks.com` to look at footwear.
Unknown to the user, `socks.com` _also_ uses the AdX network to display ads on its pages.
When the user's browser fetches the ad for this page, it sees that it is contacting AdX.com. The browser automatically attaches the existing AdX cookie (`Cookie: 7493`) to the request, along with the referrer URL (`socks.com`). The AdX server receives the request, sees ID 7493, and updates its database. AdX now secretly knows that the person who read NY Times sports yesterday is currently shopping for socks today, even though the user never directly clicked on an AdX website.

Later on, the user returns to `nytimes.com`, this time to read the arts section. The browser fetches the webpage and, once again, fetches the embedded AdX ad, sending along cookie ID 7493.  The AdX server checks its database for user 7493. It sees the recent browsing history indicating a strong interest in socks. Instead of returning a generic ad, AdX returns a highly targeted advertisement for socks, displaying it right there on the NY Times arts page.

---
### Privacy and GDPR Regulations

- Cookies are controversial because they can also be considered as an invasion of privacy. Using a combination of cookies and user-supplied account information, a Web site can learn a lot about a user and potentially sell this information to a third party.

- Under the EU General Data Protection Regulation (GDPR), when cookies can identify a natural person or be combined with other identifiers to create a profile, they are strictly considered personal data.

- Due to these data regulations, websites must provide users with explicit control over whether or not cookies are allowed to be placed on their devices.

---
### Web Caches

A Web cache, also known as a proxy server, is a network entity designed to satisfy HTTP requests on behalf of the origin Web server. 
The primary goal of a cache is to satisfy a client's request without having to involve the origin server directly. It maintains its own disk storage to keep local copies of recently requested objects.  A Web cache acts as both a server and a client.

![[Screenshot 2026-08-11 at 11.28.29 PM.png]]

A user configures their browser to direct all HTTP requests to the local Web cache first.
If the requested object is already stored in the cache, the cache immediately returns the object to the client within an HTTP response.
If the object is missing, the cache requests it from the origin server, stores a new copy in its local storage, and then forwards the object to the client. Derver tells cache about object’s allowable caching in response header.

```HTTP
Cache-Control: max-age=<seconds>
OR
Cache-Control: no cache
```

These caches are typically purchased and installed by ISPs, such as universities for their campus networks or residential ISPs for their subscribers.

#### Advantages

- Caching substantially reduces the response time for client requests because the cache is physically closer to the client than the distant origin server.
- It significantly reduces traffic on an institution's access link to the public Internet.
- By lowering the traffic intensity on access links, institutions can maintain fast network performance without having to pay for costly bandwidth upgrades.
- Widespread caching reduces Web traffic across the entire Internet, which improves overall performance for all applications globally.


**UNDERSTAND THE EXAMPLE GIVEN IN BOOK**

---
### Conditional GET

Caching introduces the risk of serving stale data if the origin server modifies the object after the cache originally stored it. The Conditional GET mechanism allows the browser or cache to verify if its stored object is still up-to-date without triggering an unnecessary object transmission delay. It is telling the server to send the object only if the object has been modified since the specified date.

To perform this check, the client sends an HTTP GET request that includes an **If-Modified-Since:** header, specifying the exact date of its cached copy. If the object has not changed on the server since that date, the server responds with an **HTTP/1.0 304 Not Modified** status code and an empty entity body.

---
### HOL Blocking

HTTP/1.1 utilises pipelined GET requests over a single persistent TCP connection. Because the server responds to these requests in a strict first-come-first-served order, it creates a vulnerability known as Head of Line (HOL) blocking.

 If a large object (like a heavy video file) is requested first, smaller objects get stuck waiting indefinitely behind it for transmission over a bottleneck link.

TCP congestion control (discussed later), also provides browsers an unintended incentive to use multiple parallel TCP connections rather than a single persistent connection. TCP congestion control aims to give each TCP connection sharing a bottleneck link an equal share of the available bandwidth of that link; so if there are n TCP connections operating over a bottleneck link, then each connection approximately gets 1/nth of the bandwidth. By opening multiple parallel TCP connections to transport a single Web page, the browser can cheat and grab a larger portion of the link bandwidth. Many HTTP/1.1 browsers open up to six parallel TCP connections not only to circumvent HOL blocking but also to obtain more bandwidth.

---
### HTTP/2

Standardized in 2015, HTTP/2 aims to drastically reduce user-perceived latency for multi-object requests while maintaining the exact same HTTP methods, status codes, URLs, and header fields used in HTTP/1.1.

One of the primary goals of HTTP/2 is to get rid of (or at least reduce the number of) parallel TCP connections for transporting a single Web page. This not only reduces the number of sockets that need to be open and maintained at servers, but also allows TCP congestion control to operate as intended **(this is weird, since I can still open multiple connects to hog bandwidth)**. 

- HTTP/2 solves HOL blocking by breaking messages down into smaller, binary-encoded frames. The server can interleave the frame transmissions of multiple different objects over a single TCP connection, allowing small objects to be delivered quickly alongside large ones. The framing is done by the framing sub-layer of the HTTP/2 protocol. As the frames arrive at the client, they are first reassembled into the original response messages at the framing sub-layer and then processed by the browser as usual. 

- In addition to breaking down each HTTP message into independent frames, the framing sublayer also binary encodes the frames. Binary protocols are more efficient to parse, lead to slightly smaller frames, and are less error-prone.

- Clients can optimize their application performance by assigning a priority weight (from 1 to 256) and dependency IDs to their requests, dictating the transmission order rather than relying on a strict FCFS queue.

- Another feature of HTTP/2 is the ability for a server to send multiple responses for a single client request. Instead of sitting idle waiting for explicit requests, the server can analyze an HTML base page and push necessary, unrequested objects to the client to save time.

---
### Evolution to HTTP/3

While HTTP/2 vastly improved framing, its reliance on a single TCP connection means that if a packet is lost, the underlying TCP loss recovery process still stalls the transmission of _all_ objects in the stream **(GOTTA LOOK INTO THE MECHANISM TO VERIFY THIS)**. Because of this stalling risk and the lack of built-in security over a vanilla TCP connection, browsers still have an incentive to open multiple parallel connections to maximise throughput.

HTTP/3 is a newer protocol designed to completely replace TCP by operating over QUIC, which is built on the UDP protocol.

HTTP/3 adds built-in security and provides per-object error and congestion control, preventing a single packet loss from stalling the entire connection and allowing for a much simpler, streamlined design.

---
### File Transfer Protocols (FTP)

FTP is used specifically for transferring files between a client and a server.
It uses yhe TCP transport protocol
It supports a robust set of file and directory operations, including uploading, downloading, renaming, and deleting files, as well as creating and listing directories.
It also includes built-in support for resuming interrupted transfers and authenticating users.

A unique characteristic of FTP is that it uses **two separate connections** to operate:

- **Control Connection:** This operates on port 21 and remains persistent throughout the entire session to handle commands and responses.
- **Data Connection:** This uses a dynamic port and is opened and closed separately for _each individual data transfer_.

To facilitate these connections, FTP operates in two distinct modes:
- **Active Mode:** The client asks the server to connect back to it.
- **Passive Mode:** The server asks the client to connect back to it.

To find the actual port number, the receiving machine applies a specific formula using those last two 8-bit numbers: $(p1 \times 256) + p2 = \text{Actual Port Number}$

Example: `192,168,1,5,19,136`:
- The IP address is **192.168.1.5**
- The port calculation is $(19 \times 256) + 136$
- The resulting port is **5000**

Breaking the address and port down into these strict 8-bit chunks (bytes) was much more efficient for early network machines to parse and process at the hardware level.

Why modern FTP doesn't just adopt a simpler, updated format ?
Internet protocols almost never change once they have been widely deployed.

---
### Telnet 

Telnet is an application-layer protocol designed to let a user log in and interact with a remote computer/  
It operates using TCP on **port 23**.
It is highly interactive, and a single session might be left open for hours at a time.
Telnet was eventually discontinued and phased out of modern use due to its extremely weak security protection (it sends commands and passwords over the network in plain, unencrypted text).

---
### SSH (Secure Shell)

SSH was developed to replace insecure protocols like Telnet.
Unlike Telnet, SSH establishes a **secure, encrypted channel** between the client and the server _before_ any commands or passwords are ever sent across the network.
It operates on **port 22**.

---
