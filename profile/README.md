# MIT MOOS Gateway

**Bidirectional MOOS-IvP ↔ ROS 2 middleware bridge for autonomous vehicles.**

Developed at the [MIT AUV Lab](https://oceanai.mit.edu), Department of
Mechanical Engineering.  The MIT MOOS Gateway connects the proven
[MOOS-IvP](https://oceanai.mit.edu/moos-ivp) maritime autonomy stack with
modern ROS 2 robotics infrastructure, enabling teams to run production
missions while leveraging ROS 2 for perception, sensor fusion, and simulation.

---

## Why MOOS-IvP + ROS 2?

MOOS-IvP is the autonomy middleware of choice for many maritime robotics
programs worldwide.  Its multi-objective helm, inter-vehicle communication,
and mission management tools are mature and field-proven.  ROS 2 brings a
rich ecosystem of perception, sensor drivers, simulation (Gazebo), and
visualization tools.

The MIT MOOS Gateway makes both available simultaneously — no need to choose.

---

## Repositories

### [`protobuf_client_ros2`](https://github.com/mit-moos-gateway/protobuf_client_ros2)

A ROS 2 node (`protobuf_client_node`) that acts as a TCP client to
`iMOOSGateway`, bridging MOOS variables and ROS 2 topics bidirectionally via
Google Protocol Buffers.  Built on Boost ASIO for non-blocking I/O with a
configurable polling interval, automatic reconnection, and a heartbeat
mechanism that keeps the connection alive through gateway restarts and
network interruptions. Supports multi-robot deployments via configurable
robot IDs and ports.

### [`lib_gateway_tcp`](https://github.com/mit-moos-gateway/lib_gateway_tcp)

A portable, middleware-agnostic C++ library implementing the TCP transport and
protobuf serialization layer shared across the gateway stack.  Handles
connection lifecycle, protobuf encoding and decoding for MOOS variable types
(double, string, binary), and is used as a core dependency by
`protobuf_client_ros2`.

### [`moos-ivp-gateway`](https://github.com/mit-moos-gateway/moos-ivp-gateway)

A MOOS-IvP application suite containing `iMOOSGateway` (TCP server) and
`iMOOSClient` (TCP client) that run inside the MOOS community as standard
`pAntler`-launched processes.  `iMOOSGateway` exposes a TCP server for
external ROS 2 clients to connect to, forwarding configurable MOOS variables
in both directions and posting inbound protobuf messages directly to the
MOOS DB.

---

## Architecture

```
┌──────────────────────────────┐     ┌─────────────────────────────────┐
│       MOOS-IvP Mission       │     │       External Ecosystems       │
│  ┌──────────────────────┐    │TCP  │  ┌───────────────────────────┐  │
│  │    iMOOSGateway      │◄───┼─────┼─►│   protobuf_client_ros2    │  │
│  │  (TCP Server Hub)    │    │     │  │       (ROS 2 Node)        │  │
│  └──────────┬───────────┘    │     │  └───────────────────────────┘  │
│        ┌────▼─────┐          │TCP  │  ┌───────────────────────────┐  │
│        │  MOOSDB  │          │◄────┼─►│     iMAVLinkBridge        │  │
│        │  NAV_X/Y │          │     │  │  (ArduSub / PX4 / UAV)    │  │
│        │  DEPLOY  │          │TCP  │  └───────────────────────────┘  │
│        └──────────┘          │◄────┼─►│      iMOOSClient          │  │
└──────────────────────────────┘     │  │  (Remote MOOS Community)  │  │
                                     │  └───────────────────────────┘  │
                                     └─────────────────────────────────┘
```
MEI architecture — iMOOSGateway serves as a central hub for all client types.

---

## Field-Proven

The MIT MOOS Gateway has been developed, tested, and deployed on autonomous
surface vehicles (ASVs) operating in real ocean environments.  It supports
multiple DARPA and ONR funded projects, including multi-vehicle formation
missions such as underway replenishment (UNREP) operations where precise
inter-vessel coordination is safety-critical.

Latency and throughput have been characterized through instrumented
testing: the gateway sustains multi-Hz bidirectional telemetry and command
exchange between MOOS-IvP and ROS 2 with zero measured message loss.

Integration has been validated with:

- ROS 2 Humble and Jazzy
- MOOS-IvP 24.x
- Gazebo Harmonic (simulation)
- WAM-V and custom ASV platforms

---

## Licensing

The MIT MOOS Gateway is available for licensing for research, commercial, and
government applications.  Please contact the MIT Technology Licensing Office
or reach out via the MIT AUV Lab for inquiries.

**Authors:** Michael DeFilippo (mikedef@mit.edu), Dr. Supun Randeni (supun@mit.edu)
**Institution:** MIT Department of Mechanical Engineering
