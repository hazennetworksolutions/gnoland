<div align="center">

# 🌐 Gnoland Pearl Full Node & Validator Setup Guide

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

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Network Endpoints](#network-endpoints)
- [Step 1 — System Verification](#step-1--system-verification)
- [Step 2 — System Update and Dependencies](#step-2--system-update-and-dependencies)
- [Step 3 — Install Go](#step-3--install-go)
- [Step 4 — Get the Binaries](#step-4--get-the-binaries)
- [Step 5 — Initialize, Download Genesis and Config](#step-5--initialize-download-genesis-and-config)
- [Step 6 — Configure the Node](#step-6--configure-the-node)
- [Step 7 — Create Systemd Service](#step-7--create-systemd-service)
- [Step 8 — Sync Speed Note](#step-8--sync-speed-note)
- [Step 9 — Start the Node](#step-9--start-the-node)
- [Step 10 — Create a Wallet](#step-10--create-a-wallet)
- [Step 11 — Register as a Validator](#step-11--register-as-a-validator)
- [Useful Commands](#useful-commands)
- [Firewall](#firewall)
- [What's New Since Sapphire](#whats-new-since-sapphire)
- [Staying Updated](#staying-updated)

---

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| Operating System | Ubuntu 22.04+ | Ubuntu 24.04 |
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 16 GB |
| Disk | 100 GB SSD | 250 GB NVMe SSD |
| Network | 100 Mbps | 1 Gbps |

> ℹ️ Pearl is a **fresh chain** (not a hardfork) with a ~2.6–2.7 MB genesis (85 curated packages) — disk and sync requirements stay light, same spirit as Topaz and Sapphire.

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| RPC | https://rpc.pearl.testnets.gno.land |
| Explorer (Gnoweb) | https://pearl.testnets.gno.land |
| Faucet | https://pearl.testnets.gno.land/faucet |
| Valopers | https://pearl.testnets.gno.land/r/gnops/valopers |
| Active Validators | https://pearl.testnets.gno.land/r/sys/validators/v3 |
| Gnockpit (dashboard) | https://gnockpit.pearl.testnets.gno.land |
| Status page | https://status.pearl.testnets.gno.land |
| Tx-indexer (GraphQL) | https://indexer.pearl.testnets.gno.land/graphql |
| Official Docs | https://docs.gno.land |
| GitHub | https://github.com/gnolang/gno |

---

## Step 1 — System Verification

After SSH-ing into your server, verify the system meets requirements:

```bash
lsb_release -a
uname -r
lscpu | grep -E "Model name|CPU\(s\)|Thread|Socket|Core"
free -h
df -h
```

---

## Step 2 — System Update and Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget htop tmux build-essential jq make lz4 gcc unzip \
  screen btop iotop nethogs hdparm cmake perl automake autoconf libtool libssl-dev zstd pv
```

---

## Step 3 — Install Go

Pearl requires **Go 1.25+** (its `go.mod` pins `go 1.25.9`, same pin as Topaz and Sapphire). This step installs Go 1.25.12 and configures the PATH:

```bash
cd $HOME
VER="1.25.12"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"

[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo 'export PATH=/usr/local/go/bin:$HOME/go/bin:$PATH' >> ~/.bash_profile
echo 'export GNOROOT=$HOME/gno' >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
export PATH="$HOME/go/bin:$PATH"
```

Verify the installation:

```bash
go version
```

Expected output:
```
go version go1.25.12 linux/amd64
```

> ℹ️ If your installed Go is older than 1.25 but at least 1.21, `GOTOOLCHAIN=auto` (the default) will transparently download and use `go1.25.9` the first time you build — but installing 1.25+ directly avoids relying on that.

> ℹ️ Like Sapphire (and unlike Topaz), the `chain/pearl` release assets are built with `CGO_ENABLED=0`, so the prebuilt `gnoland`/`gnokey`/`gnoweb` binaries don't carry the newer-glibc dependency that forced Topaz operators to build from source. You can use either path in Step 4 below.

---

## Step 4 — Get the Binaries

You have two options: use the prebuilt release binaries, or build from source. Everything is built from the **`chain/pearl`** branch.

### Option A — Prebuilt binaries (recommended)

Download the binaries matching your OS/arch from the [chain/pearl release page](https://github.com/gnolang/gno/releases/tag/chain%2Fpearl) (`gnoland`, `gnokey`, `gnoweb`, `gno` — `linux_amd64`, `linux_arm64`, `darwin_amd64`, `darwin_arm64`), then verify against the release's `CHECKSUMS.txt`:

```bash
cd $HOME
wget https://github.com/gnolang/gno/releases/download/chain/pearl/gno_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/pearl/gnoland_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/pearl/gnokey_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/pearl/gnoweb_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/pearl/CHECKSUMS.txt

sha256sum -c CHECKSUMS.txt --ignore-missing

sudo install -m 0755 gno_linux_amd64 /usr/local/bin/gno
sudo install -m 0755 gnoland_linux_amd64 /usr/local/bin/gnoland
sudo install -m 0755 gnokey_linux_amd64 /usr/local/bin/gnokey
sudo install -m 0755 gnoweb_linux_amd64 /usr/local/bin/gnoweb
```

The rest of this guide runs its `gnoland`/`gnokey` commands from a `$HOME/gno` working directory (that's where Step 5 downloads `genesis.json` and where the systemd service's `WorkingDirectory`/`GNOROOT` point). Option B gets this directory for free from `git clone`; on the prebuilt-binary path you need to create it yourself:

```bash
mkdir -p $HOME/gno
```

### Option B — Build from source

Clone the official gno repository and checkout the pearl branch:

```bash
cd $HOME
git clone https://github.com/gnolang/gno.git
cd gno
git checkout chain/pearl
```

Build and install all binaries:

```bash
make -C gno.land install.gnoland install.gnokey
make install
make -C contribs/gnogenesis install
```

Copy binaries to system path and set permissions:

```bash
for bin in gno gnokey gnodev gnoland gnogenesis gnoweb; do
  sudo cp /root/go/bin/$bin /usr/local/bin/
  sudo chmod +x /usr/local/bin/$bin
done
```

### Verify the installation (either option)

```bash
gno version
gnoland version
```

Expected output (or similar):
```
gnoland version: chain/pearl
```

---

## Step 5 — Initialize, Download Genesis and Config

Run a quick start to generate the default data directory structure, then stop with `Ctrl+C`:

```bash
cd $HOME/gno
gnoland start --lazy
```

> ℹ️ Wait until you see the node start printing output, then press `Ctrl+C` to stop it.

Remove the default data and genesis to prepare for a clean setup:

```bash
rm -rf gnoland-data/ genesis.json
```

Download the official Pearl genesis file:

```bash
wget -O genesis.json \
  https://github.com/gnolang/gno/releases/download/chain/pearl/genesis.json
```

Verify the genesis checksum — **the hash must match exactly**:

```bash
shasum -a 256 genesis.json
```

Expected output:
```
c45fe60c8c8a1f859d9e4d5aad7ce4d100ff0eb78302e71318ba0de481a8dc91  genesis.json
```

> ⚠️ If the checksum does not match, do not continue. Re-download the genesis file.

Initialize node secrets and config:

```bash
gnoland secrets init
gnoland config init
```

---

## Step 6 — Configure the Node

Set your moniker (replace with your own node name):

```bash
MONIKER="YOUR_MONIKER"
```

Apply all required configuration settings:

```bash
cd $HOME/gno

gnoland config set moniker "$MONIKER"
gnoland config set application.prune_strategy syncable
gnoland config set consensus.timeout_commit 3s
gnoland config set consensus.peer_gossip_sleep_duration 10ms
gnoland config set p2p.flush_throttle_timeout 10ms
gnoland config set p2p.pex true
gnoland config set p2p.max_num_outbound_peers 40
gnoland config set mempool.size 10000
gnoland config set telemetry.metrics_enabled false
gnoland config set p2p.laddr "tcp://0.0.0.0:26656"
gnoland config set rpc.laddr "tcp://127.0.0.1:26657"
gnoland config set p2p.external_address "YOUR-SERVER-IP:26656"
gnoland config set p2p.persistent_peers \
  "g1m37xukfq6yl555k93fcyzns83qnmgyax9zm875@seed-1.pearl.testnets.gno.land:26656,g1ngukqd3khekaqjf90k45cglzm0l25wwzl2fkn2@seed-2.pearl.testnets.gno.land:26656"
```

> ℹ️ Replace `YOUR-SERVER-IP` with your actual server's public IP address.

> ℹ️ Pearl's own official [`VALIDATOR.md`](https://github.com/gnolang/gno/blob/chain/pearl/misc/deployments/pearl.gno.land/VALIDATOR.md) lists these addresses directly under `p2p.persistent_peers` (the field the node actually reads), same as Sapphire — no `p2p.seeds` workaround needed, unlike Topaz.

---

## Step 7 — Create Systemd Service

Create the systemd service file to run the node as a managed background process:

```bash
sudo tee /etc/systemd/system/gnoland.service > /dev/null << EOF
[Unit]
Description=Gnoland pearl-1 Node
After=network-online.target
Wants=network-online.target

[Service]
User=root
WorkingDirectory=/root/gno
Environment=GNOROOT=/root/gno
Environment=HOME=/root
ExecStart=$(which gnoland) start \
  --chainid pearl-1 \
  --genesis /root/gno/genesis.json \
  --skip-genesis-sig-verification
Restart=on-failure
RestartSec=5s
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=gnoland

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable gnoland
```

> ⚠️ `--skip-genesis-sig-verification` is **required** — some genesis transactions carry placeholder/patched signatures (e.g. the `r/sys/names.Enable` bootstrap call, whose caller field is jq-patched to the admin address at genesis), and the node panics on startup without this flag.

---

## Step 8 — Sync Speed Note

Like Topaz and Sapphire, Pearl starts from a **fresh, empty ~2.6–2.7 MB genesis** (85 curated packages) instead of replaying history — built from the repo's `examples/` tree in about a minute, booting in seconds. In practice a full sync from height 0 to chain tip takes well under an hour on a normal VPS once peers are connected (Step 6).

If you still want to check progress instead of waiting blind, see the sync-status command in [Step 9](#step-9--start-the-node) or [Useful Commands](#useful-commands).

---

## Step 9 — Start the Node

Start the gnoland service:

```bash
sudo systemctl restart gnoland
```

Follow the live logs to confirm the node is running:

```bash
sudo journalctl -u gnoland -f --no-hostname -o cat
```

Verify the service status:

```bash
sudo systemctl status gnoland --no-pager
```

Expected output:
```
● gnoland.service - Gnoland pearl-1 Node
     Active: active (running) since ...
```

Check sync status:

```bash
curl -s http://localhost:26657/status | jq .result.sync_info
```

Expected output when fully synced:
```json
{
  "latest_block_height": "XXXXXX",
  "catching_up": false
}
```

> ⚠️ Wait until `catching_up` is `false` before proceeding to validator registration.

---

## Step 10 — Create a Wallet

Create a new wallet:

```bash
gnokey add wallet
```

> ⚠️ **CRITICAL:** You will be shown a mnemonic phrase. Save it in a secure location immediately. Without it, you cannot recover your wallet.

To recover an existing wallet from mnemonic:

```bash
gnokey add wallet --recover
```

List your wallets and get your `g1...` address:

```bash
gnokey list
```

Expected output:
```
* wallet (local) - addr: g1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx pub: gpub1...
```

Get testnet GNOT from the faucet: **https://pearl.testnets.gno.land/faucet**

> ℹ️ Pearl balances start from **zero** for everyone — nothing carries over from Sapphire, even if you reuse the same wallet/mnemonic.

Verify your balance:

```bash
gnokey query \
  -remote "https://rpc.pearl.testnets.gno.land" \
  auth/accounts/YOUR-G1-ADDRESS
```

---

## Step 11 — Register as a Validator

> ⚠️ Gnoland uses a **GovDAO-based validator registration** system. Registration is done by calling a realm (smart contract). Becoming active in the validator set requires a GovDAO governance proposal to pass — registration alone only lists you as a **candidate**.

> ℹ️ Pearl launches with **3 founding validators** (`gno-core-validator-1/-2/-3`, power 60 each). At 3 × 60 power, a single validator going offline sits exactly at the one-third halt boundary — accepted for launch until additional validator partners join via the process below.

### Get your Validator Public Key

Run from the `/root/gno` directory:

```bash
cd /root/gno && gnoland secrets get validator_key
```

Expected output:
```json
{
  "address": "g1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "pub_key": "gpub1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

> ⚠️ In Pearl, only the `pub_key` is used for registration. The `address` here is the consensus key address — **do not use it** as the operator address. Use your wallet address from `gnokey list` instead.

### Submit Validator Registration

Replace all placeholder values before running. The registration must be **signed by your operator key** — the `gnokey` account whose `g1...` address you pass as the operator address; the realm rejects the call if the signer doesn't control that address:

```bash
gnokey maketx call \
  --pkgpath gno.land/r/gnops/valopers \
  --func Register \
  --args "MONIKER" \
  --args "DESCRIPTION" \
  --args "cloud|on-prem|data-center" \
  --args "OPERATOR_ADDRESS" \
  --args "VAL_PUBKEY" \
  --gas-fee 1000000ugnot \
  --gas-wanted 50000000 \
  --chainid pearl-1 \
  --remote https://rpc.pearl.testnets.gno.land \
  --broadcast \
  WALLETNAME
```

| Placeholder | Description |
|---|---|
| `MONIKER` | Your validator display name |
| `DESCRIPTION` | Short description of your validator |
| `cloud\|on-prem\|data-center` | Your infrastructure category |
| `OPERATOR_ADDRESS` | Your **wallet** `g1...` address from `gnokey list` |
| `VAL_PUBKEY` | `pub_key` from `cd /root/gno && gnoland secrets get validator_key` |
| `WALLETNAME` | Key name from `gnokey list` |

> ℹ️ After a successful transaction you can view your profile at:  
> https://pearl.testnets.gno.land/r/gnops/valopers
>
> Registering only lists you as a candidate — a GovDAO member must then create and pass a proposal (via `r/sys/validators/v3`) to add you to the active validator set. Once that proposal executes, your node joins the valset. Pearl launches with `aeddi` as the sole GovDAO T1 member; additional members join via GovDAO proposals.

### Update Description (Optional)

Description limit is **2048 characters**. To update after registration:

```bash
gnokey maketx call \
  --pkgpath gno.land/r/gnops/valopers \
  --func UpdateDescription \
  --args "YOUR-G1-OPERATOR-ADDRESS" \
  --args "YOUR-NEW-DESCRIPTION" \
  --gas-fee 1000000ugnot \
  --gas-wanted 50000000 \
  --chainid pearl-1 \
  --remote https://rpc.pearl.testnets.gno.land \
  --broadcast \
  WALLETNAME
```

---

## Useful Commands

### Service Management

```bash
sudo systemctl start gnoland
sudo systemctl stop gnoland
sudo systemctl restart gnoland
sudo systemctl status gnoland
```

### Logs & Sync

```bash
# Live logs
sudo journalctl -u gnoland -f --no-hostname -o cat

# Logs from last hour
sudo journalctl -u gnoland --since "1 hour ago"

# Sync status
curl -s http://localhost:26657/status | jq .result.sync_info

# Connected peers
curl -s http://localhost:26657/net_info | jq .result.n_peers
```

### Node Info

> ℹ️ Run secrets commands from `/root/gno` directory:

```bash
cd /root/gno && gnoland secrets get node_id
cd /root/gno && gnoland secrets get validator_key
cd /root/gno && gnoland secrets get
```

### Wallet & Balance

```bash
# List wallets
gnokey list

# Check balance
gnokey query \
  -remote "https://rpc.pearl.testnets.gno.land" \
  auth/accounts/YOUR_ADDRESS
```

### Send Tokens

```bash
gnokey maketx send \
  -send "1000000ugnot" \
  -to "RECIPIENT_ADDRESS" \
  -gas-fee 1000000ugnot \
  -gas-wanted 10000000 \
  -broadcast \
  -chainid "pearl-1" \
  -remote "https://rpc.pearl.testnets.gno.land" \
  wallet
```

---

## Firewall

```bash
# P2P — must be open to the public
sudo ufw allow 26656/tcp comment "gnoland P2P"

# RPC — open only if you serve public endpoints
sudo ufw allow 26657/tcp comment "gnoland RPC"
```

---

## What's New Since Sapphire

Pearl carries the fix for the [gnolang/gno#6011](https://github.com/gnolang/gno/issues/6011) AppHash-mismatch race condition in its baseline, same as Sapphire — no equivalent "Disaster Recovery" workaround section is needed here (see the [Sapphire guide](../sapphire/sapphire.md#known-issue-fixed-since-topaz--apphash-mismatch) for the history). Keep regular off-server backups of `secrets/` and `config/` as routine hygiene regardless.

Notable additions in this release (see the [chain/pearl release notes](https://github.com/gnolang/gno/releases/tag/chain%2Fpearl) for the full changelog):

- **Genesis vesting accounts** — the genesis builder now supports a `VESTED_ACCOUNTS` list (continuous linear-unlock or delayed/cliff schedules). Pearl ships ten vested test accounts covering the schedule matrix; this has no effect on validator/full-node setup.
- **`grc721` rewritten** on a Token/PrivateLedger/Teller axis, with stackable metadata, royalty, and enumerable extensions ([#6072](https://github.com/gnolang/gno/pull/6072), [#6073](https://github.com/gnolang/gno/pull/6073)).
- **`AddPackage` now type-checks production files only** — test files (`_test.gno`) are stored and syntax-parsed but not type-checked at deploy time ([#6025](https://github.com/gnolang/gno/pull/6025)).
- **`p2p.seeds` is now actually wired into the switch** ([#6023](https://github.com/gnolang/gno/pull/6023)) — the field that was silently ignored back in Topaz (Step 6 note above) is functional as of this release, though this guide continues to use `p2p.persistent_peers` per Pearl's own `VALIDATOR.md`.
- Three separate cases where coins could be lost or over-authorized around the send envelope were fixed ([#6062](https://github.com/gnolang/gno/pull/6062)).
- GovDAO allowlist lockdown, proposal-page escaping, and executor disclosure ([#6068](https://github.com/gnolang/gno/pull/6068)).

---

## Staying Updated

- Discord: [Gnoland Discord](https://discord.com/invite/S8nKUqwkPn)
- GitHub: [gnolang/gno](https://github.com/gnolang/gno)
- Official Docs: [docs.gno.land](https://docs.gno.land)
- Explorer: [pearl.testnets.gno.land](https://pearl.testnets.gno.land)
- Gnockpit: [gnockpit.pearl.testnets.gno.land](https://gnockpit.pearl.testnets.gno.land)
- Status: [status.pearl.testnets.gno.land](https://status.pearl.testnets.gno.land)

### Upgrade to a New Version

```bash
sudo systemctl stop gnoland

cd $HOME/gno
git fetch --all --tags
git checkout chain/NEXT_TAG
make -C gno.land install.gnoland install.gnokey
make install
make -C contribs/gnogenesis install

sudo cp /root/go/bin/gnoland /usr/local/bin/
sudo chmod +x /usr/local/bin/gnoland

sudo systemctl restart gnoland
sudo journalctl -u gnoland -f --no-hostname -o cat
```

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.  
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
