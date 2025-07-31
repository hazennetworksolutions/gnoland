<h1 align="center"> Gnoland Node Ubuntu Installation Version 7.2 </h1>

<img width="1920" height="1080" alt="gnoland" src="https://github.com/user-attachments/assets/fcb92812-05a8-46f5-933a-66393fff96c2" />
* [Gnoland Website](https://gno.land/)<br>
* [Gnoland Discord](https://discord.com/invite/S8nKUqwkPn)<br>
* [Gnoland Explorer](https://gnoscan.io/?type=custom&rpcUrl=https://rpc.test7.testnets.gno.land/&indexerUrl=)<br>


```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip screen btop iotop nethogs hdparm -y
sudo apt install -y curl git jq lz4 build-essential cmake perl automake autoconf libtool wget libssl-dev -y
```

```
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

```
git clone https://github.com/gnolang/gno.git
cd gno
git checkout tags/chain/test7.2
```

```
make install
```

```
make -C gno.land install.gnoland
make -C contribs/gnogenesis install
```

```
sudo cp /root/go/bin/gno /usr/local/bin/
sudo cp /root/go/bin/gnokey /usr/local/bin/
sudo cp /root/go/bin/gnodev /usr/local/bin/
sudo cp /root/go/bin/gnoland /usr/local/bin/
sudo cp /root/go/bin/gnogenesis /usr/local/bin/
```

```
sudo chmod +x /usr/local/bin/gno
sudo chmod +x /usr/local/bin/gnokey
sudo chmod +x /usr/local/bin/gnodev
sudo chmod +x /usr/local/bin/gnoland
sudo chmod +x /usr/local/bin/gnogenesis
```

```
gno version
```

### let's start and stop it locally first. 
```
gnoland start --lazy
```

```
rm -rf gnoland-data/
rm -f genesis.json
```

```
wget -O genesis.json https://gno-testnets-genesis.s3.eu-central-1.amazonaws.com/test7/genesis.json
shasum -a 256 genesis.json
```

```
gnoland secrets init
```

```
gnoland config init
```

### write each line one by one.

```
gnoland config set application.prune_strategy syncable
gnoland config set consensus.timeout_commit 3s
gnoland config set consensus.peer_gossip_sleep_duration 10ms
gnoland config set p2p.flush_throttle_timeout 10ms

gnoland config set p2p.persistent_peers "g137jz3hjhz6psrxxjtj5h7h4s6llfyrv2zxtfq3@gno-core-sen-01.test7.testnets.gno.land:26656,g1kpxll39mgzfhsepazzs0vne2l42mmkylxkt6un@gno-core-sen-02.test7.testnets.gno.land:26656"

gnoland config set p2p.seeds "g137jz3hjhz6psrxxjtj5h7h4s6llfyrv2zxtfq3@gno-core-sen-01.test7.testnets.gno.land:26656,g1kpxll39mgzfhsepazzs0vne2l42mmkylxkt6un@gno-core-sen-02.test7.testnets.gno.land:26656"

gnoland config set p2p.pex true
gnoland config set moniker "MONIKER"
gnoland config set mempool.size 10000
gnoland config set p2p.max_num_outbound_peers 40
gnoland config set telemetry.enabled false
gnoland config set p2p.laddr "tcp://0.0.0.0:26656"
gnoland config set rpc.laddr "tcp://127.0.0.1:26657"
```

### you can start it inside a screen session.
```
gnoland start --skip-genesis-sig-verification --genesis genesis.json
```

### if you want to create new wallet.
```
gnokey add wallet
```


### if you want to import the previous wallet first.
```
gnokey add hazenwallet --recover
```

```
gnoland secrets get node_id
```
{
    "id": "NODEID",
    "p2p_address": "P2PADRESS",
    "pub_key": "PUBKEY"
}

### when you're registering a valoper profile: the address you need to specify is the output of gnoland secrets get validator_key ----> the local validator key derived address the public key you need to specify (bech32) is the output of gnoland secrets get validator_key.

```
gnoland secrets get validator_key
```

{
    "address": "VALADRESS",
    "pub_key": "VALPUBKEY"
}


```
gnokey maketx call \
  -pkgpath "gno.land/r/gnoland/valopers" \
  -func "Register" \
  -gas-fee 1000000ugnot \
  -gas-wanted 30000000 \
  -broadcast \
  -chainid "test7.2" \
  -args "MONIKER" \
  -args "INFORMATION" \
  -args "VALADRESS" \
  -args "VALPUBKEY" \
  -remote "https://rpc.test7.testnets.gno.land:443" \
  WALLETNAME
```


