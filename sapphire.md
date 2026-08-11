<div align="center">

# 🌐 Gnoland Sapphire Full Node & Validator Setup Guide

**A complete guide to running a Gnoland Sapphire full node and registering as a validator**  
*Build from source, configuration, fast fresh-genesis sync, and validator registration via GovDAO — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Gnoland](https://img.shields.io/badge/Gnoland-Sapphire-38BDF8?style=flat-square)](https://gno.land)
[![Branch](https://img.shields.io/badge/Branch-chain%2Fsapphire-brightgreen?style=flat-square)](https://github.com/gnolang/gno)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-sapphire--1-blue?style=flat-square)](https://docs.gno.land)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions  
> **Network:** Gnoland Sapphire Testnet (Chain ID: sapphire-1)  
> **Branch:** chain/sapphire  
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
- [Known Issue Fixed Since Topaz — AppHash Mismatch](#known-issue-fixed-since-topaz--apphash-mismatch)
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

> ℹ️ Sapphire is a **fresh chain** (not a hardfork) with a ~2.6 MB genesis (85 curated packages) — disk and sync requirements stay light, same spirit as Topaz.

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| RPC | https://rpc.sapphire.testnets.gno.land |
| Explorer | https://sapphire.testnets.gno.land |
| Faucet | https://sapphire.testnets.gno.land/faucet |
| Valopers | https://sapphire.testnets.gno.land/r/gnops/valopers |
| Active Validators | https://sapphire.testnets.gno.land/r/sys/validators/v3 |
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

Sapphire requires **Go 1.25+** (its `go.mod` pins `go 1.25.9`, same pin as Topaz). This step installs Go 1.25.12 and configures the PATH:

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

> ℹ️ Unlike the Topaz release, the `chain/sapphire` release assets are built with `CGO_ENABLED=0`, so the prebuilt `gnoland`/`gnokey`/`gnoweb` binaries don't carry the newer-glibc dependency that forced Topaz operators to build from source. You can use either path in Step 4 below.

---

## Step 4 — Get the Binaries

You have two options: use the prebuilt release binaries, or build from source. Everything is built from the **`chain/sapphire`** branch.

### Option A — Prebuilt binaries (recommended)

Download the binaries matching your OS/arch from the [chain/sapphire release page](https://github.com/gnolang/gno/releases/tag/chain%2Fsapphire) (`gnoland`, `gnokey`, `gnoweb`, `gno` — `linux_amd64`, `linux_arm64`, `darwin_amd64`, `darwin_arm64`), then verify against the release's `CHECKSUMS.txt`:

```bash
cd $HOME
wget https://github.com/gnolang/gno/releases/download/chain/sapphire/gno_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/sapphire/gnoland_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/sapphire/gnokey_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/sapphire/gnoweb_linux_amd64
wget https://github.com/gnolang/gno/releases/download/chain/sapphire/CHECKSUMS.txt

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

Clone the official gno repository and checkout the sapphire branch:

```bash
cd $HOME
git clone https://github.com/gnolang/gno.git
cd gno
git checkout chain/sapphire
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
gnoland version: chain/sapphire
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

Download the official Sapphire genesis file:

```bash
wget -O genesis.json \
  https://github.com/gnolang/gno/releases/download/chain/sapphire/genesis.json
```

Verify the genesis checksum — **the hash must match exactly**:

```bash
shasum -a 256 genesis.json
```

Expected output:
```
d511e0e5b767d4e53f5c1afeeea1bc61d2c7b2118146c820f1f3e4296f67498e  genesis.json
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
  "g10xll77gz6yzg43v9mdalj8360ng6sunt2vvvhf@seed-1.sapphire.testnets.gno.land:26656,g1gw2d7qsmrg06p204ty2qs8ygzd32t2c7p46te0@seed-2.sapphire.testnets.gno.land:26656"
```

> ℹ️ Replace `YOUR-SERVER-IP` with your actual server's public IP address.

> ℹ️ Sapphire's own official [`VALIDATOR.md`](https://github.com/gnolang/gno/blob/chain/sapphire/misc/deployments/sapphire.gno.land/VALIDATOR.md) already lists these addresses under `p2p.persistent_peers` directly (this is the field the node actually reads) — unlike Topaz's `VALIDATOR.md`, which pointed at the unused `p2p.seeds` field. No workaround needed here.

---

## Step 7 — Create Systemd Service

Create the systemd service file to run the node as a managed background process:

```bash
sudo tee /etc/systemd/system/gnoland.service > /dev/null << EOF
[Unit]
Description=Gnoland sapphire-1 Node
After=network-online.target
Wants=network-online.target

[Service]
User=root
WorkingDirectory=/root/gno
Environment=GNOROOT=/root/gno
Environment=HOME=/root
ExecStart=$(which gnoland) start \
  --chainid sapphire-1 \
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

> ⚠️ `--skip-genesis-sig-verification` is **required** — some genesis transactions carry placeholder/patched signatures (e.g. the `r/sys/names.Enable` bootstrap call), and the node panics on startup without this flag.

---

## Step 8 — Sync Speed Note

Like Topaz, Sapphire starts from a **fresh, empty ~2.6 MB genesis** (85 curated packages) instead of replaying history. In practice a full sync from height 0 to chain tip takes well under an hour on a normal VPS once peers are connected (Step 6).

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
● gnoland.service - Gnoland sapphire-1 Node
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

Get testnet GNOT from the faucet: **https://sapphire.testnets.gno.land/faucet**

> ℹ️ Sapphire balances start from **zero** for everyone — nothing carries over from Topaz, even if you reuse the same wallet/mnemonic.

Verify your balance:

```bash
gnokey query \
  -remote "https://rpc.sapphire.testnets.gno.land" \
  auth/accounts/YOUR-G1-ADDRESS
```

---

## Step 11 — Register as a Validator

> ⚠️ Gnoland uses a **GovDAO-based validator registration** system. Registration is done by calling a realm (smart contract). Becoming active in the validator set requires a GovDAO governance proposal to pass — registration alone only lists you as a **candidate**.

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

> ⚠️ In Sapphire, only the `pub_key` is used for registration. The `address` here is the consensus key address — **do not use it** as the operator address. Use your wallet address from `gnokey list` instead.

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
  --chainid sapphire-1 \
  --remote https://rpc.sapphire.testnets.gno.land \
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
> https://sapphire.testnets.gno.land/r/gnops/valopers
>
> Registering only lists you as a candidate — a GovDAO member must then create and pass a proposal (via `r/sys/validators/v3`) to add you to the active validator set. Once that proposal executes, your node joins the valset.

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
  --chainid sapphire-1 \
  --remote https://rpc.sapphire.testnets.gno.land \
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
  -remote "https://rpc.sapphire.testnets.gno.land" \
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
  -chainid "sapphire-1" \
  -remote "https://rpc.sapphire.testnets.gno.land" \
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

## Known Issue Fixed Since Topaz — AppHash Mismatch

Topaz (and gno.land's Tendermint2 fork generally) suffered from a race condition — [gnolang/gno#6011](https://github.com/gnolang/gno/issues/6011) — where concurrent RPC queries during block commit could corrupt the app's state hash computation, eventually crashing the node with a `wrong Block.Header.AppHash` panic and no in-place repair.

The Sapphire release changelog lists the fix — **tm2: snapshot-isolated query paths and write-proof bptree fast-index maintenance** ([#6018](https://github.com/gnolang/gno/pull/6018)) — among its baseline changes, so `chain/sapphire` nodes already carry this fix from genesis. Hazen independently reproduced #6011 in production on topaz-1 (3 incidents + 3 deliberate reproductions) and validated #6018 against it: the fixed build synced genesis→tip under heavier query load than the load that reliably killed the unfixed build, with zero panics.

> ℹ️ Because this fix ships in Sapphire's own baseline, there is no equivalent "Disaster Recovery / AppHash Mismatch" workaround section in this guide — keep regular off-server backups of `secrets/` and `config/` as routine hygiene regardless.

---

## Staying Updated

- Discord: [Gnoland Discord](https://discord.com/invite/S8nKUqwkPn)
- GitHub: [gnolang/gno](https://github.com/gnolang/gno)
- Official Docs: [docs.gno.land](https://docs.gno.land)
- Explorer: [sapphire.testnets.gno.land](https://sapphire.testnets.gno.land)

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
