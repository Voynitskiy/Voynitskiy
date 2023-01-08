# Vidulum [vidulum-1]
![Vidulum Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Vidulum/Vidulum.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/vidulum/staking/vdlvaloper1cgq0vdp6vs5ggntjf93wy88e236ky634d2me7a
* `Valoper` vdlvaloper1cgq0vdp6vs5ggntjf93wy88e236ky634d2me7a
### Links
* `Website` - https://vidulum.app/
* `Twitter` - https://twitter.com/VidulumApp
* `Vidulum App` - https://wallet.vidulum.app/
* `Discord` - https://discord.gg/Z3xZjCHJRY
* `CoinGecko` https://www.coingecko.com/en/coins/vidulum
* `Github` https://github.com/vidulum
* `Reddit` https://www.reddit.com/r/VidulumOfficial/
* `YouTube` https://www.youtube.com/channel/UCNd92ZViZweu6zz5ydt_wrQ
* `Blog` https://vidulum.app/blog
* `Wallet` https://wallet.vidulum.app/
### Links mainnet
* https://github.com/vidulum/mainnet
### RPC
* `RPC` https://vidulum.rpc.m.anode.team
### API
* `API` https://vidulum.api.m.anode.team
### Peers and seeds
* `Peer` b63833b8b8740660ae3ac87f058447465f91f8f4@65.109.28.177:26726
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/BitCannaGlobal/bcna/main/genesis.json
* `Addrbook` https://anode.team/BitCanna/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/vidulum
* `Vidulum` https://explorers.vidulum.app/vidulum
* `Ping` https://ping.pub/vidulum
* `Atomscan` https://atomscan.com/vidulum
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```
git clone https://github.com/vidulum/mainnet vidulum
cd vidulum
make install
```
* Init
```
vidulumd init <moniker> --chain-id vidulum-1
vidulumd config chain-id vidulum-1
```

### Generate keys
```
vidulumd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/vidulum/mainnet/main/genesis.json > ~/.vidulum/config/genesis.json
curl https://anode.team/Vidulum/main/addrbook.json > ~/.vidulum/config/addrbook.json
```
### Peers, seed
```
seeds="883ec7d5af7222c206674c20c997ccc5c242b38b@ec2-3-82-120-39.compute-1.amazonaws.com:26656,eed11fff15b1eca8016c6a0194d86e4a60a65f9b@apollo.erialos.me:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.vidulum/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/vidulumd.service > /dev/null <<EOF
[Unit]
Description=Vidulum
After=network.target

[Service]
User=$USER
ExecStart=$(which vidulumd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=8192

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
systemctl daemon-reload && systemctl enable vidulumd
sudo systemctl restart vidulumd && journalctl -u vidulumd -f
```
## Create Validator
```
vidulumd tx staking create-validator \
  --amount="110000000uvdl" \
  --pubkey=$(vidulumd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="vidulum-1" \
  --commission-max-change-rate="0.1" \
  --commission-max-rate="0.3" \
  --commission-rate="0.10" \
  --min-self-delegation="1" \
  --fees 20000uvdl \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://vidulum.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop vidulumd && vidulumd unsafe-reset-all --home $HOME/.vidulum
```
```
peers="b63833b8b8740660ae3ac87f058447465f91f8f4@65.109.28.177:26726"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.vidulum/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.vidulum/config/config.toml
```
```
sudo systemctl restart vidulumd && journalctl -u vidulumd -f
