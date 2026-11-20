# Multi-Area OSPF Network Configuration Using Cisco Packet Tracer

## 1. Project Overview

This project demonstrates the design and configuration of a **Multi-Area Open Shortest Path First (OSPFv2)** network using Cisco Packet Tracer.

The network consists of six interconnected Cisco routers organized into three OSPF areas:

- **Area 0:** Backbone Area
- **Area 10:** Contains Router R5 and four Loopback networks
- **Area 20:** Contains Router R6

The primary objective is to implement dynamic routing across multiple OSPF areas and enable communication between different networks through Area Border Routers (ABRs).

The project also demonstrates how to configure and advertise multiple Loopback interfaces, configure OSPF Router IDs, and verify inter-area routing.

### Learning Objectives

- Design a Multi-Area OSPF network.
- Configure IPv4 addressing on router interfaces.
- Understand OSPF backbone and non-backbone areas.
- Configure OSPF routing across six routers.
- Identify and configure Area Border Routers.
- Configure multiple Loopback interfaces.
- Advertise Loopback subnets using OSPF.
- Understand OSPF Wildcard Masks.
- Verify OSPF neighbor relationships.
- Understand intra-area and inter-area routing.
- Examine OSPF routing tables.
- Troubleshoot OSPF configuration and connectivity issues.

---

## 2. Network Topology

The topology consists of six routers divided into three OSPF areas.

![Multi-Area OSPF Topology](images/multi-area-ospf-topology.png)

*Figure 1: Multi-Area OSPF network topology in Cisco Packet Tracer.*

### Topology Structure

```text
                       AREA 0
                    (Backbone)

                   R4
                    |
              10.10.14.0/24
                    |
                   R1
                  /  \
                 /    \
      10.10.12.0/24  10.10.13.0/24
              /          \
             R2           R3
             |            |
        AREA 10         AREA 20
             |            |
      10.10.25.0/24  10.10.35.0/24
             |            |
             R5           R6
             |
       Four Loopbacks
       192.168.0.0/24
       192.168.1.0/24
       192.168.2.0/24
       192.168.3.0/24
```

### OSPF Area Design

| Area | Routers | Purpose |
|---|---|---|
| Area 0 | R1, R2, R3, R4 | Backbone routing |
| Area 10 | R2, R5 | Loopback network connectivity |
| Area 20 | R3, R6 | Additional network segment |

**R2 and R3 function as Area Border Routers (ABRs).**

R2 connects Area 0 to Area 10, while R3 connects Area 0 to Area 20.

All inter-area communication is routed through the OSPF backbone.

---

## 3. IP Addressing Scheme

### Area 0 – Backbone

| Connection | Network | Router Interface | IP Address |
|---|---|---|---|
| R1–R2 | 10.10.12.0/24 | R1 Gi0/0 | 10.10.12.1 |
| | | R2 Gi0/0 | 10.10.12.2 |
| R1–R3 | 10.10.13.0/24 | R1 Gi0/2 | 10.10.13.1 |
| | | R3 Gi0/2 | 10.10.13.2 |
| R1–R4 | 10.10.14.0/24 | R1 Gi0/1 | 10.10.14.1 |
| | | R4 Gi0/1 | 10.10.14.2 |

### Area 10

| Connection | Network | Router Interface | IP Address |
|---|---|---|---|
| R2–R5 | 10.10.25.0/24 | R2 Gi0/1 | 10.10.25.2 |
| | | R5 Gi0/1 | 10.10.25.1 |

### Area 20

| Connection | Network | Router Interface | IP Address |
|---|---|---|---|
| R3–R6 | 10.10.35.0/24 | R3 Gi0/1 | 10.10.35.2 |
| | | R6 Gi0/1 | 10.10.35.1 |

### Loopback Networks on R5

| Interface | IP Address | Network | OSPF Area |
|---|---|---|---|
| Loopback0 | 192.168.0.1/24 | 192.168.0.0/24 | Area 10 |
| Loopback1 | 192.168.1.1/24 | 192.168.1.0/24 | Area 10 |
| Loopback2 | 192.168.2.1/24 | 192.168.2.0/24 | Area 10 |
| Loopback3 | 192.168.3.1/24 | 192.168.3.0/24 | Area 10 |

All physical links use the subnet mask `255.255.255.0`.

---

## 4. OSPF Fundamentals

OSPF is a link-state Interior Gateway Protocol (IGP) used to exchange routing information within an autonomous system.

OSPF calculates routes using the Shortest Path First (SPF) algorithm, also known as Dijkstra's algorithm.

### Key OSPF Characteristics

| Property | Description |
|---|---|
| Protocol | OSPFv2 |
| Type | Link-State |
| Algorithm | Dijkstra SPF |
| Administrative Distance | 110 |
| Metric | Cost |
| Backbone Area | Area 0 |
| Routing Process | OSPF 1 |
| Design | Multi-Area |

### Why Use Multiple OSPF Areas?

Dividing an OSPF network into multiple areas provides several advantages:

1. Reduces the scope of link-state flooding.
2. Limits topology changes to individual areas where possible.
3. Reduces the size of individual area link-state databases.
4. Improves scalability in large enterprise networks.
5. Allows inter-area route summarization at ABRs.
6. Supports hierarchical routing design.

In this project, Area 0 serves as the central backbone connecting Area 10 and Area 20.

---

## 5. Router Configuration

The following configurations reproduce the IP addressing and OSPF area design shown in the topology.

All interfaces must be enabled using `no shutdown`.

### 5.1 Router R1 – Backbone Router

R1 connects to R2, R3, and R4 within Area 0.

```cisco
enable
configure terminal

hostname R1

interface gigabitEthernet 0/0
 ip address 10.10.12.1 255.255.255.0
 no shutdown
exit

interface gigabitEthernet 0/1
 ip address 10.10.14.1 255.255.255.0
 no shutdown
exit

interface gigabitEthernet 0/2
 ip address 10.10.13.1 255.255.255.0
 no shutdown
exit

router ospf 1
 router-id 1.1.1.1
 network 10.10.12.0 0.0.0.255 area 0
 network 10.10.13.0 0.0.0.255 area 0
 network 10.10.14.0 0.0.0.255 area 0
exit

end
copy running-config startup-config
```

R1 operates entirely within the backbone area.

### 5.2 Router R2 – Area Border Router (Area 0 and Area 10)

R2 connects the backbone network to Area 10.

```cisco
enable
configure terminal

hostname R2

interface gigabitEthernet 0/0
 ip address 10.10.12.2 255.255.255.0
 no shutdown
exit

interface gigabitEthernet 0/1
 ip address 10.10.25.2 255.255.255.0
 no shutdown
exit

router ospf 1
 router-id 2.2.2.2
 network 10.10.12.0 0.0.0.255 area 0
 network 10.10.25.0 0.0.0.255 area 10
exit

end
copy running-config startup-config
```

R2 is an **Area Border Router (ABR)** because it has interfaces participating in two different OSPF areas.

It exchanges routing information between Area 10 and Area 0.

### 5.3 Router R3 – Area Border Router (Area 0 and Area 20)

R3 connects Area 0 to Area 20.

```cisco
enable
configure terminal

hostname R3

interface gigabitEthernet 0/2
 ip address 10.10.13.2 255.255.255.0
 no shutdown
exit

interface gigabitEthernet 0/1
 ip address 10.10.35.2 255.255.255.0
 no shutdown
exit

router ospf 1
 router-id 3.3.3.3
 network 10.10.13.0 0.0.0.255 area 0
 network 10.10.35.0 0.0.0.255 area 20
exit

end
copy running-config startup-config
```

R3 distributes routing information between Area 20 and the backbone.

### 5.4 Router R4 – Backbone Router

R4 belongs exclusively to Area 0.

```cisco
enable
configure terminal

hostname R4

interface gigabitEthernet 0/1
 ip address 10.10.14.2 255.255.255.0
 no shutdown
exit

router ospf 1
 router-id 4.4.4.4
 network 10.10.14.0 0.0.0.255 area 0
exit

end
copy running-config startup-config
```

### 5.5 Router R5 – Area 10 Router with Multiple Loopbacks

R5 connects to R2 and contains four Loopback networks.

```cisco
enable
configure terminal

hostname R5

interface gigabitEthernet 0/1
 ip address 10.10.25.1 255.255.255.0
 no shutdown
exit

interface loopback 0
 ip address 192.168.0.1 255.255.255.0
 ip ospf network point-to-point
exit

interface loopback 1
 ip address 192.168.1.1 255.255.255.0
 ip ospf network point-to-point
exit

interface loopback 2
 ip address 192.168.2.1 255.255.255.0
 ip ospf network point-to-point
exit

interface loopback 3
 ip address 192.168.3.1 255.255.255.0
 ip ospf network point-to-point
exit

router ospf 1
 router-id 5.5.5.5
 network 10.10.25.0 0.0.0.255 area 10
 network 192.168.0.0 0.0.3.255 area 10
exit

end
copy running-config startup-config
```

### Why Use `0.0.3.255`?

Instead of entering four separate OSPF network statements, one wildcard mask matches all four Loopback subnet ranges.

```cisco
network 192.168.0.0 0.0.3.255 area 10
```

This matches addresses from:

```text
192.168.0.0
to
192.168.3.255
```

The statement enables OSPF on matching interfaces; it does not itself summarize four routes into one.

Because the Loopback interfaces use `/24` masks and have the OSPF point-to-point network type, the four networks can be advertised individually as `/24` routes.

### 5.6 Router R6 – Area 20 Router

R6 is connected to R3 in Area 20.

```cisco
enable
configure terminal

hostname R6

interface gigabitEthernet 0/1
 ip address 10.10.35.1 255.255.255.0
 no shutdown
exit

router ospf 1
 router-id 6.6.6.6
 network 10.10.35.0 0.0.0.255 area 20
exit

end
copy running-config startup-config
```

---

## 6. Understanding Area Border Routers (ABRs)

An Area Border Router connects the backbone to one or more other OSPF areas.

In this topology:

**R2 connects:**

```text
Area 0 ↔ R2 ↔ Area 10
```

**R3 connects:**

```text
Area 0 ↔ R3 ↔ Area 20
```

### Example: Communication Between R5 and R6

When R6 needs to reach a Loopback network on R5, traffic can follow:

```text
R6 → R3 → R1 → R2 → R5
```

The routing information flows through the backbone Area 0.

OSPF uses the routing topology and interface costs to determine the preferred path.

### Intra-Area vs Inter-Area Routes

| Route Type | Routing Table Code | Description |
|---|---|---|
| Intra-Area | O | Learned within the same OSPF area |
| Inter-Area | O IA | Learned from another OSPF area |

For example, R6 should learn the four Loopback networks from R5 as inter-area routes.

Expected routing table entries on R6 may look similar to:

```text
O IA 192.168.0.0/24 via 10.10.35.2
O IA 192.168.1.0/24 via 10.10.35.2
O IA 192.168.2.0/24 via 10.10.35.2
O IA 192.168.3.0/24 via 10.10.35.2
```

The exact metric values depend on the interface costs.

---

## 7. Loopback Interface Configuration

Four Loopback interfaces are configured on R5.

Loopback interfaces are logical interfaces that remain operational independently of physical Ethernet cables.

They are useful for simulating internal networks and testing routing reachability.

### Configured Loopback Networks

```text
Loopback0 → 192.168.0.0/24
Loopback1 → 192.168.1.0/24
Loopback2 → 192.168.2.0/24
Loopback3 → 192.168.3.0/24
```

### Default OSPF Behavior

By default, Cisco OSPF commonly advertises Loopback interfaces as `/32` host routes.

For example:

```text
O 192.168.0.1/32
```

However, in this project the objective is to advertise complete `/24` networks.

This is achieved using:

```cisco
interface loopback 0
 ip ospf network point-to-point
```

The same command is applied to each Loopback interface.

With this configuration, the networks can be advertised using their configured subnet masks.

### Verifying Loopback Interfaces

Run on R5:

```cisco
show ip interface brief
```

Expected interface information:

```text
Loopback0   192.168.0.1   up   up
Loopback1   192.168.1.1   up   up
Loopback2   192.168.2.1   up   up
Loopback3   192.168.3.1   up   up
```

---

## 8. OSPF Router ID Configuration

Each router requires a unique Router ID within the OSPF routing domain.

| Router | Router ID |
|---|---|
| R1 | 1.1.1.1 |
| R2 | 2.2.2.2 |
| R3 | 3.3.3.3 |
| R4 | 4.4.4.4 |
| R5 | 5.5.5.5 |
| R6 | 6.6.6.6 |

Example:

```cisco
router ospf 1
 router-id 5.5.5.5
```

If OSPF is already running when the Router ID is changed, restart the OSPF process:

```cisco
clear ip ospf process
```

Confirm the reset when prompted.

This temporarily resets OSPF adjacencies.

### Verification

```cisco
show ip ospf
```

Expected on R5:

```text
Routing Process "ospf 1" with ID 5.5.5.5
```

---

## 9. OSPF Neighbor Verification

OSPF routers must establish adjacency before exchanging routing information.

Use:

```cisco
show ip ospf neighbor
```

### Expected Neighbor Relationships

| Router | Expected Neighbor(s) |
|---|---|
| R1 | R2, R3, R4 |
| R2 | R1, R5 |
| R3 | R1, R6 |
| R4 | R1 |
| R5 | R2 |
| R6 | R3 |

The expected OSPF state after successful adjacency formation is generally `FULL`.

Example expected output on R5:

```text
Neighbor ID    State      Address       Interface

2.2.2.2        FULL/DR    10.10.25.2    GigabitEthernet0/1
```

The DR/BDR role shown can vary depending on the election process.

### Common OSPF Neighbor States

| State | Meaning |
|---|---|
| DOWN | No Hello packets recently received |
| INIT | Hello packets received |
| 2-WAY | Bidirectional communication established |
| EXSTART | Database exchange negotiation |
| EXCHANGE | Database descriptions exchanged |
| LOADING | Missing link-state information requested |
| FULL | Adjacency established |

---

## 10. Routing Table Verification

To display the OSPF routing table:

```cisco
show ip route ospf
```

### Expected Routes on R6

If multi-area routing is working correctly, R6 should learn networks from both the backbone and Area 10.

Examples:

```text
O IA 10.10.12.0/24
O IA 10.10.13.0/24
O IA 10.10.14.0/24
O IA 10.10.25.0/24

O IA 192.168.0.0/24
O IA 192.168.1.0/24
O IA 192.168.2.0/24
O IA 192.168.3.0/24
```

Actual output depends on the configured routes, current adjacency states, and OSPF calculations.

### Understanding OSPF Route Codes

**O**

Intra-area OSPF route.

**O IA**

Inter-area OSPF route learned through an ABR.

### Administrative Distance

OSPF uses an Administrative Distance of 110 by default.

Example:

```text
O IA 192.168.1.0 [110/4] via 10.10.35.2
```

Here:

- `110` = Administrative Distance
- `4` = Example OSPF Cost
- `10.10.35.2` = Next-hop router

The cost shown is illustrative and may differ in the actual Packet Tracer topology.

---

## 11. Connectivity Testing

ICMP Ping is used to verify reachability between directly connected and remote networks.

### Test 1 – R5 to R2

```cisco
ping 10.10.25.2
```

This verifies connectivity within Area 10.

### Test 2 – R6 to R3

```cisco
ping 10.10.35.2
```

This verifies connectivity within Area 20.

### Test 3 – R5 to R1

```cisco
ping 10.10.12.1
```

This verifies routing from Area 10 toward the backbone.

### Test 4 – R6 to R5 Loopback Networks

On R6:

```cisco
ping 192.168.0.1
ping 192.168.1.1
ping 192.168.2.1
ping 192.168.3.1
```

These tests verify inter-area routing between Area 20 and Area 10.

### Test 5 – Traceroute

On R6:

```cisco
traceroute 192.168.0.1
```

The expected routed path is:

```text
R6 → R3 → R1 → R2 → R5
```

This helps visualize the path traffic takes through the OSPF backbone.

### Successful Ping Example

```text
!!!!!
Success rate is 100 percent (5/5)
```

These tests should be performed and their actual results recorded before the project is considered fully verified.

---

## 12. Troubleshooting

### Problem 1 – No OSPF Neighbor Appears

Command:

```cisco
show ip ospf neighbor
```

If no neighbors appear, possible causes include:

- Incorrect IP address or subnet mask.
- Incorrect OSPF network statement.
- Wrong OSPF Area.
- Disabled physical interface.
- Duplicate OSPF Router IDs.
- Hello/Dead timer mismatches.
- Authentication or network type mismatches.

### Problem 2 – Incorrect Wildcard Mask

An incorrect OSPF network statement might look like:

```cisco
network 10.0.0.0 255.255.255.0 area 10
```

This incorrectly uses a subnet-mask-style value as the wildcard mask and does not select the intended `10.10.25.0/24` interface correctly.

The correct statement for the R2–R5 connection is:

```cisco
network 10.10.25.0 0.0.0.255 area 10
```

### Problem 3 – Loopback Interface Range Command Rejected

Attempted:

```cisco
interface range loopback 0-3
```

In the Packet Tracer IOS environment used for this lab, the command was rejected.

**Solution:** Configure Loopback interfaces individually.

```cisco
interface loopback 0
exit

interface loopback 1
exit

interface loopback 2
exit

interface loopback 3
exit
```

### Problem 4 – Loopback Advertised as /32

By default, OSPF may advertise Loopback interfaces as host routes.

Solution:

```cisco
interface loopback 0
 ip ospf network point-to-point
```

Repeat the configuration for all Loopback interfaces.

### Problem 5 – Inter-Area Routes Missing

Verify that R2 and R3 connect their respective non-backbone areas to Area 0.

On R2:

```cisco
router ospf 1
 network 10.10.12.0 0.0.0.255 area 0
 network 10.10.25.0 0.0.0.255 area 10
```

On R3:

```cisco
router ospf 1
 network 10.10.13.0 0.0.0.255 area 0
 network 10.10.35.0 0.0.0.255 area 20
```

Then verify:

```cisco
show ip ospf neighbor
show ip route ospf
```

### Problem 6 – Interfaces Down

Check:

```cisco
show ip interface brief
```

Enable the correct interface if administratively disabled:

```cisco
configure terminal
interface gigabitEthernet 0/1
no shutdown
end
```

Also inspect the cable and remote interface.

---

## 13. Useful Verification Commands

| Command | Purpose |
|---|---|
| `show ip interface brief` | Display IP addresses and interface status |
| `show ip ospf` | Display OSPF process information |
| `show ip ospf neighbor` | Check OSPF neighbor relationships |
| `show ip ospf interface brief` | Check OSPF-enabled interfaces and areas |
| `show ip route` | Display complete routing table |
| `show ip route ospf` | Display OSPF routes |
| `show ip protocols` | Display routing protocol settings |
| `show running-config` | Inspect active configuration |
| `show ip ospf database` | Inspect the OSPF link-state database |
| `ping` | Test IP connectivity |
| `traceroute` | Trace packet forwarding path |
| `copy running-config startup-config` | Save the router configuration |

---

## 14. Single-Area vs Multi-Area OSPF

This project extends the concept of a basic Single-Area OSPF lab.

| Feature | Single-Area OSPF | Multi-Area OSPF |
|---|---|---|
| Number of Areas | One | Multiple |
| Backbone | Area 0 | Area 0 |
| ABRs | Not required | Required between areas |
| Routing Table | Intra-area routes | Intra-area and inter-area routes |
| Scalability | Suitable for smaller designs | Better suited for larger designs |
| LSDB Organization | One area | Separate LSDB per participating area |
| Route Summarization | Limited area-boundary options | Possible at ABRs |

The main difference is that Multi-Area OSPF introduces a hierarchical routing architecture.

This allows each area to maintain its own topology information while exchanging inter-area reachability through ABRs.

---

## 15. Project Verification Checklist

Use this checklist to verify the complete lab:

- [ ] All required interfaces have the correct IPv4 addresses.
- [ ] All physical links are operational.
- [ ] R1, R2, R3 and R4 participate in Area 0.
- [ ] R2 and R5 establish an OSPF adjacency in Area 10.
- [ ] R3 and R6 establish an OSPF adjacency in Area 20.
- [ ] Every router has a unique OSPF Router ID.
- [ ] R5 has four operational Loopback interfaces.
- [ ] All four Loopback networks are advertised as `/24`.
- [ ] R2 learns the four Loopback networks as intra-area routes.
- [ ] R1, R3, R4 and R6 learn the Loopback networks as inter-area routes.
- [ ] Ping from R6 to all four R5 Loopback IPs succeeds.
- [ ] Traceroute confirms the expected route through Area 0.
- [ ] Router configurations are saved.

---

## 16. Tools and Technologies

**Network Simulator**

Cisco Packet Tracer

**Routing Protocol**

OSPFv2 (Open Shortest Path First)

**Networking Concepts**

- IPv4 Addressing
- OSPF Multi-Area Design
- Backbone Area 0
- Area Border Routers
- Link-State Routing
- OSPF Router IDs
- Loopback Interfaces
- Wildcard Masks
- Intra-Area Routing
- Inter-Area Routing
- OSPF Cost
- Administrative Distance
- ICMP Ping
- Traceroute
- Network Troubleshooting

---

## 17. Repository Structure

Suggested GitHub repository structure:

```text
Multi-Area-OSPF-Cisco-Packet-Tracer/
│
├── README.md
│
├── multi-area-ospf.pkt
│
└── images/
    └── multi-area-ospf-topology.png
```

**Files**

`README.md` — Project documentation, configuration commands, and verification procedures.

`multi-area-ospf.pkt` — Cisco Packet Tracer simulation file.

`images/multi-area-ospf-topology.png` — Screenshot of the network topology.

---

## 18. Conclusion

This project demonstrates the implementation of a Multi-Area OSPF network using six Cisco routers distributed across three areas.

The design introduces hierarchical routing through Area Border Routers and demonstrates how OSPF enables communication between separate network areas.

A key part of the project is the configuration of four Loopback networks on R5 and their advertisement through Area 10 to the OSPF backbone and Area 20.

The lab also provides practical experience with OSPF network statements, wildcard masks, Router IDs, routing tables, interface configuration, and troubleshooting.

Overall, the project strengthens hands-on knowledge of dynamic routing and provides a foundation for understanding scalable enterprise network architecture.

---

**Level:** CCNA / Intermediate Networking Lab
