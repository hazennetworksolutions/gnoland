<div align="center">

# 🌐 Gnoland Test13 Full Node & Validator Setup Guide

**A complete guide to running a Gnoland test13 full node and registering as a validator**  
*Build from source, configuration, snapshot sync, and validator registration via GovDAO — step by step.*

[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04+-E95420?style=flat-square&logo=ubuntu&logoColor=white)](https://ubuntu.com)
[![Gnoland](https://img.shields.io/badge/Gnoland-Test13-A259FF?style=flat-square)](https://gno.land)
[![Branch](https://img.shields.io/badge/Branch-chain%2Ftest13-brightgreen?style=flat-square)](https://github.com/gnolang/gno)
[![Chain ID](https://img.shields.io/badge/Chain%20ID-test--13-blue?style=flat-square)](https://docs.gno.land)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[hazennetworksolutions.com](https://hazennetworksolutions.com)

</div>

---

> **Author:** HazenNetworkSolutions  
> **Network:** Gnoland Test13 Testnet (Chain ID: test-13)  
> **Branch:** chain/test13  
> **Last Updated:** June 2026

---

## Table of Contents

- [Hardware Requirements](#hardware-requirements)
- [Network Endpoints](#network-endpoints)
- [Step 1 — System Verification](#step-1--system-verification)
- [Step 2 — System Update and Dependencies](#step-2--system-update-and-dependencies)
- [Step 3 — Install Go](#step-3--install-go)
- [Step 4 — Build Binaries from Source](#step-4--build-binaries-from-source)
- [Step 5 — Initialize, Download Genesis and Config](#step-5--initialize-download-genesis-and-config)
- [Step 6 — Configure the Node](#step-6--configure-the-node)
- [Step 7 — Create Systemd Service](#step-7--create-systemd-service)
- [Step 8 — Apply Snapshot (Optional)](#step-8--apply-snapshot-optional)
- [Step 9 — Start the Node](#step-9--start-the-node)
- [Step 10 — Create a Wallet](#step-10--create-a-wallet)
- [Step 11 — Register as a Validator](#step-11--register-as-a-validator)
- [Useful Commands](#useful-commands)
- [Staying Updated](#staying-updated)

---

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| Operating System | Ubuntu 22.04+ | Ubuntu 24.04 |
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 16 GB |
| Disk | 200 GB SSD | 500 GB NVMe SSD |
| Network | 100 Mbps | 1 Gbps |

> ℹ️ Gnoland is a testnet — hardware requirements are lighter than production chains.

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| RPC | https://rpc.test13.testnets.gno.land |
| Explorer | https://test13.testnets.gno.land |
| Faucet | https://test13.testnets.gno.land/faucet |
| Valopers | https://test13.testnets.gno.land/r/gnops/valopers |
| Active Validators | https://test13.testnets.gno.land/r/sys/validators/v3 |
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

Gnoland requires **Go 1.22+**. This step installs Go 1.23 and configures the PATH:

```bash
cd $HOME
VER="1.23.0"
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
go version go1.23.0 linux/amd64
```

---

## Step 4 — Build Binaries from Source

Clone the official gno repository and checkout the test13 branch:

```bash
cd $HOME
git clone https://github.com/gnolang/gno.git
cd gno
git checkout chain/test13
```

Build and install all binaries:

```bash
make install
make -C gno.land install.gnoland
make -C contribs/gnogenesis install
```

Copy binaries to system path and set permissions:

```bash
for bin in gno gnokey gnodev gnoland gnogenesis; do
  sudo cp /root/go/bin/$bin /usr/local/bin/
  sudo chmod +x /usr/local/bin/$bin
done
```

Verify the installation:

```bash
gno version
gnoland version
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

Download the official test13 genesis file:

```bash
wget -O genesis.json \
  https://github.com/gnolang/gno/releases/download/chain/test13/genesis.json
```

Verify the genesis checksum — **the hash must match exactly**:

```bash
shasum -a 256 genesis.json
```

Expected output:
```
56f56e135174feff9f93283d5ec7e4ec955cd5155108aff5009d4fd51c5adaf2  genesis.json
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
  "g142k7zc2qym3c0u6jmkf6rv26llgr2f4nakmlmt@sentry-1.test13.testnets.gno.land:26656,g1lxkf9gn7kddrr26c640ww5wg3ezsm22we8cjpc@sentry-2.test13.testnets.gno.land:26656"
```

> ℹ️ Replace `YOUR-SERVER-IP` with your actual server's public IP address.

---

## Step 7 — Create Systemd Service

Create the systemd service file to run the node as a managed background process:

```bash
sudo tee /etc/systemd/system/gnoland.service > /dev/null << EOF
[Unit]
Description=Gnoland test-13 Node
After=network-online.target
Wants=network-online.target

[Service]
User=root
WorkingDirectory=/root/gno
Environment=GNOROOT=/root/gno
Environment=HOME=/root
ExecStart=$(which gnoland) start \
  --chainid test-13 \
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

---

## Step 8 — Apply Snapshot (Optional)

Using a snapshot significantly speeds up the initial sync by skipping hours of block replay. Make sure the node is **stopped** before applying.

```bash
sudo systemctl stop gnoland
```

Clear only the database and WAL — your keys and config are **not touched**:

```bash
rm -rf /root/gno/gnoland-data/db /root/gno/gnoland-data/wal
```

Download and extract the snapshot:

```bash
wget -O - https://server-13.hazennetworksolutions.com/gnoland-db-snapshot.tar.lz4 \
  | lz4 -d | tar -xf - -C /root/gno/gnoland-data
```

> ℹ️ Snapshot size and download time will vary. Do not interrupt the process.

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
● gnoland.service - Gnoland test-13 Node
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

Get testnet GNOT from the faucet: **https://test13.testnets.gno.land/faucet**

Verify your balance:

```bash
gnokey query \
  -remote "https://rpc.test13.testnets.gno.land" \
  auth/accounts/YOUR-G1-ADDRESS
```

---

## Step 11 — Register as a Validator

> ⚠️ Gnoland uses a **GovDAO-based validator registration** system. Registration is done by calling a realm (smart contract). Becoming active in the validator set requires a GovDAO governance proposal to pass — registration alone is not enough.

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

> ⚠️ In test13, only the `pub_key` is used for registration. The `address` here is the consensus key address — **do not use it** as the operator address. Use your wallet address from `gnokey list` instead.

### Submit Validator Registration

Replace all placeholder values before running:

```bash
gnokey maketx call \
  --pkgpath gno.land/r/gnops/valopers \
  --func Register \
  --args "MONIKER" \
  --args "DESCRIPTION" \
  --args "data-center" \
  --args "OPERATOR_ADDRESS" \
  --args "VAL_PUBKEY" \
  --gas-fee 1000000ugnot \
  --gas-wanted 50000000 \
  --chainid test-13 \
  --remote https://rpc.test13.testnets.gno.land \
  --broadcast \
  WALLETNAME
```

| Placeholder | Description |
|---|---|
| `MONIKER` | Your validator display name |
| `DESCRIPTION` | Short description of your validator |
| `data-center` | Your infrastructure location |
| `OPERATOR_ADDRESS` | Your **wallet** `g1...` address from `gnokey list` |
| `VAL_PUBKEY` | `pub_key` from `cd /root/gno && gnoland secrets get validator_key` |
| `WALLETNAME` | Key name from `gnokey list` |

> ℹ️ After a successful transaction you can view your profile at:  
> https://test13.testnets.gno.land/r/gnops/valopers

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
  --chainid test-13 \
  --remote https://rpc.test13.testnets.gno.land \
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
  -remote "https://rpc.test13.testnets.gno.land" \
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
  -chainid "test-13" \
  -remote "https://rpc.test13.testnets.gno.land" \
  wallet
```

### Snapshot Server (Update)

```bash
sudo tar -C /root/gno/gnoland-data -cf - db \
  | sudo lz4 -f - /var/www/html/gnoland-db-snapshot.tar.lz4 \
  && sudo chown www-data:www-data /var/www/html/gnoland-db-snapshot.tar.lz4 \
  && sudo chmod 644 /var/www/html/gnoland-db-snapshot.tar.lz4
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

## Staying Updated

- Discord: [Gnoland Discord](https://discord.com/invite/S8nKUqwkPn)
- GitHub: [gnolang/gno](https://github.com/gnolang/gno)
- Official Docs: [docs.gno.land](https://docs.gno.land)
- Explorer: [test13.testnets.gno.land](https://test13.testnets.gno.land)

### Upgrade to a New Version

```bash
sudo systemctl stop gnoland

cd $HOME/gno
git fetch --all --tags
git checkout chain/test14
make install
make -C gno.land install.gnoland
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
