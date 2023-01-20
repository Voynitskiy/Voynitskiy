# Uptick Network [uptick_7000-2]
![Uptick Network Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Uptick/Uptick.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/uptick/staking/uptickvaloper1tztp0sp047zxxj2j3pzhtr67w2nmn2a8q0dxse
* `Valoper` uptickvaloper1tztp0sp047zxxj2j3pzhtr67w2nmn2a8q0dxse
### Links
* `Website` - https://uptick.network/ 
* `Twitter` - https://twitter.com/uptickproject
* `Telegram` - https://t.me/uptickproject
* `Discord` - https://discord.gg/5rVycYfNzw
* `Github` https://github.com/UptickNetwork
* `Medium` https://medium.com/@uptickproject
* `Reddit` https://www.reddit.com/r/UptickNetwork/
* `Uptick NFT Marketplace on Cosmos-IRIS` https://www.upticknft.com/
* `Uptick NFT Marketplace on Loopring` https://loopring.upticknft.com/
### RPC
* `RPC` https://uptick.rpc.t.anode.team
### API
* `API` https://uptick.api.t.anode.team
### Peers and seeds
* `Peer` 78fa616bb67efd86e48529fde26309681ee213b6@65.108.199.222:26636
* `Live peers` https://anode.team/Uptick/test/peers.txt
### Genesis and addrbook
* `Genesis` https://anode.team/Uptick/test/genesis.json
* `Addrbook` https://anode.team/Uptick/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/uptick
### Auto installation script
```
wget https://anode.team/Uptick/test/setup_uptick.sh && chmod u+x setup_uptick.sh && ./setup_uptick.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/UptickNetwork/uptick-testnet
cd uptick-testnet
git checkout v0.2.4
make install
```
* Init
```
uptickd init <moniker> --chain-id uptick_7000-2
uptickd config chain-id uptick_7000-2
```

### Generate keys
```
uptickd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/Uptick/test/genesis.json > ~/.uptickd/config/genesis.json
curl https://anode.team/Uptick/test/addrbook.json > ~/.uptickd/config/addrbook.json
```
### Peers, seed
```
peers="eecdfb17919e59f36e5ae6cec2c98eeeac05c0f2@peer0.testnet.uptick.network:26656,178727600b61c055d9b594995e845ee9af08aa72@peer1.testnet.uptick.network:26656,f97a75fb69d3a5fe893dca7c8d238ccc0bd66a8f@uptick-seed.p2p.brocha.in:30554,94b63fddfc78230f51aeb7ac34b9fb86bd042a77@uptick-testnet-rpc.p2p.brocha.in:30556,902a93963c96589432ee3206944cdba392ae5c2d@65.108.42.105:27656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.uptickd/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/uptickd.service > /dev/null <<EOF
[Unit]
Description=Uptick Network
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/
ExecStart=$(which uptickd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
systemctl daemon-reload && systemctl enable uptickd
systemctl restart uptickd && journalctl -n 100 -f -u uptickd
```
## Create Validator
```
uptickd tx staking create-validator \
  --amount=10000000000000000000auptick \
  --pubkey=$(uptickd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="300000" \
  -y \
  -b block \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://uptick.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop uptickd && uptickd tendermint unsafe-reset-all --home $HOME/.uptickd --keep-addr-book
```
```
peers="2c57fc71ccb618acd7823422afaa78bffb8550cd@65.109.93.152:31256"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.uptickd/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.uptickd/config/config.toml
```
```
sudo systemctl restart uptickd && journalctl -fu uptickd -o cat
```
