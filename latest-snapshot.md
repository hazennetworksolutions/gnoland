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

---

## Option A — One-line restore (recommended)

```bash
curl -fsSL https://server-9.hazennetworksolutions.com/gnoland-sapphire/restore.sh | bash
```

If your setup differs from the default paths in [sapphire.md](sapphire.md):

```bash
DATA_DIR=/path/to/gnoland-data SERVICE=your-service-name \
  curl -fsSL https://server-9.hazennetworksolutions.com/gnoland-sapphire/restore.sh | bash
```

---

## Option B — Manual restore

```bash
sudo apt update
sudo apt install lz4 -y

sudo systemctl stop gnoland

cp -a $HOME/gno/gnoland-data/secrets /tmp/secrets.bak
cp -a $HOME/gno/gnoland-data/config /tmp/config.bak
rm -rf $HOME/gno/gnoland-data/db $HOME/gno/gnoland-data/wal

wget -O - https://server-9.hazennetworksolutions.com/gnoland-db-snapshot.tar.lz4 \
  | lz4 -d | tar -xf - -C $HOME/gno/gnoland-data

cp -a /tmp/secrets.bak/. $HOME/gno/gnoland-data/secrets/
cp -a /tmp/config.bak/. $HOME/gno/gnoland-data/config/

sudo systemctl start gnoland
```

> ⚠️ Never copy another node's `secrets/` onto yours — mixing validator keys between two live nodes causes a slashable double-sign.
