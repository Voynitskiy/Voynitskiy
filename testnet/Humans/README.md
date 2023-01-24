# Humans [lava-testnet-1]
![Humans Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Humans/Humans.png)
### Validator AlxVoy
* `Delegate` https://test.anode.team/humans/staking/humanvaloper19af3s7j3zvsd66ur4l4pmnd7y36n0xp3ykrrnj
* `Valoper` humanvaloper19af3s7j3zvsd66ur4l4pmnd7y36n0xp3ykrrnj
### Links
* `Website` - https://humans.ai/
* `Twitter` - https://twitter.com/humansdotai
* `Discord` - https://discord.gg/humansdotai
* `Medium` https://medium.com/humansdotai
* `Instagram` https://www.instagram.com/humansdotai
### RPC
* `RPC` https://humans.rpc.t.anode.team
### API
* `API` https://humans.api.t.anode.team
### Peers and seeds
* `Peer` 47004828307a0e0ab8a8a5388dd2e5350af0f7bb@65.109.93.152:51656
* `Live peers` https://anode.team/Humans/test/peers.txt
### Genesis and addrbook
* `Genesis` https://anode.team/Humans/test/genesis.json
* `Addrbook` https://anode.team/Humans/test/addrbook.json
### Explorer
* `ANODE.TEAM` https://test.anode.team/humans
### Auto installation script
```
wget https://anode.team/Humans/test/setup_humans.sh && chmod u+x setup_humans.sh && ./setup_humans.sh
```
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```shell
git clone https://github.com/humansdotai/humans
cd humans
git checkout v1
go build -o humansd cmd/humansd/main.go
mv humansd /root/go/bin/humansd
```
* Init
```
humansd init <moniker> --chain-id testnet-1
humansd config chain-id testnet-1
```

### Generate keys
```
humansd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://anode.team/Humans/test/genesis.json > ~/.humans/config/genesis.json
curl https://anode.team/Humans/test/addrbook.json > ~/.humans/config/addrbook.json
```
### Peers, seed
```
peers="1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.humans/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=Humans
After=network-online.target

[Service]
User=$USER
ExecStart=$(which humansd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable humansd
sudo systemctl restart humansd && journalctl -fu humansd -o cat
```
## Create Validator
```
humansd tx staking create-validator \
  --amount=10000000uheart \
  --pubkey=$(humansd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="testnet-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://humans.rpc.t.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop humansd && humansd tendermint unsafe-reset-all --home $HOME/.humans --keep-addr-book
```
```
peers="47004828307a0e0ab8a8a5388dd2e5350af0f7bb@65.109.93.152:51656"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.humans/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.humans/config/config.toml
```
```
sudo systemctl restart humansd && journalctl -u humansd -f
```
