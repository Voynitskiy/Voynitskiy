# BitSong [bitsong-2b]
### Validator AlxVoy
* `Delegate` https://ping.pub/bitsong/staking/bitsongvaloper138zp40g5pnjg4e7a24j5035q2f7nzwfec8ruee
* `Valoper` bitsongvaloper138zp40g5pnjg4e7a24j5035q2f7nzwfec8ruee
### Links
* `Website` - https://bitsong.io/
* `Twitter` - https://twitter.com/bitsongofficial
* `Telegram` - https://t.me/BitSongOfficial
* `Discord` - https://discord.gg/AhMuxkdEuW
* `Facebook` https://www.facebook.com/bitsongofficial
* `Reddit` https://www.reddit.com/r/bitsong
* `Github` https://github.com/bitsongofficial
* `Linkedin` https://www.linkedin.com/company/bitsong/

* `Blog` https://bitsongofficial.medium.com/
* `Bitsong Developer` https://docs.bitsong.io/
* `Wallet` https://wallet.bitsong.io/
* `Sinfonia` https://testnet.sinfonia.zone/
* `Sinfonia App` https://app.sinfonia.zone/ 

### Links mainnet
* https://github.com/bitsongofficial/networks/tree/master/bitsong-2
* https://github.com/bitsongofficial/networks/blob/master/bitsong-2/UPGRADE.md
### Delegation program
* https://github.com/bitsongofficial/delegation-program/blob/master/.github/ISSUE_TEMPLATE/application.yaml
### RPC
* `RPC` 65.108.12.222:26617
### Peers and seeds
* `Peer` b8a60ad6246ec986d29c1ab900032f3c78605b76@65.108.12.222:26616
* `Peers` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/BitSong/peers.txt
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/bitsongofficial/networks/master/bitsong-2b/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/BitSong/addrbook.json
### Explorer
* `Mintscan` https://www.mintscan.io/bitsong
* `Big Dipper` https://bitsong.bigdipper.live/
* `Atomscan` https://atomscan.com/bitsong
* `Ping` https://ping.pub/bitsong
## Installation Steps
>Prerequisite: go1.16+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
```shell
git clone https://github.com/bitsongofficial/go-bitsong
cd go-bitsong
git checkout v0.11.0
make install
```
* Init
```
bitsongd init <moniker> --chain-id bitsong-2b
bitsongd config chain-id bitsong-2b
```

### Generate keys
```
bitsongd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/bitsongofficial/networks/master/bitsong-2b/genesis.json > ~/.bitsongd/config/genesis.json
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/BitSong/addrbook.json > ~/.bitsongd/config/addrbook.json
```
### Peers, seed
```
peers="e2b9971222adf71f7199c670b7e85471c447e926@157.90.255.143:26656,120740c15a8a19c232b1aa4d80b20de248b33db3@135.181.129.94:26656,d741773bc5eecbefb7b14fcca5e3e0fedd49d5a3@157.90.95.104:26656,6e93a30587671e2cecacbcbb27092809bb20249f@95.217.203.59:31656,adfe1cf240780cf8d58266171ced72fb4e9a7a6d@23.226.14.168:26656,f36d3a926ae0583e60f00e7bc54711f3cb7fe769@195.201.58.166:26656,9c9f030298bdda9ca69de7db8e9a3aef33972fba@135.181.16.236:31656,9806602afb65ba45d1048d65285d5c6e50285088@178.18.242.242:26656,4fdd438ea70927003022ecc308e36bc1924ec598@51.210.104.207:26656,3cf3effd3ecb33bdbb5c5e6528c88fde4869b97c@116.202.139.113:26656,075cf589e44c74687ef3a4df3a583f482bce57e0@46.166.143.79:26656,f9d318eaf38988ce2b65b795068d86b214866c91@141.94.170.26:26256,fa932748b327fdde6d235b28a9850f8b8bd3326a@95.217.119.101:31656,d52f6e4fe1819133474e977d7e1d73124d1f4af5@95.217.156.76:26656,5ebab02914638005773dac8026f441e06c115a44@74.207.226.176:26656,e5428ce29ccd26434828a577906ac9c413ca6a48@80.71.57.42:26656,2afc435e2246ff3f16ade85b52264367945d12b5@176.58.124.226:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.bitsongd/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/bitsongd.service > /dev/null <<EOF
[Unit]
Description=BitSong
After=network-online.target

[Service]
User=$USER
ExecStart=$(which bitsongd) start
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable bitsongd
sudo systemctl restart bitsongd && journalctl -fu bitsongd -o cat
```
## Create Validator
```
bitsongd tx staking create-validator \
  --amount=271935384ubtsg \
  --pubkey=$(bitsongd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="bitsong-2b" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --fees="500ubtsg" \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=65.108.12.222:26617 && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop bitsongd && bitsongd tendermint unsafe-reset-all --home $HOME/.bitsongd
```
```
peers="b8a60ad6246ec986d29c1ab900032f3c78605b76@65.108.12.222:26616"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bitsongd/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bitsongd/config/config.toml
```
```
sudo systemctl restart bitsongd && journalctl -fu bitsongd -o cat
```
