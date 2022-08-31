# Ki Chain [kichain-t-4]
### Links
* https://github.com/KiFoundation/ki-networks/tree/v0.1/Testnet/kichain-t-4
* https://github.com/KiFoundation/ki-networks/blob/v0.1/Testnet/kichain-t-4/UPGRADE_V3.md
* https://github.com/KiFoundation/ki-networks/blob/v0.1/Testnet/kichain-t-4/UPGRADE_t3_t4.md

## Installation Steps
>Prerequisite: go1.16+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
```shell
git clone https://github.com/KiFoundation/ki-tools.git
cd ki-tools
git checkout -b v3.0.0-beta tags/3.0.0-beta
make build-testnet
cp build/kid $HOME/go/bin/
```
* Init
```
kid init <moniker> --chain-id kichain-t-4
kid config chain-id kichain-t-4
```

### Generate keys
```
kid keys add <wallet_name>
```
### Genesis, peers, seed
```
curl https://raw.githubusercontent.com/KiFoundation/ki-networks/v0.1/Testnet/kichain-t-4/genesis.json > ~/.kid/config/genesis.json
```
```
seeds="381dff5439ed042353c5333e61bab1510711f2f5@seed-testnet.blockchain.ki:6969"
peers="46b25d81510f8dcc535ca0924961b266e4f59244@135.125.183.94:26656,ada3bbf64f963e764bfe003276354bd121e80ae0@95.111.248.200:26656,276f6fb420b3595b63c2a13d35868cb530a31578@65.21.159.19:26656,7e5710ee0b1576a78a21a89e1588b6c95ee69873@194.163.137.193:26656,323a5c9ccfb73573cbcd634c497b2a7405b198fa@142.132.137.114:26656"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.kid/config/config.toml
```
* Minimum gas prices
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025utki\"/;" ~/.kid/config/app.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/kid.service > /dev/null <<EOF
[Unit]
Description=Ki Chain Testnet
After=network-online.target

[Service]
User=$USER
ExecStart=$(which kid) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable kid
sudo systemctl restart kid && journalctl -fu kid -o cat
```
## Create Validator
```
kid tx staking create-validator \
  --amount=1000000utki \
  --pubkey=$(kid tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="kichain-t-4" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas-prices=0.025utki \
  --from=<wallet_name>
```

