# DeFund [defund-private-4]
![DeFund Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/DeFund/DeFund.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/defund/staking/defundvaloper17fw3v7500qhd38ztjvjg40tjryng43ep22juc6
* `Valoper` defundvaloper17fw3v7500qhd38ztjvjg40tjryng43ep22juc6
### Links
* `Website` - https://www.defund.app/
* `Twitter` - https://twitter.com/defund_finance
* `Discord` - https://discord.gg/yQeBZBCQGc
* `Github` https://github.com/defund-labs/defund
* `Medium` https://medium.com/defund-finance
### RPC
* `RPC` https://defund.rpc.t.anode.team
### API
* `API` https://defund.api.t.anode.team
### Peers and seeds
* `Peer` 2c57fc71ccb618acd7823422afaa78bffb8550cd@65.109.93.152:31256
* `Live peers` https://anode.team/DeFund/test/peers.txt
### Genesis and addrbook
* `Genesis` https://anode.team/DeFund/test/genesis.json
* `Addrbook` https://anode.team/DeFund/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/defund
### Auto installation script
```
wget https://anode.team/DeFund/test/setup_defund.sh && chmod u+x setup_defund.sh && ./setup_defund.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/defund-labs/defund
cd defund
git checkout v0.2.2
make install
```
* Init
```
defundd init <moniker> --chain-id defund-private-4
defundd config chain-id defund-private-4
```

### Generate keys
```
defundd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/DeFund/test/genesis.json > ~/.defund/config/genesis.json
curl https://anode.team/DeFund/test/addrbook.json > ~/.defund/config/addrbook.json
```
### Peers, seed
```
peers="d837b7f78c03899d8964351fb95c78e84128dff6@174.83.6.129:30791,f03f3a18bae28f2099648b1c8b1eadf3323cf741@162.55.211.136:26656,f8fa20444c3c56a2d3b4fdc57b3fd059f7ae3127@148.251.43.226:56656,70a1f41dea262730e7ab027bcf8bd2616160a9a9@142.132.202.86:17000,e47e5e7ae537147a23995117ea8b2d4c2a408dcb@172.104.159.69:45656,74e6425e7ec76e6eaef92643b6181c42d5b8a3b8@defund-testnet-seed.itrocket.net:443"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.defund/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/defundd.service > /dev/null <<EOF
[Unit]
Description=Defund
After=network-online.target

[Service]
User=$USER
ExecStart=$(which defundd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable defundd
sudo systemctl restart defundd && journalctl -fu defundd -o cat
```
## Create Validator
```
defundd tx staking create-validator \
  --amount=100000000ufetf \
  --pubkey=$(defundd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="defund-private-4" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://defund.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop defundd && defundd tendermint unsafe-reset-all --home $HOME/.defund --keep-addr-book
```
```
peers="2c57fc71ccb618acd7823422afaa78bffb8550cd@65.109.93.152:31256"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.defund/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.defund/config/config.toml
```
```
curl -o - -L https://anode.team/DeFund/test/anode.team_defund_wasm.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.defund
```
```
sudo systemctl restart defundd && journalctl -fu defundd -o cat
```
