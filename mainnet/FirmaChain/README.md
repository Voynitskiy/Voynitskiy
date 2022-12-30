# FirmaChain [colosseum-1]
![FirmaChain Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/FirmaChain/FirmaChain.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/firmachain/staking/firmavaloper1vnglxjwv6pc0ssz2zhsxsvfwehjddgeuusgfhu
* `Valoper` firmavaloper1vnglxjwv6pc0ssz2zhsxsvfwehjddgeuusgfhu
### Links
* `Website` - https://firmachain.org/
* `Twitter` - https://twitter.com/FirmaChain
* `CoinGecko` https://www.coingecko.com/en/coins/firmachain
* `Github` https://github.com/FirmaChain
* `Colosseum` https://github.com/FirmaChain/firmachain-testnet-colosseum
* `Docs` https://docs.firmachain.org/master
* `Station` https://station-colosseum.firmachain.dev/
### RPC
* `RPC` https://firmachain.rpc.m.anode.team
### API
* `API` https://firmachain.api.m.anode.team
### Peers and seeds
* `Peer` a94f70e215a429f4b479ff463183703c0b315d01@144.76.97.251:26117
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/FirmaChain/mainnet/main/colosseum-1/genesis.json
* `Addrbook` https://anode.team/FirmaChain/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/firmachain
* `FirmaChain` https://explorer-colosseum.firmachain.dev
* `Atomscan` https://atomscan.com/firmachain
## Installation Steps
>Prerequisite: go1.18.5+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/BitCannaGlobal/bcna.git
cd bcna && git checkout v1.5.3
make build
sudo mv $HOME/bcna/build/bcnad /usr/local/bin/bcnad
```
* Init
```
bcnad init <moniker> --chain-id bitcanna-1
bcnad config chain-id bitcanna-1
```

### Generate keys
```
bcnad keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/BitCannaGlobal/bcna/main/genesis.json > ~/.firmachain/config/genesis.json
curl https://anode.team/FirmaChain/main/addrbook.json > ~/.firmachain/config/addrbook.json
```
### Peers, seed
```
seeds="f89dcc15241e30323ae6f491011779d53f9a5487@mainnet-seed1.firmachain.dev:26656,04cce0da4cf5ceb5ffc04d158faddfc5dc419154@mainnet-seed2.firmachain.dev:26656,940977bdc070422b3a62e4985f2fe79b7ee737f7@mainnet-seed3.firmachain.dev:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.firmachain/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/firmachaind.service > /dev/null <<EOF
[Unit]
Description=FirmaChain Daemon
After=network-online.target

[Service]
User=$USER
ExecStart=$(which firmachaind) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable firmachaind
sudo systemctl restart firmachaind && journalctl -fu firmachaind -o cat
```
## Create Validator
```
firmachaind tx staking create-validator \
  --amount=103000000ufct \
  --pubkey=$(firmachaind tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="colosseum-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --fees="20000ufct" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://firmachain.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop firmachaind && firmachaind unsafe-reset-all --home $HOME/.firmachain
```
```
peers="a94f70e215a429f4b479ff463183703c0b315d01@144.76.97.251:26117"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.firmachain/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.firmachain/config/config.toml
```
```
sudo systemctl restart firmachaind && journalctl -fu firmachaind -o cat
```
