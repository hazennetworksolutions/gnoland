<div align="center">

# 🌐 Gnoland Onyx — Full Node & Validator Setup Guide

**A complete guide to running a Gnoland Onyx full node and registering as a validator candidate**  
*Pinned release binaries, verified genesis, mainnet-equivalent configuration, systemd operation, and GovDAO-based validator onboarding — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Gnoland](https://img.shields.io/badge/Gnoland-Onyx-111827?style=flat-square)](https://gno.land)
[![Release](https://img.shields.io/badge/Release-chain%2Fonyx-brightgreen?style=flat-square)](https://github.com/gnolang/gno/releases/tag/chain%2Fonyx)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-onyx--1-blue?style=flat-square)](https://docs.gno.land)
[![Launch](https://img.shields.io/badge/Launch-v1.5.0-8B5CF6?style=flat-square)](https://github.com/gnolang/gno/releases/tag/v1.5.0)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions  
> **Network:** Gnoland Onyx Testnet (Chain ID: onyx-1)  
> **Chain release:** chain/onyx  
> **Launch version:** v1.5.0  
> **Launch time:** September 28, 2026 at 00:00 UTC  
> **Last Updated:** September 2026

---

## 📚 Guide

| Type | Language | Link |
|------|----------|------|
| Full Node & Validator Setup | 🇬🇧 English | [onyx.md](onyx.md) |

---

## 📋 Overview

Gnoland is a smart-contract platform powered by Gnolang (Gno), an interpreted version of Go designed for transparency and composability. **Onyx** is the mainnet-rehearsal testnet that replaces Pearl. It is a fresh chain, not a Pearl hardfork: Pearl balances, packages, names, and state do not carry over.

Onyx launched with the unchanged **v1.5.0 mainnet binaries** and is designed to run mainnet code one release candidate ahead from the next release onward. Every coordinated mainnet upgrade is rehearsed on Onyx first. Operators must therefore use the exact version in Onyx's [`UPGRADES.md`](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/UPGRADES.md), never a floating tag, `master`, or an arbitrary branch tip.

Onyx mirrors mainnet's genesis shape and operating rules:

- 89 curated genesis packages on the `/v0` layout, byte-for-byte aligned with mainnet's package list at launch
- The same seven initial namespaces, with `r/sys/names` enforcement enabled from block 1
- The same sole GovDAO T1 seed, `aeddi`
- Inert post-genesis package submission with the `gpao` approvals oracle
- `maketx run` restricted to the seeded member
- One founding validator, `gno-core-validator-1`

The differences are economic and testnet-oriented: Onyx uses faucet GNOT, funds four operational accounts, has open transfers from genesis, has no Constitution §126 lock or exemption list, and has no vesting allocation.

Validator onboarding is GovDAO-based. An operator first registers a valoper profile on `gno.land/r/gnops/valopers`; registration creates a **candidate**, not an active validator. A GovDAO proposal through `r/sys/validators/v0` must pass before the node enters the active set.

### Official Resources

- Chain release and genesis: [chain/onyx](https://github.com/gnolang/gno/releases/tag/chain%2Fonyx)
- Version binaries: [v1.5.0](https://github.com/gnolang/gno/releases/tag/v1.5.0)
- Upgrade ledger: [UPGRADES.md](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/UPGRADES.md)
- Official validator notes: [VALIDATOR.md](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/VALIDATOR.md)
- Deployment directory: [onyx.gno.land](https://github.com/gnolang/gno/tree/chain/mainnet/misc/deployments/onyx.gno.land)
- Web / Explorer: [onyx.testnets.gno.land](https://onyx.testnets.gno.land)
- RPC: [rpc.onyx.testnets.gno.land](https://rpc.onyx.testnets.gno.land)
- Faucet: [onyx.testnets.gno.land/faucet](https://onyx.testnets.gno.land/faucet)
- Valopers: [onyx.testnets.gno.land/r/gnops/valopers](https://onyx.testnets.gno.land/r/gnops/valopers)
- Active Validators: [onyx.testnets.gno.land/r/sys/validators/v0](https://onyx.testnets.gno.land/r/sys/validators/v0)
- Gnockpit: [gnockpit.onyx.testnets.gno.land](https://gnockpit.onyx.testnets.gno.land)
- Status: [status.onyx.testnets.gno.land](https://status.onyx.testnets.gno.land)
- Documentation: [docs.gno.land](https://docs.gno.land)
- Discord: [discord.com/invite/S8nKUqwkPn](https://discord.com/invite/S8nKUqwkPn)

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.  
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
