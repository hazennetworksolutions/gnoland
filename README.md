<div align="center">

# 🌐 Gnoland Test9 Full Node & Validator Setup Guide

**A complete guide to running a Gnoland test9 full node and registering as a validator**  
*Build from source, configuration, snapshot sync, and validator registration via GovDAO — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Gnoland](https://img.shields.io/badge/Gnoland-Test9.1-A259FF?style=flat-square)](https://gno.land)
[![Version](https://img.shields.io/badge/Tag-chain%2Ftest9.1-brightgreen?style=flat-square)](https://github.com/gnolang/gno/releases)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-test9.1-blue?style=flat-square)](https://docs.gno.land)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Network:** Gnoland Test9.1 Testnet (Chain ID: test9.1)  
> **Tag:** chain/test9.1  
> **Last Updated:** May 2026

---

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Network Endpoints](#network-endpoints)
- [Step 1 — System Verification](#step-1--system-verification)
- [Step 2 — System Update and Dependencies](#step-2--system-update-and-dependencies)
- [Step 3 — Install Go](#step-3--install-go)
- [Step 4 — Build Binaries from Source](#step-4--build-binaries-from-source)
- [Step 5 — Initialize, Download Genesis and Config](#step-5--initialize-node-secrets-and-config)
- [Step 6 — Download Genesis](#step-6--download-genesis)
- [Step 7 — Configure the Node](#step-7--configure-the-node)
- [Step 8 — Create Systemd Service](#step-8--create-systemd-service)
- [Step 9 — Apply Snapshot (Optional)](#step-9--apply-snapshot-optional)
- [Step 10 — Start the Node](#step-10--start-the-node)
- [Step 11 — Create a Wallet](#step-11--create-a-wallet)
- [Step 12 — Register as a Validator](#step-12--register-as-a-validator)
- [Monitoring the Node](#monitoring-the-node)
- [Useful Commands](#useful-commands)
- [Staying Updated](#staying-updated)

---

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| Operating System | Ubuntu 22.04+ | Ubuntu 24.04 |
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 16 GB |
| Disk | 100 GB SSD | 500 GB NVMe SSD |
| Network | 100 Mbps | 1 Gbps |

> ℹ️ Gnoland is a testnet — hardware requirements are lighter than production chains.

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| RPC | https://rpc.test9.testnets.gno.land:443 |
| Explorer | https://gnoscan.io |
| Faucet | https://faucet.gno.land |
| Official Docs | https://docs.gno.land |
| GitHub | https://github.com/gnolang/gno |

---

## Step 1 — System Verification

After SSH-ing into your server, verify the system meets requirements:

```bash
lsb_release -a          # Should be Ubuntu 22.04 or higher
uname -r                # Kernel version
lscpu | grep -E "Model name|CPU\(s\)|Thread|Socket|Core"
free -h                 # Minimum 8 GB RAM
df -h                   # Minimum 100 GB free disk
```

---

## Step 2 — System Update and Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget htop tmux build-essential jq make lz4 gcc unzip \
  screen btop iotop nethogs hdparm cmake perl automake autoconf libtool libssl-dev
```

---

## Step 3 — Install Go

Gnoland requires **Go 1.23+**:

```bash
cd $HOME
VER="1.23.0"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"

[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo 'export PATH=/usr/local/go/bin:$HOME/go/bin:$PATH' >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
export PATH="$HOME/go/bin:$PATH"
```

Verify the installation:

```bash
go version
```

Expected output: `go version go1.23.0 linux/amd64`

---

## Step 4 — Build Binaries from Source

Clone the repository and checkout the correct tag:

```bash
cd $HOME
git clone https://github.com/gnolang/gno.git
cd gno
git checkout tags/chain/test9.1
```

Build and install all binaries:

```bash
make install
make -C gno.land install.gnoland
make -C contribs/gnogenesis install
```

Copy binaries to system path:

```bash
sudo cp /root/go/bin/gno /usr/local/bin/
sudo cp /root/go/bin/gnokey /usr/local/bin/
sudo cp /root/go/bin/gnodev /usr/local/bin/
sudo cp /root/go/bin/gnoland /usr/local/bin/
sudo cp /root/go/bin/gnogenesis /usr/local/bin/

sudo chmod +x /usr/local/bin/gno
sudo chmod +x /usr/local/bin/gnokey
sudo chmod +x /usr/local/bin/gnodev
sudo chmod +x /usr/local/bin/gnoland
sudo chmod +x /usr/local/bin/gnogenesis
```

Verify:

```bash
gno version
gnoland --version
```

---

## Step 5 — Initialize Node Secrets and Config

Do a quick local start and stop first to initialize the data directory:

```bash
cd $HOME/gno
gnoland start --lazy
```

Press `Ctrl+C` after a few seconds to stop. Then clean up the default data:

```bash
rm -rf gnoland-data/
rm -f genesis.json
```

## Step 6 — Download Genesis

```bash
cd $HOME/gno
wget -O genesis.json \
  https://gno-testnets-genesis.s3.eu-central-1.amazonaws.com/test9/genesis.json
```

Verify the checksum:

```bash
shasum -a 256 genesis.json
```

Initialize secrets and config:

```bash
gnoland secrets init
gnoland config init
```

---

## Step 7 — Configure the Node



Set your moniker first:

```bash
MONIKER="YOUR_MONIKER"
```

Apply all configuration settings one by one:

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
gnoland config set telemetry.enabled false
gnoland config set p2p.laddr "tcp://0.0.0.0:26656"
gnoland config set rpc.laddr "tcp://127.0.0.1:26657"
```

Set seeds and persistent peers:

```bash
gnoland config set p2p.seeds \
  "g1d60r9u40340kqrt62cffh6yuc0gfmevz60n8s9@gno-core-sen-01.test9.testnets.gno.land:26656"

gnoland config set p2p.persistent_peers \
  "g1d60r9u40340kqrt62cffh6yuc0gfmevz60n8s9@gno-core-sen-01.test9.testnets.gno.land:26656"
```

---

## Step 8 — Create Systemd Service

```bash
sudo tee /etc/systemd/system/gnoland.service > /dev/null << EOF
[Unit]
Description=Gnoland Node
After=network-online.target

[Service]
User=root
WorkingDirectory=/root/gno
ExecStart=$(which gnoland) start \
  --genesis /root/gno/genesis.json \
  --data-dir /root/gno/gnoland-data \
  --skip-genesis-sig-verification
Restart=always
RestartSec=5
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable gnoland
```

---

## Step 9 — Apply Snapshot (Optional)

Using a snapshot significantly speeds up the initial sync. Make sure the node is **stopped** before applying.

```bash
sudo systemctl stop gnoland
sudo apt install lz4 -y

wget -O - https://server-9.hazennetworksolutions.com/gnoland-db-snapshot.tar.lz4 \
  | lz4 -d | tar -xf - -C /root/gno/gnoland-data
```

---

## Step 10 — Start the Node

```bash
sudo systemctl enable gnoland
sudo systemctl restart gnoland
sudo journalctl -u gnoland -f --no-hostname -o cat
```

Verify the service is running:

```bash
sudo systemctl status gnoland --no-pager
```

The service should show `active (running)`.

Check sync status:

```bash
curl -s http://localhost:26657/status | jq .result.sync_info
```

Wait until `catching_up` is `false` before proceeding to validator registration.

---

## Step 11 — Create a Wallet

Create a new wallet:

```bash
gnokey add wallet
```

> ⚠️ **CRITICAL:** Save your mnemonic phrase in a secure location. Without it, you cannot recover your wallet.

To recover an existing wallet:

```bash
gnokey add wallet --recover
```

List your wallets:

```bash
gnokey list
```

---

## Step 12 — Register as a Validator

> ⚠️ Gnoland uses a **GovDAO-based validator registration** system. Registration is done by calling a realm (smart contract), not via a standard staking transaction. Validator approval depends on DAO governance.

### Get your Node ID and Validator Key

```bash
gnoland secrets get node_id
```

Output example:
```json
{
  "id": "YOUR_NODE_ID",
  "p2p_address": "YOUR_P2P_ADDRESS",
  "pub_key": "YOUR_NODE_PUBKEY"
}
```

```bash
gnoland secrets get validator_key
```

Output example:
```json
{
  "address": "YOUR_VALIDATOR_ADDRESS",
  "pub_key": "YOUR_VALIDATOR_PUBKEY"
}
```

> The **address** and **pub_key** from `validator_key` are what you need for registration.

### Submit Validator Registration

Replace `VALADRESS`, `VALPUBKEY`, `MONIKER`, `INFORMATION`, and `WALLETNAME` with your actual values:

```bash
gnokey maketx call \
  -pkgpath "gno.land/r/gnops/valopers" \
  -func "Register" \
  -gas-fee 1000000ugnot \
  -gas-wanted 30000000 \
  -broadcast \
  -chainid "test9.1" \
  -args "MONIKER" \
  -args "INFORMATION" \
  -args "VALADRESS" \
  -args "VALPUBKEY" \
  -remote "https://rpc.test9.testnets.gno.land:443" \
  wallet
```

> ℹ️ `INFORMATION` is a short description of your validator (e.g. your website or a brief note).  
> After submission, your registration will be reviewed by the GovDAO.

---

## Monitoring the Node

### Watch live logs:

```bash
sudo journalctl -u gnoland -f --no-hostname -o cat
```

### Check sync status:

```bash
curl -s http://localhost:26657/status | jq .result.sync_info
```

### Check node info:

```bash
curl -s http://localhost:26657/net_info | jq .result.n_peers
```

### Service management:

```bash
# Restart service
sudo systemctl restart gnoland

# Stop service
sudo systemctl stop gnoland

# Check status
sudo systemctl status gnoland
```

---

## Useful Commands

### Node Info

```bash
# Get node ID
gnoland secrets get node_id

# Get validator key
gnoland secrets get validator_key

# Check all secrets
gnoland secrets get
```

### Wallet

```bash
# List wallets
gnokey list

# Show wallet address
gnokey list | grep wallet

# Check balance
gnokey query bank/balances/YOUR_ADDRESS \
  -remote "https://rpc.test9.testnets.gno.land:443"
```

### Transactions

```bash
# Send tokens
gnokey maketx send \
  -send "1000000ugnot" \
  -to "RECIPIENT_ADDRESS" \
  -gas-fee 1000000ugnot \
  -gas-wanted 10000000 \
  -broadcast \
  -chainid "test9.1" \
  -remote "https://rpc.test9.testnets.gno.land:443" \
  wallet
```

---

## Staying Updated

Follow these channels to stay informed about upgrades and announcements:

- Discord: [Gnoland Discord](https://discord.com/invite/S8nKUqwkPn)
- GitHub: [gnolang/gno](https://github.com/gnolang/gno)
- Official Docs: [docs.gno.land](https://docs.gno.land)
- Explorer: [gnoscan.io](https://gnoscan.io)

### Upgrade to a New Version (example: test9.2)

```bash
sudo systemctl stop gnoland

cd $HOME/gno
git fetch --all --tags
git checkout tags/chain/test9.2
make install
make -C gno.land install.gnoland
make -C contribs/gnogenesis install

sudo cp /root/go/bin/gnoland /usr/local/bin/
sudo chmod +x /usr/local/bin/gnoland

sudo systemctl restart gnoland
sudo journalctl -u gnoland -f --no-hostname -o cat
```

---

*Guide maintained by [HazenNetworkSolutions](https://hazennetworksolutions.com)*
