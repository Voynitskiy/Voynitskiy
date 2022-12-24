# Jackal [jackal-1]
![Jackal Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Jackal/Jackal.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/jackal/staking/jklvaloper15qdczgg53gxfqjspzsz0rllk08wm76elymkegk
* `Valoper` jklvaloper15qdczgg53gxfqjspzsz0rllk08wm76elymkegk
### Links
* `Website` - https://jackalprotocol.com/
* `Twitter` - https://twitter.com/Jackal_Protocol
* `Telegram` - https://t.me/+kyaQs5qFMF8zZDcx
* `Discord` - https://discord.com/invite/5GKym3p6rj
* `CoinGecko` https://www.coingecko.com/en/coins/jackal-protocol
* `Github` https://github.com/JackalLabs
* `Retriever name service` https://jackaldao.com/rns/
* `Jackkal docs` https://docs.jackaldao.com/
### RPC
* `RPC` https://jackal.rpc.m.anode.team
### API
* `API` https://jackal.api.m.anode.team
### Peers and seeds
* `Peer` e08efc0b0e15e4d8eacf0f4ed5e52f6e9bdc312d@144.76.97.251:36156
### Genesis and addrbook
* `Genesis` https://cdn.discordapp.com/attachments/1002389406650466405/1034968352591986859/updated_genesis2.json
* `Addrbook` https://anode.team/Jackal/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/jackal
* `Ping` https://ping.pub/jackal
## Installation Steps
>Prerequisite: go1.18.5+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/JackalLabs/canine-chain.git
cd canine-chain
git checkout v1.1.2-hotfix
make install
```
* Init
```
canined init <moniker> --chain-id jackal-1
canined config chain-id jackal-1
```

### Generate keys
```
canined keys add <wallet_name>
```
### Genesis, addrbook
```
wget -O ~/.canine/config/genesis.json https://cdn.discordapp.com/attachments/1002389406650466405/1034968352591986859/updated_genesis2.json
curl https://anode.team/Jackal/main/addrbook.json > ~/.canine/config/addrbook.json
```
### Peers, seed and min gas
```
PEERCOUNT=7
SEEDS=$(wget https://raw.githubusercontent.com/JackalLabs/canine-mainnet-genesis/master/genesis/seeds.txt -q -O -)
PEERS=`curl -sL https://raw.githubusercontent.com/JackalLabs/canine-mainnet-genesis/master/genesis/peers.txt | sort -R | head -n $PEERCOUNT | awk '{print $1}' | paste -s -d, -`
GAS="0.002ujkl"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.canine/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"$GAS\"/" $HOME/.canine/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/canined.service > /dev/null <<EOF
[Unit]
Description=Jackal
After=network-online.target

[Service]
User=$USER
ExecStart=$(which canined) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable canined
sudo systemctl restart canined && journalctl -fu canined -o cat
```
## Create Validator
```
canined tx staking create-validator \
  --amount=390363ujkl \
  --pubkey=$(canined tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="jackal-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --fees="400ujkl" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://jackal.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop canined && canined tendermint unsafe-reset-all --home $HOME/.canine --keep-addr-book
```
```
peers="e08efc0b0e15e4d8eacf0f4ed5e52f6e9bdc312d@144.76.97.251:36156"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.canine/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.canine/config/config.toml
```
```
cd $HOME/.canine/ && rm -r wasm && \
curl -o - -L https://anode.team/Jackal/main/anode.team_jackal_wasm.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.canine/
```
```
sudo systemctl restart canined && journalctl -fu canined -o cat
```
### SnapShot (2 times a day)
```
sudo systemctl stop bcnad && \
cp $HOME/.canine/data/priv_validator_state.json $HOME/.canine/priv_validator_state.json.backup && \
rm -rf $HOME/.canine/data/
```
```
curl -o - -L https://anode.team/Jackal/main/anode.team_jackal.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.canine
```
```
curl -o - -L https://anode.team/Jackal/main/anode.team_jackal_wasm.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.canine/
```
```
mv $HOME/.canine/priv_validator_state.json.backup $HOME/.canine/data/priv_validator_state.json && \
sudo systemctl restart canined && journalctl -fu canined -o cat
```
