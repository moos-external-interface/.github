# MIT MOOS Gateway

**Bidirectional MOOS-IvP ↔ ROS 2 middleware bridge for autonomous marine vehicles.**

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
50 Hz update rate, automatic reconnection, and a heartbeat mechanism that
keeps the connection alive through gateway restarts and network interruptions.
Supports multi-robot deployments via configurable robot IDs and ports.

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
┌──────────────────────────────────────┐
│             ROS 2 Graph              │
│                                      │
│  Perception  ◄──── /gateway_msg      │
│  Navigation                          │
│  Simulation  ────► /send_to_gateway  │
│  (Gazebo)                            │
└─────────────────┬────────────────────┘
                  │ TCP + Protobuf
                  │ (lib_gateway_tcp)
                  │ 50 Hz | auto-reconnect
┌─────────────────▼────────────────────┐
│           MOOS Community             │
│                                      │
│  iMOOSGateway ◄──► MOOSDB            │
│                       │              │
│             pHelmIvP  │              │
│             pNodeReport              │
│             pShare    │              │
└──────────────────────────────────────┘
```

---

## Field-Proven

The MIT MOOS Gateway has been developed, tested, and deployed on autonomous
surface vehicles (ASVs) operating in real ocean environments.  It supports
multi-vehicle formation missions including underway replenishment (UNREP)
operations where precise inter-vessel coordination is safety-critical.

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

**Authors:** Michael DeFilippo, Dr. Supun Randeni
**Institution:** MIT Department of Mechanical Engineering

---

*Keywords: MOOS-IvP ROS2 bridge, maritime autonomy middleware, autonomous
surface vehicle, ASV ROS2, MOOS ROS integration, multi-vehicle coordination,
marine robotics middleware, ROS2 Jazzy MOOS, iMOOSGateway, protobuf MOOS,
underway replenishment autonomy, MIT AUV Lab.*
