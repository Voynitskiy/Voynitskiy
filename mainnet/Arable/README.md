# Arable [bamboo_9052-1]
![Arable Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Arable/Arable.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/arable/staking/acrevaloper14pfm0zvwn58ctys273ua9vpj97gjlhf5u8h7nl
* `Valoper` acrevaloper14pfm0zvwn58ctys273ua9vpj97gjlhf5u8h7nl
### Links
* `Website` - https://app.arable.finance/
* `Twitter` - https://twitter.com/ArableProtocol
* `Telegram` - https://t.me/ArableProtocol
* `Discord` - https://discord.gg/bxFXCpk3hK
* `CoinGecko` https://www.coingecko.com/en/coins/arable-protocol
* `Github` https://github.com/ArableProtocol
* `Medium` https://medium.com/@ArableProtocol
* `FAQ` https://docs.google.com/document/d/14x1gPqb8PBmhbM_tKsN5EEjXFp5ZHLLl_DcIVfuR4F0/edit?usp=sharing
* `Litepaper` https://github.com/ArableProtocol/arableintro/blob/main/Arable_Litepaper.pdf
* `Documentation` https://about.arable.finance/
### RPC
* `RPC` https://arable.rpc.m.anode.team
### API
* `API` https://arable.api.m.anode.team
### Peers and seeds
* `Peer` e8d03dd64cdd5658167d0b1913c9fd1f05c80d9a@144.76.97.251:29966
* `Live peers` https://anode.team/Arable/main/peers.txt
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/ArableProtocol/acrechain/main/networks/mainnet/acre_9052-1/genesis.json
* `Addrbook` https://anode.team/Arable/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/arable
### Auto installation script
```
wget https://anode.team/Arable/main/setup_arable.sh && chmod u+x setup_arable.sh && ./setup_arable.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```
git clone https://github.com/ArableProtocol/acrechain
cd acrechain
git checkout v1.1.1
make install
```
* Init
```
acred init <moniker> --chain-id acre_9052-1
acred config chain-id acre_9052-1
```

### Generate keys
```
acred keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/ArableProtocol/acrechain/main/networks/mainnet/acre_9052-1/genesis.json > ~/.acred/config/genesis.json
curl https://anode.team/Arable/main/addrbook.json > ~/.acred/config/addrbook.json
```
### Peers, seed
```
peers="e2d029c95a3476a23bad36f98b316b6d04b26001@49.12.33.189:36656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.acred/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/acred.service > /dev/null <<EOF
[Unit]
Description=Arable Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which acred) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable acred
sudo systemctl restart acred && journalctl -fu acred -o cat
```
## Create Validator
```
acred tx staking create-validator \
  --amount=9999000000000000000aacre \
  --pubkey=$(acred tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="acre_9052-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas=300000 \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://arable.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop acred && acred tendermint unsafe-reset-all --home $HOME/.acred
```
```
peers="e8d03dd64cdd5658167d0b1913c9fd1f05c80d9a@144.76.97.251:29966"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.acred/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.acred/config/config.toml
```
```
sudo systemctl restart acred && journalctl -fu acred -o cat
```
