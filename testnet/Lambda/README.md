# Lambda [lambdatest_92001-2]
![Lambda Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Lambda/Lambda.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/lambda/staking/lambvaloper1r3zcl2du6jhqlt0qvkxgcmkpq38tnug8tjpyyz
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
* `RPC` https://lambda.rpc.t.anode.team
### API
* `API` https://lambda.api.t.anode.team
### Peers and seeds
* `Peer` 3da933459886bcf1435ec42aa92c87d4de6f1f95@65.109.28.177:45657
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/LambdaIM/testnets/main/lambdatest_92001-2/genesis.json
* `Addrbook` https://anode.team/Lambda/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/lambda
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
lambdavm init <moniker> --chain-id lambdatest_92001-2
lambdavm config chain-id lambdatest_92001-2
```

### Generate keys
```
lambdavm keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/LambdaIM/testnets/main/lambdatest_92001-2/genesis.json > ~/.lambdavm/config/genesis.json
curl https://anode.team/Lambda/test/addrbook.json > ~/.lambdavm/config/addrbook.json
```
### Peers, seed
```
seeds="ef150ce05782b05bd58ffc90a70d777d96482909@18.143.13.243:26656"
peers="ef150ce05782b05bd58ffc90a70d777d96482909@18.143.13.243:26656"
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
  --chain-id="lambdatest_92001-2" \
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
SNAP_RPC=https://lambda.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop lambdavm && lambdavm tendermint unsafe-reset-all --home $HOME/.lambdavm --keep-addr-book
```
```
peers="3da933459886bcf1435ec42aa92c87d4de6f1f95@65.109.28.177:45657"
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
sudo systemctl restart lambdavm && journalctl -u lambdavm -f
```
