# Terp Network [athena-1]
![Terp Network Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Terp/TerpNetwork.png)
### Validator AlxVoy
* `Valoper` terpvaloper126jdyc74p70ahzrf3wvaka7hmeke8g2ntym70p
### Links testnet
* https://github.com/terpnetwork/terp-core
* https://github.com/terpnetwork/test-net
### Peers and seeds
* `Peer` 69dd5a6c7d11903a6198109576fa739a216ed92a@97.84.107.110:26656
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/terpnetwork/test-net/master/athena-1/genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Terp/addrbook.json
## Installation Steps
>Prerequisite: go1.19+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
```shell
git clone https://github.com/terpnetwork/terp-core.git
cd terp-core
git fetch --tags
git checkout v0.1.0
make build && make install
```
* Init
```
terpd init <moniker> --chain-id athena-1
terpd config chain-id athena-1
```

### Generate keys
```
terpd keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/terpnetwork/test-net/master/athena-1/genesis.json > ~/.terp/config/genesis.json
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Terp/addrbook.json > ~/.terp/config/addrbook.json
```
### Peers, seed
```
peers="69dd5a6c7d11903a6198109576fa739a216ed92a@97.84.107.110:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.terp/config/config.toml
```
* Minimum gas prices
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0125ucgas\"/;" ~/.terp/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/terpd.service > /dev/null <<EOF
[Unit]
Description=Terp Network
After=network-online.target

[Service]
User=$USER
ExecStart=$(which terpd) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable terpd
sudo systemctl restart terpd && journalctl -fu terpd -o cat
```
## Create Validator
```
terpd tx staking create-validator \
  --amount=1000000uterpx \
  --pubkey=$(terpd tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="athena-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --fees=2000upersyx \
  --from=<wallet_name>
```
