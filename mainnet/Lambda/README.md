# Lambda [lambda_92000-1]
![Lambda Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Lambda/Lambda.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/lambda/staking/lambvaloper1r3zcl2du6jhqlt0qvkxgcmkpq38tnug8tjpyyz
* `Valoper` lambvaloper1r3zcl2du6jhqlt0qvkxgcmkpq38tnug8tjpyyz
### Links
* `Website` - https://lambda.im/
* `Twitter` - https://twitter.com/Lambdaim
* `Discord` - https://discord.gg/lambdanetwork
* `CoinGecko` https://www.coingecko.com/en/coins/lambda
* `Github` https://github.com/LambdaIM
### Links mainnet
* https://github.com/LambdaIM/lambdavm
### RPC
* `RPC` https://lambda.rpc.m.anode.team
### API
* `API` https://lambda.api.m.anode.team
### Peers and seeds
* `Peer` 59b74a8b4870ab3f914ca4d9d9fd50e715cc0a58@144.76.97.251:45657
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/genesis.json
* `Addrbook` https://anode.team/Lambda/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/lambda
* `Ping` https://ping.pub/lambda
* `Atomscan` https://atomscan.com/lambda
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/LambdaIM/lambdavm.git
cd lambdavm
make build
mv $HOME/lambdavm/build/lambdavm /usr/local/bin
```
* Init
```
lambdavm init <moniker> --chain-id lambda_92000-1
lambdavm config chain-id lambda_92000-1
```

### Generate keys
```
lambdavm keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/LambdaIM/mainnet/main/lambda_92000-1/genesis.json > ~/.lambdavm/config/genesis.json
curl https://anode.team/Lambda/main/addrbook.json > ~/.lambdavm/config/addrbook.json
```
### Peers, seed
```
seeds="2c4f8e193fd10ab3a2bc919b484fd1c78eceffb3@13.213.214.88:26656"
peers="2c4f8e193fd10ab3a2bc919b484fd1c78eceffb3@13.213.214.88:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.lambdavm/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/lambdavm.service > /dev/null <<EOF
[Unit]
Description=Lambdavm
After=network-online.target

[Service]
User=$USER
ExecStart=$(which lambdavm) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable lambdavm
sudo systemctl restart lambdavm && journalctl -fu lambdavm -o cat
```
## Create Validator
```
lambdavm tx staking create-validator \
  --amount=19999990000000000000000ulamb \
  --pubkey=$(lambdavm tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="lambda_92000-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --gas="300000" \
  --gas-prices="0.025ulamb" \
  --min-self-delegation="1" \
  --broadcast-mode block \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://lambda.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop lambdavm && lambdavm tendermint unsafe-reset-all --home $HOME/.lambdavm --keep-addr-book
```
```
peers="59b74a8b4870ab3f914ca4d9d9fd50e715cc0a58@144.76.97.251:45657"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lambdavm/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.lambdavm/config/config.toml
```
```
sudo systemctl restart bcnad && journalctl -fu bcnad -o cat
```
