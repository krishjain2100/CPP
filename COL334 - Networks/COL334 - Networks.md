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

- **Firewalls:** These are specialized middleboxes placed in access and core networks that operate on a "off-by-default" basis, i.e., it assumes _all_ incoming traffic is malicious unless proven otherwise, actively filtering incoming packets to restrict specific senders, receivers, or applications, and detecting or reacting to DoS attacks.

---
