<h1 align="center"> Gnoland Sapphire — Snapshot Service </h1>

<img width="1920" height="1080" alt="gnoland" src="https://github.com/user-attachments/assets/fcb92812-05a8-46f5-933a-66393fff96c2" />

**Skip syncing from genesis** — restore a recent Sapphire (`sapphire-1`) node-state snapshot instead of replaying the chain.

* [Gnoland Website](https://gno.land/)<br>
* [Gnoland Discord](https://discord.com/invite/S8nKUqwkPn)<br>
* [Gnoland Sapphire Explorer](https://sapphire.testnets.gno.land)<br>
* [Hazen Explorer (Sapphire)](https://explorer.hazennetworksolutions.com/gnoland-testnet/)<br>
* [Full Node & Validator Setup Guide](sapphire.md)<br>

---

## Snapshot details

| | |
|---|---|
| Chain | Sapphire (`sapphire-1`) |
| Contents | `db/` + `wal/` only (never `secrets/`, `config/`, or genesis) |
| Latest block height | `265198` |
| Size | ~5.51 GB compressed (`lz4`) |
| Generated at | `2026-08-19T03:18:50Z` |
| SHA-256 | `d7e8c5b32c483aaebc36f19f8490d7db962742b7413bb35834cf00a871376064` |
| Manifest (JSON, always current) | https://server-9.hazennetworksolutions.com/gnoland-sapphire/index.json |

The manifest above always points at the *latest* snapshot — check it before restoring if you want the current block height/hash without downloading first.

---

## Option A — One-line restore (recommended)

```bash
curl -fsSL https://server-9.hazennetworksolutions.com/gnoland-sapphire/restore.sh | bash
