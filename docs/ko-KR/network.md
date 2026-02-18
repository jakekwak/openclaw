---
summary: "Network hub: gateway surfaces, pairing, discovery, and security"
read_when:
  - You need the network architecture + security overview
  - You are debugging local vs tailnet access or pairing
  - You want the canonical list of networking docs
title: "Network"
---

# Network hub

This hub links the core docs for how OpenClaw connects, pairs, and secures
devices across localhost, LAN, and tailnet.

## Core model

- [Gateway architecture](/ko-KR/concepts/architecture)
- [Gateway protocol](/ko-KR/gateway/protocol)
- [Gateway runbook](/ko-KR/gateway)
- [Web surfaces + bind modes](/ko-KR/web)

## Pairing + identity

- [Pairing overview (DM + nodes)](/ko-KR/start/pairing)
- [Gateway-owned node pairing](/ko-KR/gateway/pairing)
- [Devices CLI (pairing + token rotation)](/ko-KR/cli/devices)
- [Pairing CLI (DM approvals)](/ko-KR/cli/pairing)

Local trust:

- Local connections (loopback or the gateway host’s own tailnet address) can be
  auto‑approved for pairing to keep same‑host UX smooth.
- Non‑local tailnet/LAN clients still require explicit pairing approval.

## Discovery + transports

- [Discovery & transports](/ko-KR/gateway/discovery)
- [Bonjour / mDNS](/ko-KR/gateway/bonjour)
- [Remote access (SSH)](/ko-KR/gateway/remote)
- [Tailscale](/ko-KR/gateway/tailscale)

## Nodes + transports

- [Nodes overview](/ko-KR/nodes)
- [Bridge protocol (legacy nodes)](/ko-KR/gateway/bridge-protocol)
- [Node runbook: iOS](/ko-KR/platforms/ios)
- [Node runbook: Android](/ko-KR/platforms/android)

## Security

- [Security overview](/ko-KR/gateway/security)
- [Gateway config reference](/ko-KR/gateway/configuration)
- [Troubleshooting](/ko-KR/gateway/troubleshooting)
- [Doctor](/ko-KR/gateway/doctor)
