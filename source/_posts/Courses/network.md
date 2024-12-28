---
title: Cornell CS ⅘456 Network
categories:
  - Course
  - CS5456-Network
---

## How is the network shared

- consist of End hosts & switches & links
- switches for sharing
  - pros
    - support many clients at the same time
    - Send and receive information
    - limited resources
- reservation (circuit switching)
  - Pros:
    - Predictable performance
    - Reserved bandwidth leads to better application performance
    - Reliable delivery (assuming no failures)
    - Simple forwarding mechanism
  - Cons:
    - Resource underutilization
    - Blocked connections
    - Connection set up overheads
    - Per-connection state in switches (scalability problem)
  - sharing at connection level
- on-demand (packet switching)
  - Pros
    - Better resource utilization
    - Better failure recovery
    - Quicker start to transfers
  - Cons:
    - Unpredictable bandwidth availability
    - Unpredictable delay/latency
    - Packet header overhead
  - sharing at packet level
- Underlying principle: statistical multiplexing
  - `combining demands to share resources (packets / connections) efficiently` 只要能满足这个的都算
  - Sharing resources using the characteristics of the user demands
  - Similar to insurance, with the same failure mode
  - bursty traffic
    - Peak of aggregate load $<<$ aggregate of peak load
    - Peak usage of a user $>>$ average usage of a user

## Network Performance

- throughput (bandwidth)
  - data size / transfer time = bits per second, bps
- delay / latency
  - Time for all bits to go from source to destination (seconds)
  - Transmission delay (hardware properties)
    - packet size / link bandwidth = bits / bps
  - Propagation delay (hardware properties, distance)
    - link length / speed of light
  - Queuing delay (traffic characteristics, switch internals)
  - Processing delay (switches and end hosts)
  - Bandwidth Delay Product (BDP)
    - Propagation delay x bandwidth
- loss, resilience to failure

## How does internet work

- End-to-end view of tasks
  1.  Naming, addressing: locating the destination
  2.  Routing: finding the path to the destination
  3.  Forwarding: sending data to the destination
      1. Queuing packets at switches in input queues
      2. Forwarding on the packet to the output queue
  4.  Reliability: communicating despite failures
       When packets lost
      - Dropped on switches
      - Failure of switches, links
      - Bit flips, errors
- protocols
  - agreement between peer layers on different nodes
  - syntax
    - Payload —> data
    - Header —> how to process the payload
- Pipeline
  1.  Bits arrive on the wire —> (L1)
  2.  Packets need to be forwarded to next router/switch —> (L2)
  3.  Packets need to be routed globally —> (L3)
  4.  No reliable delivery on switch -> (L4)

| Functionality                      | Tasks                 | Protocols     | implementations |
| ---------------------------------- | --------------------- | ------------- | --------------- |
| Applications                       | Layer 7 (Application) | HTTP DNS      | HOST            |
| Reliable or unreliable transport   | Layer 4 (Transport)   | TCP UDP       | HOST            |
| Best-effort global packet delivery | Layer 3 (Network)     | IP            | ALL             |
| Best-effort local packet delivery  | Layer 2 (Data Link)   | Ethernet      | ALL             |
| Physical transfer of bits          | Layer 1 (Physical)    | Optical Radio | ALL             |

- Fate sharing (store state)
  - the communication's fate is tied to these hosts.
  - fundamental to the end-to-end principle
  - if there's a failure that causes the loss of state, it only impacts end host.
  - State: info from ongoing communication, such as tracking packet losses at end hosts.

## Physical layer (L1)

- signal modulation
  - packs bits on a signal by amplitude & phase
  - format
    - M = number of kinds in a symbol
    - N = $log_2M$ = number of bits per symbol
    - $f_p$ = symbol rate or baud rate in symbols/second
    - R = $f_plog_2M$ = data rate, bit rate in bits/second
- Noisy channels
  - High noise => harder to decode bits from symbols
  - Signal-to-noise ratio
    - Signal power is the power of the data signal that encodes bits
    - Noise power is the power of the noise on fiber
    - SNR = signal quality (dB) $= \frac{P_{signal}}{P_{noise}}$
  - max transition rate over a noisy channel
    - $R = B \times log_2(1 + NSR)$
    - R = 0.332 x B x SNR
    - B = bandwidth in Hz of the channel

1. Encoding: converting bits into a signal using modulation format
   - Signals flow between signaling components
   - Bits flow between network adaptors
     - connects a machine to the network
     - unique MAC address
2. Framing: delineating sequence of bits into messages
   - network adaptors recognize frame btw hosts
   - When Host 1 wants to send a frame to Host 2
     - Tells adaptor (uses signal component) to transmit a frame from its memory, resulting in a sequence of bits getting sent on the wire
     - Adaptor on Host 2 collects the sequence of bits and deposits the corresponding frame in Host 2’s memory
3. Error Detection: find corruption during transmission (cyclic redundancy check CRC)
   - sentinel bits
   - Sender Adds redundant bits of information to the frame (R)
   - Receiver Derives from the original frame (M) using an algorithm
   - k redundant bits for n message bits where k << n
4. Error Correction: 丢包 + 重新发
5. Media Access: mediate access of multiple nodes to the link
   - CSMA/CD for media access control

## Data link layer (L2)

- Broadcast
  - Each packet is received by all hosts
  - CSMA/CD is one example of media access control (MAC) protocol
- MAC protocols
  - Channel partitioning: divide the channel into parts (frequency division multiplexing)
  - Taking turns: decide who can use the channel at a point of time (time division multiplexing)
  - Random access: allow collisions, then recover from them
    - CSMA, CSMA/CD
      - listen before transmitting
        - If channel is idle, transmit the entire frame
        - If channel is busy, defer transmission
      - collisions due to non-zero propagation delay
      - transmission time > 2 X propagation delay
      - performance
        - $E = \frac{t_{trans}}{t_{trans} + Kd}$
        - K = the number of collisions per transmission
          - Kd = total collision time
        - For large packets E ~ 1
        - As bandwidth increases, E decreases
    - Switched Ethernet (current ethernets)
      - NO CSMA/CS, collisions
      - may has cycles - broadcast storms
      1.  Frames and Framing
          - Implemented in the network adaptor (NIC network interface controller) 把 data 转换成 0 和 1
      2.  Addressing:
          - dest & src MAC address from frame
          - plug-n-play
      3.  Routing
          - STP
      4.  Forwarding
          - causes storms, so we need STP
- spanning tree protocol for broadcast storms
  - (root, distance, sent by)
  - breaking ties
    - If A’s ID is < root, set root = A and next-hop = "sent by"
  - Flooding
    - Root sends packets to every neighbor recursively
    - cons
      - unnecessary packet processing
      - Higher latency
      - Lower bandwidth availability

## Network layer (L3)

routing of packets using routing tables

1. Addressing
   IP addresses
2. Forwarding (data plane - locally, ==direct by routing table==)
   - local router Output link for each packet
   - Read address from packet’s L3 header, search from routing table
3. Routing (control plane - distributed, ==compute routing table==)
   - network-wide process
   - end-to-end path
   - goals
     - validity
       - local routing
         - forwarding table in single router
       - global routing
         - collection of forwarding tables of all routers
         - always deliver to destination
       - valid routing state need correctness conditions for routing
         - no deadends / loops
     - efficient - least cost path to destination
       - least cost path routing - least cost path from each src to dest
         - Link state routing (Dijkstra)
         - Distance vector routing (Bellman-Ford algorithm)

- Link state routing (Dijkstra)
  - no privacy, limit autonomy
  - Routers broadcast their link state to all other routers
  - All routers build the global network graph
  - Centrally and locally compute shortest paths at each router
  - Link state routing protocol creates a global view of the network
    - Link state flooding (Link State Advertisements (LSAs))
      - When an LSA arrives at a router
        - remembers the packet
        - Forwards to other routers
      - If the same LSA arrives again
        - Drop it, do not forward again
    - local path calculation
  - Algorithm finds shortest paths on the global network view (Dijkstra)
    ```
    function Dijkstra(Graph, source):
        for each vertex v in Graph.Vertices:
            dist[v] <- INFINITY
            prev[v] <- UNDEFINED
            add v to Q
        dist[source] <- 0
        while Q is not empty:
            u <- vertex in Q with min dist[u]
            remove u from Q
            for each neighbor v of u still in Q:
                alt <- dist[u] + Graph.Edges(u, v)
                if alt < dist[v]:
                    dist[v] <- alt
                    prev[v] <- u
        return dist[], prev[]
    ```
  - Inconsistent link state cause loops
  - Convergence
    - same link state database
- Distance vector routing (Bellman-Ford algorithm)
  - no policy, vulnerable to loops
  - All routers run it “together” by exchange information
  - $D_x(y)$ = min { $C(x,v)$ + $D_v(y)$} for each node v ∈ N
  - Poisoned Reverse if Failures
    - if u -v is failed
- Autonomous Systems

  - network under a single administrative control
  - Intra-domain routing
    - within AS
    - LS routing, DV routing
  - Inter-domain routing
    - btw Ases
    - Administrative structure
      - autonomy - how it implements routing
      - privacy
      - policy - routing policy
    - Host addressing
      - Network (Prefix 32bits) + Host (Suffix 9bits)
      - Inter-domain routing operates only on the prefix
      - Hierarchical Addressing
        - Classful Addressing
      - Non-uniform
        - CDIR - flexible lengths
          - Crucial to aggregate addresses to reduce state and churn
          - Aggregation reduce routing table size
      - Multi-home
        - a given network has multiple “upstream provider networks”
        - prevents aggregation, increases routing table size
    - BGP (extended DV)
      - relationship btw ASes
        - Customer $\to$ everyone
        - Peer $\to$ Customer
        - Provider $\to$ Customer
      - Selection policy
        - how traffic leaves the network
        - #Make money & maximize performance & Minimize use of bandwidth
      - export policy
        - how traffic enters the network
        - advertise to neighbors
      - eBGP: btw ASes
      - iBGP: within AS
      - IGP: Interior Gateway Protocol = Intra-domain routing protocol
        - Provides reachability to destinations with in an AS
      - messages
        - Open
        - Update
        - Keepalive
      - #Route updates
      - #Route attributes
        - used in selection and export policies
        - ASPATH
          - route advertisement as traversed in reversed order
        - LOCAL PREF - pick highest
        - Multi-exit discriminator - select lowest
        - IGP cost - lowest IGP cost to next hop (egress router)
      - Diff to DV
        - not picking shortest paths
        - path vector routing, to detect loop
        - selective route advertisement $\neq$ DV
          - reachability to routes is not guaranteed
        - route aggregation for scalability $\notin$ DV
          - aggregate prefixes cleverly
      - Similarity to DV
        - Route advertisements for destination prefixes
        - No global sharing of network topology
        - Iterative or distributed convergence on paths
      - Issues
        - Reachability due to policy routing
        - #Security
        - Convergence for arbitrary policies
        - Performance - internal router hops not in count
        - #Misconfigurations
          - BGP is both bloated and under-specified
            - Lot of attributes
            - Lot of leeway in how to set and interpret attributes
            - Necessary to allow autonomy and diverse policies
            - But also makes it very open ended for operators
          - Much of the BGP configuration is manual and ad-hoc

  #Software-defined networking (SDN)

## Switch/Router Architecture

- Routers forward packets
  - N = No. Of external router ports
  - R = bandwidth (“line rate”) of a port
  - Router capacity = N x R
- Inside a router
  - Input linecards
    - Receive packets
    - Update IP header
    - Lookup the output port for the destination IP address
      - if not match then use Default route
      - longest prefix match
        - binary tree - Tire
    - Queue the packet at the switch fabric
  - Output linecard
    - Packet classification: map each packet into a flow
      - Classify a packet based on values in its header: Source and destination addresses, ports, “type of service”, “protocol”
      - #flow: A set of packets belonging to the same src—dest pair
    - Buffer management: drop packet
    - Scheduler: transmit packet on the wire among queues / flows

## Transport Layer

- pipe abstraction to applications
  - Data goes into pipe and emerges from the other side
  - Pipes are between process (not end hosts)
  - Two pipe abstractions:
    - User Datagram Protocol (UDP)
      - Unreliable packet delivery (application is responsible for resending)
    - Transmission control protocol (TCP)
      - Reliable byte stream
      - Bytes entered at one end emerge in-order at the other end
- Multiplexing and demultiplexing
  - Mux/demux packets from/to applications using ports and sockets.
    - Optional: Provide reliable packet delivery - needs many mechanisms (TCP is the standard example of reliable transport)
  - receive
    - IP header in the packet has src and dest IP addr
    - Transport layer header has src and dest port numbers
  - send by IP addr and port num
- Goals
  - Correctness condition for reliability
    - resends, make progress
  - Fairness: flows must get a fair share of the network resources
  - Flow performance: flows should get reasonable latency, jitter
  - Utilization: maximize bandwidth utilization in the network
- send pkt reliably
  - single
    - Send packet, set timer, wait for ACK, resend if no ACK when timer expires
  - multiple
    - Window-based algorithms
      - When get ACK'd, Send W packets
      - ideal
        - when ACK of first packet arrives, finished sending the last of W packets
      - Bandwidth of the pipe _ RTT = 2 _ BDP
    - Allow multiple packets (W) to be in flight
    - ideal window size = $\dfrac{B_w \text{ of pipe} * \text{RTT}}{\text{packet size}}$
- Design Considerations for reliable delivery
  - Nature of feedback from the receiver
  - Detection of loss
  - Response to loss
- Possible design for reliable transport

  - Cumulative ACKs
  - Window-based with retransmissions after
    - Timeouts
    - K subsequent duplicate ACKs
  - Correct, high performance, high utilization
  - Fairness
    - When senders want to send data at rate higher than link bandwidth
    - When there will be packet loss
    - Adjust W based on losses
    - In such a way that different flows receive the same share of bandwidth
    - Short version:
      - Cut W by half (multiplicative decrease)
      - Successive receipt of window: increase W by 1 (additive increase)

- TCP
- Focus
  - Packets identified by the bytes they carry
  - ACKs refer to bytes received
  - Window size in terms of number of bytes
- Components
  - Connections / Sessions: explicit setup & tear down of TCP sessions/connections
    - Reliability requires keeping state
      - Sender: packets sent but not yet ACK’d and related timers
      - Receiver: packets arrived out of order
    - Each byte stream in TCP is called a connection or session
      - Each session has its own #state (fate?)
      - State is at end hosts, not in the network
  - Segments, Sequence numbers, ACKs
    - TCP uses byte sequence numbers to identify packet payloads
    - ACKs refer to sequence numbers
    - Window sizes in bytes
    - Initial Sequence Number (ISN)
  - Retransmissions:
    - loss or corruption of packets, timeouts (estimated RTT) or duplicate ACKs
  - Flow control:
    - #Ensure sender does not overwhelm the receiver (ATTACK?)
    - Advertised window = W
      - send W bytes beyond the next expected byte
      - Receiver uses W to prevent sender from overflowing buffer
      - Limits the number of bytes the sender can have in flight
    - Filling the pipe
  - Congestion control
    - Dynamic adaptation to network paths’ capacity
    - End hosts adjust sending rate
    - Based on implicit feedback from the network
      - Implicit: router drops packets because buffer overflows
    - Hosts probe the network to test the level of congestion
      - Speed up when no congestion (no drops)
      - Slow down the there is congestion (packet drops)
    - Efficiently
      - Extend TCP’s window-based protocol
      - Adapt the window size in response to congestion
    - Congestion Window (CWND)
      - Maximum # of unacknowledged bytes to have in flight
      - Data Rate ~CWND/RTT
      - Adjustment
        - Consequences of over-sized window much worse than having an under-sized window
          - Over-sized window: packets dropped and retransmitted
          - Under-sized window: somewhat lower throughput
        - Approach
          - Gentle increase when un-congested (exploration)
          - Rapid decrease when congested
      - Additive increase
        - On success of last window of data, increase W
      - Multiplicative decrease
        - On loss of packets by DupACKs, divide congestion window by half
    - Adapting the congestion window
      - Increase upon lack of congestion: optimistic exploration
      - Decrease upon detecting congestion
    - detect congestion by packet loss
