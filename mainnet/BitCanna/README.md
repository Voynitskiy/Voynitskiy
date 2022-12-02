# BitCanna [bitcanna-1]
![BitCanna Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/mainnet/BitCanna/BitCanna.png)
### Validator AlxVoy
* `Delegate` https://main.anode.team/bitcanna/staking/bcnavaloper183gjgyuw9lq7dwjnhdxw9mgchrysqee0mptxsa
* `Valoper` bcnavaloper183gjgyuw9lq7dwjnhdxw9mgchrysqee0mptxsa
### Links
* `Website` - https://www.bitcanna.io/
* `Twitter` - https://twitter.com/BitcannaGlobal
* `Telegram` - https://t.me/BitcannaGlobal
* `Discord` - https://discord.gg/cnSHE7nKat
* `CoinGecko` https://www.coingecko.com/en/coins/bitcanna
* `Github` https://github.com/BitCannaGlobal
* `Github Community` https://github.com/BitCannaCommunity/
* `FAQ` https://www.bitcanna.io/faq/
* `Web Wallet` https://wallet.bitcanna.io/
* `Validator information` https://docs.bitcanna.io/
### Links mainnet
* https://github.com/BitCannaGlobal/bcna
* https://github.com/BitCannaGlobal/bcna/blob/main/last_upgrade.md
### RPC
* `RPC` https://bitcanna.rpc.m.anode.team
### API
* `API` https://bitcanna.api.m.anode.team
### Peers and seeds
* `Peer` 803fc66e3bd7b724921ef9c40636067f36e880c6@65.108.199.222:26357
* `Peers & seeds` https://github.com/BitCannaGlobal/bcna/blob/main/peers_seeds_and_services.md
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/BitCannaGlobal/bcna/main/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/BitCanna/addrbook.json
### Explorer
* `ANODE.TEAM` https://main.anode.team/bitcanna
* `BitCanna` https://cosmos-explorer.bitcanna.io/
* `Mintscan` https://www.mintscan.io/bitcanna
* `Ping` https://ping.pub/bitcanna
* `Atomscan` https://atomscan.com/bitcanna
## Installation Steps
>Prerequisite: go1.18.5+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
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
curl https://raw.githubusercontent.com/BitCannaGlobal/bcna/main/genesis.json > ~/.bcna/config/genesis.json
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/BitCanna/addrbook.json > ~/.bcna/config/addrbook.json
```
### Peers, seed
```
seeds="d6aa4c9f3ccecb0cc52109a95962b4618d69dd3f@seed1.bitcanna.io:26656,23671067d0fd40aec523290585c7d8e91034a771@seed2.bitcanna.io:26656"
peers="ecf729b2fb3c1038c55bd099b35c5d5b1d158c2b@178.62.236.228:26656,dcdc83e240eb046faabef62e4daf1cfcecfa93a2@159.65.198.245:26656,7c00beb4956bc40cd33ced6e2c2ffe07d4fa32e7@95.216.242.82:36656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.bcna/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/bcnad.service > /dev/null <<EOF
[Unit]
Description=BitCanna
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root/
ExecStart=$(which bcnad) start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
systemctl daemon-reload && systemctl enable bcnad
systemctl restart bcnad && journalctl -fu bcnad
```
## Create Validator
```
bcnad tx staking create-validator \
  --amount=1000000ubcna \
  --pubkey=$(bcnad tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="bitcanna-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas auto \
  --gas-adjustment 1.5 \
  --gas-prices 0.001ubcna \
  --from=<wallet_name>
```
### State-Sync
* start with State-Sync
```
SNAP_RPC=https://bitcanna.rpc.m.anode.team && \
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 2000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) && \
echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH
```
```
sudo systemctl stop bcnad && bcnad tendermint unsafe-reset-all --home $HOME/.bcna
```
```
peers="803fc66e3bd7b724921ef9c40636067f36e880c6@65.108.199.222:26357"
sed -i.bak -e  "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.bcna/config/config.toml
```
```
sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.bcna/config/config.toml
```
```
sudo systemctl restart bcnad && journalctl -fu bcnad -o cat
```
### SnapShot (2 times a day)
```
sudo systemctl stop bcnad
```
```
cp $HOME/.bcna/data/priv_validator_state.json $HOME/.bcna/priv_validator_state.json.backup
```
```
rm -rf $HOME/.bcna/data/
```
```
curl -o - -L https://anode.team/BitCanna/main/anode.team_bitcanna.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.bcna
```
```
mv $HOME/.bcna/priv_validator_state.json.backup $HOME/.bcna/data/priv_validator_state.json
```
```
sudo systemctl restart bcnad && journalctl -fu bcnad -o cat
```
