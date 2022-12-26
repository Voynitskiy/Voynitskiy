# OKP4 [okp4-nemeton-1]
![OKP4 Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/BitCanna/OKP4.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/opk4/staking/okp4valoper1329p0ltpd6meegdax4q3s3du8aayge2xza6e4d
* `Valoper` okp4valoper1329p0ltpd6meegdax4q3s3du8aayge2xza6e4d
### Links
* `Website` -  https://okp4.network/
* `Twitter` - https://twitter.com/OKP4_Protocol
* `Telegram` - https://t.me/okp4network
* `Discord` - https://discord.com/invite/okp4
* `Github` https://github.com/okp4
* `Medium` https://blog.okp4.network/
* `LinkedIn` https://www.linkedin.com/company/okp4-open-knowledge-platform-for
* `Whitepaper` https://docs.okp4.network/docs/whitepaper/abstract
* `Nodes & Validator Guide` https://docs.okp4.network/docs/nodes/introduction
* `Faucet` https://faucet.okp4.network/
### RPC
* `RPC` https://okp4.rpc.t.anode.team
### API
* `API` https://okp4.api.t.anode.team
### Peers and seeds
* `Peer` ee4c5d9a8ac7401f996ef9c4d79b8abda9505400@144.76.97.251:12656
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/okp4/networks/main/chains/nemeton-1/genesis.json
* `Addrbook` https://anode.team/OKP4/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/opk4
## Installation Steps
>Prerequisite: go1.18.2+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/okp4/okp4d.git
cd okp4d
make install
```
* Init
```
okp4d init <moniker> --chain-id okp4-nemeton-1
okp4d config chain-id okp4-nemeton-1
```

### Generate keys
```
okp4d keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/okp4/networks/main/chains/nemeton-1/genesis.json > ~/.okp4d/config/genesis.json
curl https://anode.team/OKP4/test/addrbook.json > ~/.okp4d/config/addrbook.json
```
### Peers, seed
```
peers="f595a1386d5ca2e0d2cd81d3c6372c3bf84bbd16@65.109.31.114:2280,a49302f8999e5a953ebae431c4dde93479e17155@162.19.71.91:26656,dc14197ed45e84ca3afb5428eb04ea3097894d69@88.99.143.105:26656,79d179ea2e1fbdcc0c59a95ab7f1a0c48438a693@65.108.106.131:26706,501ad80236a5ac0d37aafa934c6ec69554ce7205@89.149.218.20:26656,5fbddca54548bf125ee96bb388610fe1206f087f@51.159.66.123:26656,769f74d3bb149216d0ab771d7767bd39585bc027@185.196.21.99:26656,024a57c0bb6d868186b6f627773bf427ec441ab5@65.108.2.41:36656,fff0a8c202befd9459ff93783a0e7756da305fe3@38.242.150.63:16656,2bfd405e8f0f176428e2127f98b5ec53164ae1f0@142.132.149.118:26656,bf5802cfd8688e84ac9a8358a090e99b5b769047@135.181.176.109:53656,dc9a10f2589dd9cb37918ba561e6280a3ba81b76@54.244.24.231:26656,085cf43f463fe477e6198da0108b0ab08c70c8ab@65.108.75.237:6040,803422dc38606dd62017d433e4cbbd65edd6089d@51.15.143.254:26656,b8330b2cb0b6d6d8751341753386afce9472bac7@89.163.208.12:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.okp4d/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/okp4d.service > /dev/null <<EOF
[Unit]
Description=okp4d
After=network-online.target

[Service]
User=$USER
ExecStart=$(which okp4d) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable okp4d
sudo systemctl restart okp4d && journalctl -fu okp4d -o cat
```
## Create Validator
```
okp4d tx staking create-validator \
  --amount=7000000uknow \
  --pubkey=$(okp4d tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="okp4-nemeton" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://okp4.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop okp4d && okp4d tendermint unsafe-reset-all --home $HOME/.okp4d --keep-addr-book
```
```
peers="ee4c5d9a8ac7401f996ef9c4d79b8abda9505400@144.76.97.251:12656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.okp4d/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.okp4d/config/config.toml
```
```
sudo systemctl restart okp4d && journalctl -u okp4d -f
```
