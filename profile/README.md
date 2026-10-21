# MOOS External Interface (MEI)

**Interfaces that connect MOOS-IvP autonomy to everything outside the MOOSDB.**

MOOS-IvP is an excellent autonomy stack, but it speaks one language: MOOSDB
variables. Real vehicles and real operations need MOOS to talk to ROS 2 nodes,
ArduPilot autopilots, third-party command-and-control systems, TAK, and DDS
standards such as UMAA. MEI is a suite of small, focused bridges that do this
without forcing you to adopt one monolithic integration layer.

---

## Why a suite and not one package

Each integration serves a different user, so each is its own package.

- **The MOOSDB is the only contract.** Packages exchange data through MOOS
  variables, never through shared code. Run one, several, or all of them.
- **Use only what you need.** A TAK deployment does not pull in a MAVLink
  driver. A C2 integration does not pull in DDS.
- **Safe by default.** Commanding is off until you enable it. Commands are
  default-deny and allowlisted per vehicle. A stale or silent peer stops the
  vehicle rather than leaving it running.
- **Built for the MOOS loop.** Single-threaded, non-blocking, driven from
  `Iterate()`. Linux targets.

```
   EXTERNAL WORLD                                     VEHICLE SIDE

 ┌────────────────┐
 │ ROS 2 / Python │◄──────┐
 │ / C++ clients  │       │
 └────────────────┘       │
 ┌────────────────┐   ┌───┴──────────┐
 │ 3rd-party C2   │◄──┤  MEI: C2     │
 └────────────────┘   └───┬──────────┘
 ┌────────────────┐   ┌───┴──────────┐        ┌─────────┐        ┌──────────────┐
 │ ATAK / WebTAK  │◄──┤  MEI: TAK    ├───────►│         │◄───────┤ MEI: MAVLink ├──► ArduPilot / PX4
 └────────────────┘   └───┬──────────┘        │ MOOSDB  │        └──────────────┘
 ┌────────────────┐   ┌───┴──────────┐        │         │
 │ DDS / UMAA     │◄──┤  MEI: DDS    ├───────►│         │──► pHelmIvP (autonomy)
 └────────────────┘   └───┬──────────┘        │         │
 ┌────────────────┐   ┌───┴──────────┐        │         │
 │ Any TCP client │◄──┤ MEI: Gateway ├───────►│         │
 └────────────────┘   └──────────────┘        └─────────┘
```

---

## The packages

| Package | What it does | Typical user | Status |
|---|---|---|---|
| [**MEI: Gateway**](https://github.com/moos-external-interface/moos-ivp-gateway) | Shares MOOS variables with non-MOOS programs over TCP and Protocol Buffers | Teams bridging ROS 2 or custom software to MOOS | Wire protocol v1.0.0 frozen; Python and C++ clients |
| [**MEI: MAVLink**](https://github.com/moos-external-interface/moos-ivp-mavlink) | MOOS-IvP as the backseat driver for any MAVLink autopilot | Vehicle builders and integrators | ArduRover validated on hardware and in SITL; ArduCopter in SITL only |
| [**MEI: C2**](https://github.com/moos-external-interface/moos-ivp-c2) | A vendor-neutral command-and-control interface: telemetry out, validated commands in | C2 platform vendors and programs | Automated end-to-end suite (30 scenarios) |
| [**MEI: TAK**](https://github.com/moos-external-interface/moos-ivp-tak) | Puts MOOS vehicles on a TAK map and accepts waypoints and routes from TAK users | TAK operators and defense customers | In development; coexistence with MEI: C2 in progress |
| [**MEI: DDS**](https://github.com/moos-external-interface/moos-ivp-dds) | Translates between DDS and MOOS, starting with UMAA MO ICD v6.0 messages | UMAA and DDS integrators | Early development |
| [**lib_posix_comms**](https://github.com/moos-external-interface/lib_posix_comms) | Shared non-blocking socket classes (TCP, TLS, UDP multicast) | Used inside several MEI packages | Versioned separately |

---

## MEI: Gateway

A general-purpose door into the MOOSDB. `iMOOSGateway` is a TCP server that
forwards the MOOS variables you list to external clients and accepts variables
back, using Protocol Buffers. `iMOOSClient` is the inverse, for a vehicle that
connects out to a central server. Python and C++ client libraries are provided,
along with a ROS 2 client.

```
 ROS 2 node ──┐
 Python app ──┼──TCP/protobuf──►  iMOOSGateway ◄──► MOOSDB
 C++ app    ──┘                     (forward / block lists, heartbeats)
```

Use it when something that is not MOOS needs to read or write MOOS variables.

## MEI: MAVLink

`iMavlinkBridge` publishes autopilot telemetry as standard `NAV_*` variables and
turns the helm's `DESIRED_HEADING` and `DESIRED_SPEED` into MAVLink guided-mode
commands. The autopilot keeps low-level stabilization and navigation authority.
Commanding is off by default, and arming and mode changes stay manual unless you
opt in.

```
 pHelmIvP ──► DESIRED_HEADING/SPEED ──► iMavlinkBridge ──MAVLink UDP──► ArduPilot
                                              ▲                              │
                                              └────── NAV_* / MAVLINK_* ◄────┘
```

## MEI: C2

`iC2Gateway` gives an external C2 system one clean interface to a MOOS vehicle,
using length-prefixed protobuf messages over TCP.

- **Out:** capabilities, navigation state, vehicle health, alerts, and the
  status of every command as it moves from issued to completed.
- **In:** stop, goto, traverse, loiter, station-keep, convoy, hold, geofence,
  and operator commands. Every command is checked against a per-vehicle
  allowlist that denies by default.
- **Also:** contact and obstacle injection, lost-link policy, and arm, disarm
  and mode requests for autopilot-equipped vehicles.

```
 C2 client ◄──TCP──► ClientLink ──► CommandManager ──► MOOSDB ──► pHelmIvP
                         │              (allowlist, lifecycle)       │
                         └──── CommandStatus ◄── helm feedback ◄─────┘
```

## MEI: TAK

`pTAKBridge` connects a MOOS vehicle to a TAK Server. The vehicle appears on
ATAK and WebTAK, and TAK users can send go-to points and routes to it. It
connects by anonymous TCP for outbound-only use, or by mutual TLS for
bidirectional traffic. **UMVTool** is an optional ATAK plugin that shows vehicle
health and sends deploy and stop commands from inside ATAK.

```
 ATAK / WebTAK ◄──► TAK Server ◄──CoT (TCP or TLS)──► pTAKBridge ◄──► MOOSDB
        ▲                                                  ▲
        └────────── UMVTool plugin (health, deploy/stop) ──┘
```

## MEI: DDS

`pDDSBridge` translates between DDS and MOOS so a MOOS-IvP vehicle can take part
in a DDS-based architecture. The first profile targets the Unmanned Maritime
Autonomy Architecture (UMAA) MO ICD v6.0: global vector control with the full
command state machine, plus pose and speed status reports.

The design separates a generic DDS-to-MOOS engine from a thin UMAA overlay, so
other DDS profiles can reuse the engine. One binary can act as the commanding
side, the vehicle side, or both.

```
 UMAA consumer ◄──DDS──► [ UMAA overlay ] ◄──► [ generic engine ] ◄──► MOOSDB
   (commands, status)     types, state          participants,
                          machine, sessions     threading, FSM
```

This package is early. Expect the scope to grow service by service.

## lib_posix_comms

Small, non-blocking POSIX socket classes for use inside a MOOS `Iterate()` loop:
TCP client and server, TLS client with certificate authentication, and UDP
multicast receive. No threads, no callbacks, no exceptions. The caller owns the
loop. It is versioned in its own repository and shipped with each package that
uses it.

---

## Which package do I need?

| I want to... | Use |
|---|---|
| Read or write MOOS variables from ROS 2 or my own code | MEI: Gateway |
| Drive an ArduPilot or PX4 vehicle from the MOOS helm | MEI: MAVLink |
| Let a C2 system command and monitor my vehicle | MEI: C2 |
| See my vehicle in ATAK or send it waypoints from TAK | MEI: TAK |
| Connect MOOS to a DDS or UMAA system | MEI: DDS |

Packages combine freely. A vehicle running MEI: MAVLink and MEI: C2 exposes
arm, disarm, and mode control to a C2 purely through MOOS variables.

## Licensing and contact

MEI was developed at the MIT Department of Mechanical Engineering, Marine Autonomy Lab and the AUV Lab.
Licensing is handled through the MIT Technology Licensing Office
(tlo@mit.edu). Each package is licensed separately.

Technical contact: Michael DeFilippo (mikedef@mit.edu).

These repositories have a single maintainer and do not currently accept external
contributions.
