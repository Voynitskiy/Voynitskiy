# LikeCoin [likecoin-mainnet-2]
![LikeCoin Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/LikeCoin/LikeCoin.png)
### Validator AlxVoy
* `Delegate` https://explorer.voynitskiy.com/likecoin/staking/likevaloper10ff3y56q29yq7u0wmuchna8jv2rtkyclsmhv02
* `Valoper` likevaloper10ff3y56q29yq7u0wmuchna8jv2rtkyclsmhv02
### Links
* `Website` - https://like.co/
* `Twitter` - https://twitter.com/likecoin
* `Github` - https://github.com/likecoin
* `Discord` - https://discord.gg/likecoin
* `Blog` - https://blog.like.co/
* `Newsletter` - https://likecoin.substack.com/
### Links mainnet
* https://docs.like.co/validator/likecoin-chain-node/setup-a-node
### RPC
* `RPC` 65.109.28.177:29697
### Peers and seeds
* `Peer` fc6f7914e4beb4b5278e7ba32ec2abde97cd8082@65.109.28.177:29697
* `Seeds` 7a38dfc59eb43b27cf2cc87b46a43e76aeaaf012@20.205.224.107:26656,49976c3bd43da9271f226cbedf02d4b6b8fc880c@35.233.143.230:26656
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/likecoin/mainnet/master/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/LikeCoin/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/likecoin
* `Mintscan` https://www.mintscan.io/likecoin
* `Ping` https://ping.pub/likecoin
* `BigDipper` https://likecoin.bigdipper.live
* `Atomscan` https://atomscan.com/likecoin
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
wget https://github.com/likecoin/likecoin-chain/releases/download/v3.1.0/likecoin-chain_3.1.0_Linux_x86_64.tar.gz
tar -xvf likecoin-chain_3.1.0_Linux_x86_64.tar.gz
chmod +x bin/liked
mv bin/liked /usr/local/bin/
rm CHANGELOG.md LICENSE README.md likecoin-chain_3.1.0_Linux_x86_64.tar.gz
```
* Init
```
liked init <moniker> --chain-id likecoin-mainnet-2
```

### Generate keys
```
liked keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/likecoin/mainnet/master/genesis.json > ~/.liked/config/genesis.json
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/LikeCoin/addrbook.json > ~/.liked/config/addrbook.json
```
### Peers, seed
```
seeds="913bd0f4bea4ef512ffba39ab90eae84c1420862@34.82.131.35:26656,e44a2165ac573f84151671b092aa4936ac305e2a@nnkken.dev:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.liked/config/config.toml
```
* Minimum gas prices
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"1.0nanolike\"/;" ~/.liked/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/liked.service > /dev/null <<EOF
[Unit]
Description=LikeCoin
After=network-online.target

[Service]
User=$USER
ExecStart=$(which liked) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable liked
sudo systemctl restart liked && journalctl -fu liked -o cat
```
## Create Validator
```
liked tx staking create-validator \
  --amount=8673097086762nanolike \
  --pubkey=$(liked tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="likecoin-mainnet-2" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --node "tcp://127.0.0.1:26657" \
  --fees="200000nanolike" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=65.109.28.177:29697 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop liked && liked tendermint unsafe-reset-all --home $HOME/.liked
```
```
peers="fc6f7914e4beb4b5278e7ba32ec2abde97cd8082@65.109.28.177:29697"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.liked/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.liked/config/config.toml
```
```
sudo systemctl restart liked && journalctl -fu liked -o cat
```
