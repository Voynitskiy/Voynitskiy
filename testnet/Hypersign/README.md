# Hypersign [jagrat]
![Hypersign Guide](https://github.com/Voynitskiy/Voynitskiy/blob/main/testnet/Hypersign/Hypersign.png)
### Validator AlxVoy
* `Valoper` hidvaloper12v3uv7x0cg64vcystkqukwv24ecerew2xlyqtp
### Links testnet
* https://github.com/hypersign-protocol
### Peers and seeds
* `Peer` fc6f7914e4beb4b5278e7ba32ec2abde97cd8082@65.109.28.177:29226
### Genesis and addrbook
* `Genesis` https://raw.githubusercontent.com/hypersign-protocol/networks/master/testnet/jagrat/final_genesis.json
* `Addrbook` https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Hypersign/addrbook.json
## Installation Steps
>Prerequisite: go1.19+ required. [ref](https://golang.org/doc/install)

>Prerequisite: git. [ref](https://github.com/git/git)

* Fetch and install the current Testnet ki-tools version.
```shell
git clone https://github.com/hypersign-protocol/hid-node.git
cd terp-core
git checkout v0.1.2
make install
```
* Init
```
hid-noded init <moniker> --chain-id jagrat
```

### Generate keys
```
hid-noded keys add <wallet_name>
```
### Genesis, addrbook
```
curl https://raw.githubusercontent.com/hypersign-protocol/networks/master/testnet/jagrat/final_genesis.json > ~/.hid-node/config/genesis.json
curl https://raw.githubusercontent.com/Voynitskiy/Voynitskiy/main/mainnet/Hypersign/addrbook.json > ~/.hid-node/config/addrbook.json
```
### Peers, seed
```
peers="fc6f7914e4beb4b5278e7ba32ec2abde97cd8082@65.109.28.177:29226"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$seeds\"/; s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" ~/.hid-node/config/config.toml
```
### Create the service file
```
sudo tee /etc/systemd/system/hid-noded.service > /dev/null <<EOF
[Unit]
Description=Hypersign
After=network-online.target

[Service]
User=$USER
ExecStart=$(which hid-noded) start
Restart=always
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
* Load service and start
```
sudo systemctl daemon-reload && sudo systemctl enable hid-noded
sudo systemctl restart hid-noded && journalctl -fu hid-noded -o cat
```
## Create Validator
```
  hid-noded tx staking create-validator \
  --amount=1929630uhid \
  --pubkey=$(hid-noded tendermint show-validator) \
  --moniker="<moniker>" \
  --identity="<identity>" \
  --website="<website>" \
  --details="<details>" \
  --security-contact="<contact>" \
  --chain-id="jagrat" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<wallet_name>
```
