# Lum [lum-network-1]
![Lum Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/Lum/Lum.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/lum-network/staking/lumvaloper1sqck02vl79khlwc86vvfjvacgtkck2hftvgm77
* `Valoper` lumvaloper1sqck02vl79khlwc86vvfjvacgtkck2hftvgm77
### Links
* `Website` - https://lum.network/
* `Twitter` - https://twitter.com/lum_network
* `Telegram` - https://t.me/lum_network
* `Discord` - https://discord.gg/mEjzKRkGZ4
* `CoinGecko` https://www.coingecko.com/en/coins/lum-network
* `Github` https://github.com/lum-network
* `Wallet` https://wallet.lum.network/
* `Documentation` https://docs.lum.network/
* `Linktree` https://linktr.ee/lum_network
* `Blog` https://medium.com/lum-network
### RPC
* `RPC` https://lum.rpc.m.anode.team
### API
* `API` https://lum.api.m.anode.team
### Peers and seeds
* `Peer` fc6f7914e4beb4b5278e7ba32ec2abde97cd8082@65.109.28.177:26626
* `Seeds` https://github.com/lum-network/mainnet/blob/master/seeds.txt
* `Seeds` https://github.com/lum-network/mainnet/blob/master/persistent_peers.txt
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/lum-network/mainnet/master/genesis.json
* `Addrbook` https://anode.team/Lum/main/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/lum-network
* `Lum` https://explorer.lum.network/
* `Mintscan` https://mintscan.io/lum
* `Ping` https://ping.pub/lum-network
* `Atomscan` https://atomscan.com/lum-network
## Installation Steps
>Prerequisite: go1.18+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current version.
```
git clone https://github.com/lum-network/chain.git lum
cd lum
git checkout v1.2.1
make install
```
* Init
```
lumd init <moniker> --chain-id lum-network-1
```

### Generate keys
```
lumd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/lum-network/mainnet/master/genesis.json > ~/.lumd/config/genesis.json
curl https://anode.team/Lum/main/addrbook.json > ~/.lumd/config/addrbook.json
```
### Peers, seed
```
seeds="19ad16527c98b782ee35df56b65a3a251bd99971@peer-1.mainnet.lum.network:26656"
peers="b47626b9d78ed7ed3c413304387026f907c70cbe@peer-0.mainnet.lum.network:26656,19ad16527c98b782ee35df56b65a3a251bd99971@peer-1.mainnet.lum.network:26656,5ea36d78ae774c9086c2d3fc8b91f12aa4bf3029@46.101.251.76:26656,a7f8832cb8842f9fb118122354fff22d3051fb83@3.36.179.104:26656,9afac13ba62fbfaf8d06867c30007162511093c0@54.214.134.223:26656,433c60a5bc0a693484b7af26208922b84773117e@34.209.132.0:26656,8fafab32895a31a0d7f17de58eddb492c6ced6d1@185.194.219.83:36656,c06eae3d9ea779710bca44e03f57e961b59d63f1@82.65.223.126:46656,4166de0e7721b6eec9c776abf2c38c40e7f820c5@202.61.239.130:26656,5a29947212a2615e43dac54deb55356a162e173a@35.181.76.160:26656,2cda4d97de0449878da10e456b176dd0720fbcec@62.171.129.174:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.lumd/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/lumd.service > /dev/null <<EOF
[Unit]
Description=Lum
After=network-online.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/
ExecStart=$(which lumd) start --p2p.laddr tcp://0.0.0.0:26656 --home /$HOME/.lumd
Restart=on-failure
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
systemctl daemon-reload && sudo systemctl enable lumd
sudo systemctl restart lumd && journalctl -fu lumd -o cat
```
## Create Validator
```
lumd tx staking create-validator \
  --amount=2500000000ulum \
  --pubkey=$(lumd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --commission-rate="0.03" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --node "tcp://127.0.0.1:26617" \
  --min-self-delegation="1" \
  --chain-id=lum-network-1 \
  --fees=200000ulum \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://lum.rpc.m.anode.team:443 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop lumd && lumd unsafe-reset-all --home $HOME/.lumd
```
```
peers="fc6f7914e4beb4b5278e7ba32ec2abde97cd8082@65.109.28.177:26626"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.lumd/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.lumd/config/config.toml
```
```
sudo systemctl restart lumd && journalctl -fu lumd -o cat
```
