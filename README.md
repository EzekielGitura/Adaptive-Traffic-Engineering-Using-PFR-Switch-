# Breakdown of Project 

This code is designed for **OpenFlow-based Software-Defined Networking (SDN)** switches using the POX controller. It extends a basic Layer 2 learning switch (`SimpleL2LearningSwitch`) with advanced capabilities, primarily focusing on traffic handling and dynamic flow rules. This implementation can make routing decisions based on bandwidth, delay, and traffic type (TCP/UDP).

The main purpose is to create a **Policy-Based Forwarding (PFR) Switch** capable of:

1. **Dynamic Traffic Redirection**: Redirecting traffic based on bandwidth, delay thresholds, and specific conditions.
2. **Traffic Differentiation**: Applying different flow rules based on TCP/UDP traffic or specific ports.
3. **Load Balancing**: Using two paths (`p1` and `p2`) with bandwidth and delay measurements to optimize forwarding.
4. **Flow Management**: Dynamically managing OpenFlow rules with hard timeouts and idle timeouts.

---

### Key Components

1. **POX Modules Imported**:
   - `pox.core` - Core POX controller functionality.
   - `pox.openflow` - OpenFlow message handling.
   - `pox.lib.packet.*` - Packet parsing libraries for Ethernet, ARP, TCP, UDP, and IP.
   - `pox.lib.addresses` - Utilities for IP and MAC address handling.

2. **Custom Switch Class**:
   - **`PfrSwitch`**: Inherits from `SimpleL2LearningSwitch` and adds custom flow handling logic.
   - Implements `_handle_PacketIn` to process packets dynamically based on traffic conditions.

3. **Dynamic Routing Logic**:
   - Based on metrics like:
     - Bandwidth (`p1_total_bw`, `p2_total_bw`)
     - Delay (`p1_delay`, `p2_delay`)
     - Threshold (`delay_threshold`)
   - Differentiates traffic as:
     - **TCP Traffic** (handled by `_handle_PacketInTCP`)
     - **UDP Traffic** (handled by `_handle_PacketInUDP`)

4. **Flow Rules**:
   - Flow actions include modifying:
     - MAC Destination (`OFPAT_SET_DL_DST`)
     - IP Destination (`OFPAT_SET_NW_DST`)
     - Transport Port (`OFPAT_SET_TP_DST`)
   - Rules are dynamically installed using OpenFlow messages with timeouts:
     - `FLOW_HARD_TIMEOUT` - Lifetime of the flow rule.
     - `FLOW_IDLE_TIMEOUT` - Rule expires if no traffic matches.

---

### Use Cases

1. **Dynamic Load Balancing**:
   - Traffic can be rerouted between two servers (`server_ip1` and `server_ip2`) based on bandwidth and delay metrics.
   - Useful in scenarios where equal-cost paths exist but have varying performance.

2. **Traffic Engineering**:
   - Handle specific types of traffic (RTP, FTP, Web) differently:
     - **Web Traffic** (ports 80, 8080) is load-balanced.
     - **RTP Traffic** (UDP port 5001) can be routed to a backup path based on delay thresholds.

3. **Backup Routing**:
   - Automatically switch to an alternative path if one path exceeds delay thresholds or bandwidth limits.
   - Ensures traffic continuity in the event of congestion or failure.

4. **Optimized Multimedia Traffic**:
   - Real-Time Protocol (RTP) traffic is treated with priority by routing it to the path with better delay performance.

5. **Resilient Network Management**:
   - Provides dynamic failover between links to ensure reliability and redundancy.

6. **TCP Traffic Management**:
   - Redirects TCP traffic dynamically based on port numbers, such as FTP, HTTP, or transient traffic.

---

### Summary of Key Functions

- **`_handle_PacketIn`**:
  - Entry point for incoming packets.
  - Routes packets to appropriate TCP or UDP handlers.

- **`_handle_PacketInTCP`**:
  - Handles TCP-specific traffic with rules based on port numbers and metrics.

- **`_handle_PacketInUDP`**:
  - Handles UDP-specific traffic, prioritizing RTP and backup routing as necessary.

- **`createOFAction`** and **`createFlowMod`**:
  - Utility functions for creating OpenFlow actions and messages to modify flows.

---

### Real-World Scenarios

1. **Enterprise Networks**:
   - Dynamically load balance application traffic (e.g., HTTP) across servers based on real-time link performance.

2. **Media Streaming Services**:
   - Optimize RTP/UDP traffic to ensure low-latency video and audio streaming.

3. **Data Centers**:
   - Implement efficient failover mechanisms between primary and backup servers.

4. **Telecommunications**:
   - Route multimedia traffic dynamically over alternative links with better delay and bandwidth metrics.

---
### Conclusion 
This implementation showcases an advanced **OpenFlow-based SDN controller** capable of traffic differentiation, dynamic flow management, and policy-based routing. It demonstrates how SDN controllers can adapt to network conditions to optimize performance and reliability.
