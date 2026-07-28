
[34 lines collapsed]

- [Step 5 — Initialize, Download Genesis and Config](#step-5--initialize-download-genesis-and-config)
- [Step 6 — Configure the Node](#step-6--configure-the-node)
- [Step 7 — Create Systemd Service](#step-7--create-systemd-service)
- [Step 8 — Sync Speed Note (No Snapshot Needed)](#step-8--sync-speed-note-no-snapshot-needed)
- [Step 8 — Sync Speed Note](#step-8--sync-speed-note)
- [Step 9 — Start the Node](#step-9--start-the-node)
- [Step 10 — Create a Wallet](#step-10--create-a-wallet)
- [Step 11 — Register as a Validator](#step-11--register-as-a-validator)
- [Useful Commands](#useful-commands)
- [Disaster Recovery — AppHash Mismatch & Snapshots](#disaster-recovery--apphash-mismatch--snapshots)
- [Snapshot Service](#snapshot-service)
- [Disaster Recovery — AppHash Mismatch](#disaster-recovery--apphash-mismatch)
- [Staying Updated](#staying-updated)
---

[23 lines collapsed]

| Active Validators | https://topaz.testnets.gno.land/r/sys/validators/v3 |
| Official Docs | https://docs.gno.land |
| GitHub | https://github.com/gnolang/gno |
| Hazen Snapshot | https://server-9.hazennetworksolutions.com/gnoland-topaz/index.json |
| Hazen Snapshot (browsable) | https://explorer.hazennetworksolutions.com/gnoland-testnet/snapshot/ |
---

[224 lines collapsed]

Unlike test13 (whose genesis replayed ~1.26M historical transactions and took days to sync through dense blocks), Topaz starts from a **fresh, empty ~2.7 MB genesis**. In practice a full sync from height 0 to chain tip takes well under an hour on a normal VPS once peers are connected (Step 6) — you don't need a snapshot just to get a new node running.
Where a snapshot *is* useful is disaster recovery — see [Disaster Recovery — AppHash Mismatch & Snapshots](#disaster-recovery--apphash-mismatch--snapshots) below.
Where a snapshot *is* useful is disaster recovery — see [Snapshot Service](#snapshot-service) below.
If you still want to check progress instead of waiting blind, see the sync-status command in [Step 9](#step-9--start-the-node) or [Useful Commands](#useful-commands).

[236 lines collapsed]

---
## Disaster Recovery — AppHash Mismatch & Snapshots
## Snapshot Service
### The AppHash mismatch crash
Hazen runs an automated, continuously-refreshed snapshot of our own Topaz full node's `db/`+`wal/` — the actual chain state — for anyone who needs to bootstrap or recover a node faster than a full resync from genesis.
| | |
|---|---|
| Contents | `db/` + `wal/` only (never `secrets/`, `config/`, or genesis) |
| Schedule | Every 6 hours |
| Retention | Last 3 snapshots always kept, oldest auto-deleted |
| Compression | `tar` + `lz4` |
| Manifest (JSON) | `https://server-9.hazennetworksolutions.com/gnoland-topaz/index.json` |
| Browsable page | [explorer.hazennetworksolutions.com/gnoland-testnet/snapshot](https://explorer.hazennetworksolutions.com/gnoland-testnet/snapshot/) — file name, block height, size, and SHA-256 for the current snapshot |
The manifest is a small JSON file describing the latest snapshot:
```json
{
  "generatedAt": "2026-07-28T09:17:03Z",
  "file": "gnoland-topaz-20260728-0917.tar.lz4",
  "url": "https://server-9.hazennetworksolutions.com/gnoland-topaz/gnoland-topaz-20260728-0917.tar.lz4",
  "blockHeight": 253012,
  "sizeBytes": 187342911,
  "sha256": "…",
  "compression": "tar+lz4",
  "contents": ["db", "wal"]
}
```
### One-line restore
```bash
curl -fsSL https://server-9.hazennetworksolutions.com/gnoland-topaz/restore.sh | bash
```
By default this assumes the paths from this guide (`$HOME/gno/gnoland-data`, systemd unit `gnoland`). If your setup differs:
```bash
DATA_DIR=/path/to/gnoland-data SERVICE=your-service-name \
  curl -fsSL https://server-9.hazennetworksolutions.com/gnoland-topaz/restore.sh | bash
```
The script:
1. Stops your node.
2. Backs up your **entire** `secrets/` and `config/` directories to `$HOME/gnoland-restore-backup-<timestamp>/`.
3. Replaces only `db/`+`wal/` with the snapshot.
4. Restores your `secrets/`+`config/` exactly as they were.
5. Restarts the node.
It never touches — and the snapshot itself never contains — anyone's validator key, so it's safe to run without any manual file juggling first.
> ⚠️ A snapshot from a *different* Topaz node (even another validator's) is fine to use for `db/`+`wal/` — it's the same chain state either way. Just never copy `secrets/` between two nodes; that's what actually causes a double-sign.
### Manual restore (equivalent, if you'd rather do it by hand)
```bash
sudo systemctl stop gnoland
cp -a $HOME/gno/gnoland-data/secrets /tmp/secrets.bak
cp -a $HOME/gno/gnoland-data/config /tmp/config.bak
rm -rf $HOME/gno/gnoland-data/db $HOME/gno/gnoland-data/wal
curl -L "$(curl -s https://server-9.hazennetworksolutions.com/gnoland-topaz/index.json | python3 -c 'import sys,json;print(json.load(sys.stdin)["url"])')" \
  | lz4 -dc - | tar -xf - -C $HOME/gno/gnoland-data
cp -a /tmp/secrets.bak/. $HOME/gno/gnoland-data/secrets/
cp -a /tmp/config.bak/. $HOME/gno/gnoland-data/config/
sudo systemctl restart gnoland
```
---
## Disaster Recovery — AppHash Mismatch
Topaz (and gno.land's Tendermint2 fork generally) has a known race condition — [gnolang/gno#6011](https://github.com/gnolang/gno/issues/6011) — where **concurrent RPC queries during block commit** can corrupt the app's state hash computation. The node stays up for a while, then on some later restart it panics on replay with an **AppHash mismatch**: the hash it recomputes for a past block no longer matches the hash recorded in that block's header. Once this happens the node is stuck in a crash loop — it cannot replay past the bad block, and there is no in-place fix.
**Symptoms:**
- `sudo journalctl -u gnoland -n 200 --no-pager` shows the node stuck at a fixed height for a long time ("chain stalled"), or
- after a restart it immediately panics with `wrong Block.Header.AppHash` (or similar) and keeps crash-looping on every `Restart=on-failure` retry.
**There is no repair for this — the `db/` must be replaced.** Your options are a full resync from genesis (safe, but can take time depending on chain height) or restoring a known-good `db/`+`wal/` from a recent snapshot (fast). Either way, **your validator identity is never at risk** as long as `secrets/` and `config/` survive — see below.
**There is no repair for this — the `db/` must be replaced.** Your options are a full resync from genesis (safe, but can take time depending on chain height) or restoring a known-good `db/`+`wal/` from the [Snapshot Service](#snapshot-service) above (fast — a few minutes instead of an hour-plus resync). Either way, **your validator identity is never at risk** as long as `secrets/` and `config/` survive — see the table below.
### What actually needs to survive

[5 lines collapsed]

| `secrets/priv_validator_state.json` | Last height/round this key signed | **Never.** Must stay paired with the key above. |
| `secrets/node_key.json` | P2P node identity | Not signing-critical, but keep it — changing it changes your node ID/peers. |
| `config/` | `config.toml`, `addrbook.json`, moniker, peers | Keep yours — a snapshot's config isn't tailored to your server (external IP, peers). |
| `db/`, `wal/` | The actual chain state and write-ahead log | **This is the only thing that gets corrupted, and the only thing a snapshot should ever touch.** |
| `db/`, `wal/` | The actual chain state and write-ahead log | **This is the only thing that gets corrupted, and the only thing a snapshot ever contains.** |
**Recommendation: keep an off-server backup of `secrets/` and `config/`** (e.g. `scp` them to your own machine periodically) independent of everything below — that backup is what protects you if the *server itself* is lost, not just the chain state. This is separate from and in addition to the snapshot mechanism below, which only ever contains `db/`+`wal/`.
**Recommendation: keep an off-server backup of `secrets/` and `config/`** (e.g. `scp` them to your own machine periodically), independent of the snapshot service above — that backup is what protects you if the *server itself* is lost, not just the chain state.
### Hazen's snapshot service
We run an automated snapshot of our own Topaz node's `db/`+`wal/` every 6 hours, keeping the last 3, published at:
- Manifest: `https://server-9.hazennetworksolutions.com/gnoland-topaz/index.json`
- Browsable: `https://explorer.hazennetworksolutions.com/gnoland-testnet/snapshot/`
To restore your own node from it:
```bash
curl -fsSL https://server-9.hazennetworksolutions.com/gnoland-topaz/restore.sh | bash
```
By default this assumes the paths from this guide (`$HOME/gno/gnoland-data`, systemd unit `gnoland`). If your setup differs:
```bash
DATA_DIR=/path/to/gnoland-data SERVICE=your-service-name \
  curl -fsSL https://server-9.hazennetworksolutions.com/gnoland-topaz/restore.sh | bash
```
The script stops your node, backs up your entire `secrets/` and `config/` directories to `$HOME/gnoland-restore-backup-<timestamp>/` first, replaces only `db/`+`wal/` with the snapshot, restores your `secrets/`+`config/` exactly as they were, and restarts the node. It never touches — and the snapshot itself never contains — anyone's validator key.
> ⚠️ A snapshot from a *different* chain's node (even another Topaz validator) is fine to use for `db/`+`wal/` — it's the same chain state either way. Just never copy `secrets/` between two nodes.
---
## Staying Updated

[31 lines collapsed]
