<div align="center">

# 🌐 Gnoland Pearl — Full Node & Validator Setup Guide

**A complete guide to running a Gnoland Pearl full node and registering as a validator**  
*Build from source, configuration, fast fresh-genesis sync, and validator registration via GovDAO — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Gnoland](https://img.shields.io/badge/Gnoland-Pearl-4ADEDE?style=flat-square)](https://gno.land)
[![Branch](https://img.shields.io/badge/Branch-chain%2Fpearl-brightgreen?style=flat-square)](https://github.com/gnolang/gno)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-pearl--1-blue?style=flat-square)](https://docs.gno.land)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions  
> **Network:** Gnoland Pearl Testnet (Chain ID: pearl-1)  
> **Branch:** chain/pearl  
> **Last Updated:** August 2026

---

## 📚 Guide

| Type | Language | Link |
|------|----------|------|
| Full Node & Validator Setup | 🇬🇧 English | [pearl.md](pearl.md) |

---

## 📋 Overview

Gnoland is a smart contract platform powered by Gnolang (Gno), an interpreted version of Go designed for transparency and composability. Pearl is the testnet that succeeds Sapphire in the Gnoland rollout — a **fresh chain, not a hardfork**: it builds a new ~2.6–2.7 MB genesis straight from the `examples/` tree (85 curated packages) in about a minute instead of replaying Sapphire's transaction history, so nodes sync in minutes and **no balances, realms, or names carry over from Sapphire**.

Pearl launches with **3 founding validators** (operator-keyed valoper profiles, power 60 each), namespace enforcement (`r/sys/names`) live from block 1, 3 pre-funded faucet accounts, unrestricted token transfers, and a new **genesis vesting accounts** feature (linear-unlock and cliff schedules for test accounts).

Validator registration on Gnoland is **GovDAO-based** — validators register by calling a realm (smart contract) and are added to the active set through a governance proposal. This process is permissioned and community-driven.

- Official Docs: [docs.gno.land](https://docs.gno.land)
- GitHub: [github.com/gnolang/gno](https://github.com/gnolang/gno)
- Release notes: [chain/pearl](https://github.com/gnolang/gno/releases/tag/chain%2Fpearl)
- Explorer: [pearl.testnets.gno.land](https://pearl.testnets.gno.land)
- Faucet: [pearl.testnets.gno.land/faucet](https://pearl.testnets.gno.land/faucet)
- Valopers: [pearl.testnets.gno.land/r/gnops/valopers](https://pearl.testnets.gno.land/r/gnops/valopers)
- Active Validators: [pearl.testnets.gno.land/r/sys/validators/v3](https://pearl.testnets.gno.land/r/sys/validators/v3)
- Gnockpit: [gnockpit.pearl.testnets.gno.land](https://gnockpit.pearl.testnets.gno.land)
- Status: [status.pearl.testnets.gno.land](https://status.pearl.testnets.gno.land)
- Tx-indexer (GraphQL): [indexer.pearl.testnets.gno.land/graphql](https://indexer.pearl.testnets.gno.land/graphql)
- Discord: [discord.com/invite/S8nKUqwkPn](https://discord.com/invite/S8nKUqwkPn)

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.  
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
