<div align="center">

# 🌐 Gnoland Onyx Full Node & Validator Setup Guide

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

## Table of Contents

- [Important Version Rule](#important-version-rule)
- [Hardware Requirements](#hardware-requirements)
- [Network Endpoints](#network-endpoints)
- [Onyx Launch Facts](#onyx-launch-facts)
- [Step 1 — System Verification](#step-1--system-verification)
- [Step 2 — System Update and Dependencies](#step-2--system-update-and-dependencies)
- [Step 3 — Install Go](#step-3--install-go)
- [Step 4 — Get the Correct Binaries](#step-4--get-the-correct-binaries)
- [Step 5 — Download and Verify Genesis](#step-5--download-and-verify-genesis)
- [Step 6 — Initialize and Configure the Node](#step-6--initialize-and-configure-the-node)
- [Step 7 — Create Systemd Service](#step-7--create-systemd-service)
- [Step 8 — Start and Sync the Node](#step-8--start-and-sync-the-node)
- [Step 9 — Create a Wallet](#step-9--create-a-wallet)
- [Step 10 — Register as a Validator Candidate](#step-10--register-as-a-validator-candidate)
- [Useful Commands](#useful-commands)
- [Firewall](#firewall)
- [Backups and Key Safety](#backups-and-key-safety)
- [What's New Since Pearl](#whats-new-since-pearl)
- [Staying Updated](#staying-updated)

---

## Important Version Rule

Onyx is upgraded whenever mainnet is and is intended to rehearse mainnet releases one release candidate ahead. **Do not assume `v1.5.0` will remain current.**

Before every installation, restart after a coordinated halt, or binary upgrade, open the official ledger and use the version in its **last row**:

- [Onyx UPGRADES.md](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/UPGRADES.md)
- [Onyx upgrades.json](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/upgrades.json)

At launch, the last row is:

| Version | Commit | Start | Halt |
|---|---|---|---|
| `v1.5.0` | `e75fef82c02876a4df92ad6e325c5479b9532168` | Genesis | None |

> ⚠️ The `chain/onyx` release carries `genesis.json`, `genesis.json.gz`, and their checksums. **Binaries come from the version release** listed in `UPGRADES.md` — `v1.5.0` at launch — not from the chain release.

> ⚠️ Never run a validator from `master`, a branch tip, or a floating container tag. The binary and `GNOROOT` source checkout must use the same pinned version.

---

## Hardware Requirements

| Component | Minimum | Recommended |
|---|---|---|
| Operating System | Ubuntu 22.04+ | Ubuntu 24.04 LTS |
| CPU | 4 cores | 8 cores |
| RAM | 8 GB | 16 GB |
| Disk | 100 GB SSD | 250 GB NVMe SSD |
| Network | 100 Mbps, static IPv4 | 1 Gbps, static IPv4 |

Onyx began as a fresh chain with a roughly 2.96 MB genesis. Storage and sync requirements will grow with chain history, so leave sufficient headroom and monitor disk usage.

---

## Network Endpoints

| Type | Endpoint |
|---|---|
| Chain ID | `onyx-1` |
| Web / Explorer | https://onyx.testnets.gno.land |
| RPC | https://rpc.onyx.testnets.gno.land |
| WebSocket | wss://rpc.onyx.testnets.gno.land/websocket |
| Faucet | https://onyx.testnets.gno.land/faucet |
| Valoper Candidates | https://onyx.testnets.gno.land/r/gnops/valopers |
| Active Validators | https://onyx.testnets.gno.land/r/sys/validators/v0 |
| Gnockpit | https://gnockpit.onyx.testnets.gno.land |
| Status | https://status.onyx.testnets.gno.land |
| Seed 1 | seed-1.onyx.testnets.gno.land:26656 |
| Seed 2 | seed-2.onyx.testnets.gno.land:26656 |
| Official Docs | https://docs.gno.land |
| GitHub | https://github.com/gnolang/gno |

---

## Onyx Launch Facts

- **Launch:** September 28, 2026 at 00:00 UTC
- **Chain ID:** `onyx-1`
- **Launch binary:** `v1.5.0`, unchanged from mainnet's binary at that time
- **Binary commit:** `e75fef82c02876a4df92ad6e325c5479b9532168`
- **Genesis SHA-256:** `4b006fd7ccdec052865accc84dd29b2b76f8b57b2560789a15eedaa88f0e26c5`
- **Genesis packages:** 89 curated packages, matching mainnet's launch package set byte for byte
- **Initial namespaces:** `gnoswap`, `onbloc`, `moul`, `aeddi`, `aib`, `samcrew`, `howl`
- **Governance:** sole GovDAO T1 seed `aeddi`
- **Package policy:** inert after genesis; `gpao` is the approvals oracle
- **Founding validator:** `gno-core-validator-1`, power 60
- **Balances:** four operational testnet accounts plus exact fee-payer funding
- **Transfers:** open from genesis; no §126 lock or exemption list
- **Vesting:** none
- **Migration:** none — Pearl state and balances do not carry over

---

## Step 1 — System Verification

After connecting to the server, verify its resources and operating system:

```bash
lsb_release -a
uname -r
lscpu | grep -E "Model name|CPU\(s\)|Thread|Socket|Core"
free -h
df -h
ip -br address
```

Confirm that the server has a stable public IP and that port `26656/tcp` can be opened for P2P traffic.

---

## Step 2 — System Update and Dependencies

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git wget htop tmux build-essential jq make lz4 gcc unzip \
  screen btop iotop nethogs hdparm cmake perl automake autoconf libtool libssl-dev zstd pv
```

---

## Step 3 — Install Go

The `v1.5.0` source tree declares **Go 1.25.9**. Go is required when building from source and remains useful for operating the Gno toolchain. The example below installs Go 1.25.12:

```bash
cd $HOME
VER="1.25.12"
wget "https://go.dev/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"

[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
grep -qxF 'export PATH=/usr/local/go/bin:$HOME/go/bin:$PATH' ~/.bash_profile || \
  echo 'export PATH=/usr/local/go/bin:$HOME/go/bin:$PATH' >> ~/.bash_profile
grep -qxF 'export GNOROOT=$HOME/gno' ~/.bash_profile || \
  echo 'export GNOROOT=$HOME/gno' >> ~/.bash_profile
source ~/.bash_profile
mkdir -p "$HOME/go/bin"
```

Verify:

```bash
go version
```

Expected:

```text
go version go1.25.12 linux/amd64
```

> ℹ️ Operators using only official prebuilt binaries do not compile `gnoland`, but still need a checkout of the same release tag for `GNOROOT` because the node reads `gnovm/stdlibs` from it.

---

## Step 4 — Get the Correct Binaries

First check the latest row of Onyx's official upgrade ledger. The following commands use launch version `v1.5.0`; replace `VERSION` when the ledger advances.

```bash
VERSION="v1.5.0"
```

### Option A — Prebuilt binaries (recommended when host-compatible)

Download the official Linux AMD64 binaries and checksums from the **version release**:

> ⚠️ **Ubuntu 22.04 compatibility:** The official `v1.5.0` Linux AMD64 prebuilt was verified on September 29, 2026 to require `GLIBC_2.38`; stock Ubuntu 22.04 provides glibc 2.35 and exits with `GLIBC_2.38 not found`. Do **not** replace the system glibc on a multi-node server. Use [Option B](#option-b--build-from-source) with the exact pinned tag, or a separately named binary already built from that exact commit on the same host.

```bash
cd $HOME
wget "https://github.com/gnolang/gno/releases/download/${VERSION}/gnoland_linux_amd64"
wget "https://github.com/gnolang/gno/releases/download/${VERSION}/gnokey_linux_amd64"
wget "https://github.com/gnolang/gno/releases/download/${VERSION}/gno_linux_amd64"
wget "https://github.com/gnolang/gno/releases/download/${VERSION}/gnoweb_linux_amd64"
wget "https://github.com/gnolang/gno/releases/download/${VERSION}/CHECKSUMS.txt"

sha256sum -c CHECKSUMS.txt --ignore-missing
```

At `v1.5.0`, the Linux AMD64 checksums are:

```text
8dcff48228a881e398d238e3e14760c175c872fb164e85f21e5b4ee94a8b076d  gnoland_linux_amd64
878eb6599161f491a37fdcbd4214477ad28d5d6208f8428f0bffcd3115cd35b4  gnokey_linux_amd64
664d1605ef8b451c5eac7c98007bf1bb955b1bce12afe42da9c2684a988e6909  gno_linux_amd64
2e5cae80085df29994977b11d833b7d7e279ddfc763edc09ecf595a6035d83fc  gnoweb_linux_amd64
```

Install the binaries:

```bash
sudo install -m 0755 gnoland_linux_amd64 /usr/local/bin/gnoland
sudo install -m 0755 gnokey_linux_amd64 /usr/local/bin/gnokey
sudo install -m 0755 gno_linux_amd64 /usr/local/bin/gno
sudo install -m 0755 gnoweb_linux_amd64 /usr/local/bin/gnoweb
```

Clone the exact same version for `GNOROOT`:

```bash
rm -rf "$HOME/gno.new"
git clone --branch "$VERSION" --depth 1 https://github.com/gnolang/gno.git "$HOME/gno.new"

if [ -e "$HOME/gno" ]; then
  echo "$HOME/gno already exists; review it before replacing or moving it."
else
  mv "$HOME/gno.new" "$HOME/gno"
fi

export GNOROOT="$HOME/gno"
```

> ⚠️ Never delete an existing `$HOME/gno` directory without first checking whether it contains a running node's `gnoland-data`, secrets, configuration, or genesis.

### Option B — Build from source

Clone and build the exact version listed in `UPGRADES.md`:

```bash
cd $HOME
git clone --branch "$VERSION" --depth 1 https://github.com/gnolang/gno.git gno
cd gno

make -C gno.land install.gnoland install.gnokey
make install
```

Install the resulting binaries:

```bash
sudo install -m 0755 "$HOME/go/bin/gnoland" /usr/local/bin/gnoland
sudo install -m 0755 "$HOME/go/bin/gnokey" /usr/local/bin/gnokey
sudo install -m 0755 "$HOME/go/bin/gno" /usr/local/bin/gno
```

### Verify the installation

```bash
export GNOROOT="$HOME/gno"
gnoland version
gnokey version
```

`gnoland version` must print the exact pinned version, for example:

```text
v1.5.0
```

---

## Step 5 — Download and Verify Genesis

The chain release contains genesis artifacts, not the node binaries.

```bash
cd "$HOME/gno"
wget -O genesis.json \
  https://github.com/gnolang/gno/releases/download/chain/onyx/genesis.json
wget -O ONYX-CHECKSUMS.txt \
  https://github.com/gnolang/gno/releases/download/chain/onyx/CHECKSUMS.txt
```

Verify the genesis:

```bash
sha256sum genesis.json
```

Expected output:

```text
4b006fd7ccdec052865accc84dd29b2b76f8b57b2560789a15eedaa88f0e26c5  genesis.json
```

You can also verify against the downloaded manifest:

```bash
sha256sum -c ONYX-CHECKSUMS.txt --ignore-missing
```

> ⚠️ Stop immediately if the checksum does not match. Do not start a node with an unverified genesis file.

---

## Step 6 — Initialize and Configure the Node

> ⚠️ These initialization commands are for a fresh node. Do not run `secrets init` over an existing validator directory unless you intentionally want new validator and node identities.

From the pinned `GNOROOT` directory:

```bash
cd "$HOME/gno"
export GNOROOT="$HOME/gno"

gnoland config init
gnoland secrets init
```

Set your public node identity:

```bash
MONIKER="YOUR_MONIKER"
PUBLIC_IP="YOUR_SERVER_PUBLIC_IP"
```

Apply the required Onyx configuration:

```bash
cd "$HOME/gno"

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
gnoland config set p2p.external_address "${PUBLIC_IP}:26656"
gnoland config set p2p.persistent_peers \
  "g1x5mlj5ava0dw9vkf4j6admjlzswm6f06p44krn@seed-1.onyx.testnets.gno.land:26656,g1grq5zswt0dlwwe7clr4359w70k2ewgse0gcwck@seed-2.onyx.testnets.gno.land:26656"
```

Review the effective configuration:

```bash
gnoland config get moniker
gnoland config get p2p.external_address
gnoland config get p2p.persistent_peers
gnoland config get application.prune_strategy
```

> ℹ️ A standalone node should use `p2p.pex=true`. Operators using a sentry architecture should follow the official [sentry-node documentation](https://github.com/gnolang/gno/blob/master/gno.land/cmd/gnoland/README.md#sentry-node-architecture) instead of exposing the validator directly.

---

## Step 7 — Create Systemd Service

Create the service:

```bash
sudo tee /etc/systemd/system/gnoland.service > /dev/null <<'EOF'
[Unit]
Description=Gnoland onyx-1 Node
After=network-online.target
Wants=network-online.target

[Service]
User=root
WorkingDirectory=/root/gno
Environment=GNOROOT=/root/gno
Environment=HOME=/root
ExecStart=/usr/local/bin/gnoland start \
  --chainid onyx-1 \
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

> ⚠️ `--skip-genesis-sig-verification` is **required** on Onyx. Some genesis transactions contain placeholder or intentionally patched signatures, including the namespace-enforcement bootstrap call. The node can panic during genesis replay without this flag.

> ⚠️ Verify `WorkingDirectory`, `GNOROOT`, and the genesis path before starting. Adjust `/root/gno` consistently if you use a non-root operator or another installation path.

---

## Step 8 — Start and Sync the Node

Open P2P port `26656/tcp` before starting, then launch the service:

```bash
sudo systemctl restart gnoland
sudo systemctl status gnoland --no-pager
```

Follow logs:

```bash
sudo journalctl -u gnoland -f --no-hostname -o cat
```

Check synchronization:

```bash
curl -s http://127.0.0.1:26657/status | jq .result.sync_info
```

Expected when fully synchronized:

```json
{
  "latest_block_height": "XXXXXX",
  "catching_up": false
}
```

Check peers:

```bash
curl -s http://127.0.0.1:26657/net_info | jq '{n_peers: .result.n_peers, listening: .result.listening}'
```

Wait until `catching_up` is `false` before submitting a validator-candidate transaction.

### Historical upgrade halts

When joining from genesis after Onyx has completed one or more coordinated upgrades, replay reaches every historical halt. The node will stop at those heights. Consult `UPGRADES.md` and restart with the version required by each row. Never skip a halt blindly.

If a binary already satisfies an upgrade's `halt_min_version` but refuses to start in the short interval between proposal execution and the halt height, set `skip_upgrade_height` to that exact height for that one restart, or use the prior version until the halt. Remove temporary skip settings afterward.

---

## Step 9 — Create a Wallet

Create an operator wallet:

```bash
gnokey add wallet
```

> ⚠️ Save the mnemonic offline immediately. Do not keep the only copy on the node, in shell history, in Git, or in documentation.

Recover an existing wallet:

```bash
gnokey add wallet --recover
```

List accounts:

```bash
gnokey list
```

Request testnet GNOT from:

- https://onyx.testnets.gno.land/faucet

Verify the operator account balance:

```bash
gnokey query \
  -remote "https://rpc.onyx.testnets.gno.land" \
  auth/accounts/YOUR-G1-ADDRESS
```

> ℹ️ Onyx is a fresh chain. Pearl balances and state do not carry over, even when the same mnemonic is reused.

---

## Step 10 — Register as a Validator Candidate

Gnoland's Onyx validator onboarding is GovDAO-based. The following transaction creates a valoper candidate profile; it does **not** immediately add the node to the active validator set.

### Get the validator consensus public key

Run from the node's `GNOROOT` directory:

```bash
cd /root/gno
gnoland secrets get validator_key
```

Expected shape:

```json
{
  "address": "g1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "pub_key": "gpub1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

Use the `pub_key` value for registration. The consensus `address` is not the operator address. The operator address is the funded `g1...` account shown by `gnokey list`.

### Submit the candidate registration

The transaction must be signed by the operator key controlling the `OPERATOR_ADDRESS`:

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
  --chainid onyx-1 \
  --remote https://rpc.onyx.testnets.gno.land \
  --broadcast \
  WALLETNAME
```

| Placeholder | Description |
|---|---|
| `MONIKER` | Public validator display name |
| `DESCRIPTION` | Short operator description |
| `cloud\|on-prem\|data-center` | Infrastructure category |
| `OPERATOR_ADDRESS` | Funded wallet `g1...` address from `gnokey list` |
| `VAL_PUBKEY` | `gpub1...` from `gnoland secrets get validator_key` |
| `WALLETNAME` | Local key name from `gnokey list` |

After a successful transaction, review the profile at:

- https://onyx.testnets.gno.land/r/gnops/valopers

Registering only creates a candidate. A GovDAO member must create and pass a proposal through `r/sys/validators/v0` before the validator enters the active set:

- https://onyx.testnets.gno.land/r/sys/validators/v0

At genesis, Onyx has one founding validator (`gno-core-validator-1`) and one seeded GovDAO T1 member (`aeddi`).

### Update description (optional)

```bash
gnokey maketx call \
  --pkgpath gno.land/r/gnops/valopers \
  --func UpdateDescription \
  --args "YOUR-G1-OPERATOR-ADDRESS" \
  --args "YOUR-NEW-DESCRIPTION" \
  --gas-fee 1000000ugnot \
  --gas-wanted 50000000 \
  --chainid onyx-1 \
  --remote https://rpc.onyx.testnets.gno.land \
  --broadcast \
  WALLETNAME
```

---

## Useful Commands

### Service management

```bash
sudo systemctl start gnoland
sudo systemctl stop gnoland
sudo systemctl restart gnoland
sudo systemctl status gnoland --no-pager
```

### Logs and synchronization

```bash
# Live logs
sudo journalctl -u gnoland -f --no-hostname -o cat

# Last hour
sudo journalctl -u gnoland --since "1 hour ago" --no-hostname -o cat

# Sync status
curl -s http://127.0.0.1:26657/status | jq .result.sync_info

# Connected peers
curl -s http://127.0.0.1:26657/net_info | jq .result.n_peers
```

### Node identity

```bash
cd /root/gno
gnoland secrets get node_id
gnoland secrets get validator_key
gnoland secrets get
```

### Wallet and balance

```bash
gnokey list

gnokey query \
  -remote "https://rpc.onyx.testnets.gno.land" \
  auth/accounts/YOUR-G1-ADDRESS
```

### Send testnet GNOT

```bash
gnokey maketx send \
  -send "1000000ugnot" \
  -to "RECIPIENT_ADDRESS" \
  -gas-fee 1000000ugnot \
  -gas-wanted 10000000 \
  -broadcast \
  -chainid "onyx-1" \
  -remote "https://rpc.onyx.testnets.gno.land" \
  wallet
```

### Version and upgrade checks

```bash
gnoland version
cd /root/gno && git describe --tags --always
```

Always compare the result with the last row in the official Onyx `UPGRADES.md`.

---

## Firewall

For a standalone validator or full node:

```bash
# P2P — public and required
sudo ufw allow 26656/tcp comment "Gnoland Onyx P2P"
```

Keep RPC on `127.0.0.1:26657` unless you intentionally operate a hardened public RPC service. If a remote monitoring system needs access, use a private network or an authenticated reverse proxy rather than exposing validator RPC directly.

Verify:

```bash
sudo ufw status numbered
sudo ss -ltnp | grep -E ':26656|:26657'
```

---

## Backups and Key Safety

The chain database can be resynchronized. Validator identities cannot be reconstructed without their secrets.

Back up these items off-server and encrypt them:

```text
/root/gno/gnoland-data/secrets/
/root/gno/gnoland-data/config/
/root/gno/genesis.json
/root/gno/ONYX-CHECKSUMS.txt
```

Also keep the operator-wallet mnemonic offline, separately from the node backup.

Example archive creation:

```bash
cd /root/gno
tar -czf "onyx-validator-backup-$(date +%Y%m%d).tar.gz" \
  gnoland-data/secrets \
  gnoland-data/config \
  genesis.json \
  ONYX-CHECKSUMS.txt
chmod 600 onyx-validator-backup-*.tar.gz
```

Encrypt the archive before moving it off-server. Delete any unencrypted temporary archive only after confirming the encrypted copy is valid.

> ⚠️ Never start two nodes with the same validator key. Stop and verify the old node is offline before restoring validator secrets on a replacement server.

---

## What's New Since Pearl

Onyx is not another independent pre-mainnet test chain. It is the long-lived rehearsal network for mainnet behavior.

- **Fresh chain:** Onyx does not inherit Pearl packages, balances, names, or history.
- **Mainnet binary line:** launch uses unchanged `v1.5.0` mainnet binaries; later mainnet candidates are rehearsed on Onyx first.
- **Pinned upgrade ledger:** every binary, commit, halt height, and image is recorded in `UPGRADES.md` and `upgrades.json`.
- **Mainnet package set:** 89 curated packages at genesis, matching mainnet byte for byte at launch.
- **Mainnet namespaces:** the same seven initial names, with namespace enforcement active from block 1.
- **Mainnet governance shape:** sole GovDAO T1 seed `aeddi`, restricted `maketx run`, and `gpao` package approval.
- **Testnet economics:** faucet-funded GNOT, open transfers, no §126 lock, no exemption list, and no vesting.
- **Single founding validator:** `gno-core-validator-1`; later operators register as valoper candidates and require a GovDAO proposal.
- **Required genesis flag:** every node must start with `--skip-genesis-sig-verification`.

---

## Staying Updated

Primary sources:

- Upgrade ledger: [UPGRADES.md](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/UPGRADES.md)
- Machine-readable ledger: [upgrades.json](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/upgrades.json)
- Validator instructions: [VALIDATOR.md](https://github.com/gnolang/gno/blob/chain/mainnet/misc/deployments/onyx.gno.land/VALIDATOR.md)
- Chain release: [chain/onyx](https://github.com/gnolang/gno/releases/tag/chain%2Fonyx)
- Discord: [Gnoland Discord](https://discord.com/invite/S8nKUqwkPn)
- Status: [status.onyx.testnets.gno.land](https://status.onyx.testnets.gno.land)
- Gnockpit: [gnockpit.onyx.testnets.gno.land](https://gnockpit.onyx.testnets.gno.land)

### Upgrade to the next listed version

Do not upgrade merely because a newer tag exists. Wait for the Onyx ledger and coordinated-halt instructions.

When instructed, replace `NEXT_VERSION` with the exact version from the ledger:

```bash
NEXT_VERSION="vX.Y.Z-or-rc.N"

sudo systemctl stop gnoland
cd /root

wget -O gnoland_next \
  "https://github.com/gnolang/gno/releases/download/${NEXT_VERSION}/gnoland_linux_amd64"
wget -O CHECKSUMS-${NEXT_VERSION}.txt \
  "https://github.com/gnolang/gno/releases/download/${NEXT_VERSION}/CHECKSUMS.txt"

sha256sum gnoland_next
# Compare the result with CHECKSUMS-${NEXT_VERSION}.txt and upgrades.json.

chmod +x gnoland_next
./gnoland_next version
```

Update `GNOROOT` to the same tag without deleting the old checkout until the new binary is verified:

```bash
git clone --branch "$NEXT_VERSION" --depth 1 \
  https://github.com/gnolang/gno.git "/root/gno-${NEXT_VERSION}"
```

Back up the current binary, install the verified replacement, and update the systemd `WorkingDirectory` and `GNOROOT` paths together if you switch checkout directories:

```bash
sudo cp /usr/local/bin/gnoland "/usr/local/bin/gnoland.previous"
sudo install -m 0755 gnoland_next /usr/local/bin/gnoland
sudo systemctl daemon-reload
sudo systemctl start gnoland
sudo journalctl -u gnoland -f --no-hostname -o cat
```

Confirm the node resumes at the expected height. Keep the previous verified binary and checkout until the upgrade is stable.

---

## About the Author

This guide was prepared by **HazenNetworkSolutions**.  
🌐 [hazennetworksolutions.com](https://hazennetworksolutions.com)
