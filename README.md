# 0G-Newton-Node-Guide
0G Newton Node Guide

### Official Twitter 0G_Labs
https://x.com/0G_labs

### Discord Role
Go to Discord and Take Roles
https://discord.gg/Fh5kCkjy

### Faucet 
https://faucet.0g.ai/

### Explorer 
https://explorer.scannerx.net/0G-Testnet/staking

### Requirements 

| COMPONENTS | MINIMUM REQUIREMENTS | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| +500 GB SSD |

### Update Upgrade Machine
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget build-essential jq make lz4 gcc unzip -y
```

### Installation Go
```
cd $HOME && \
ver="1.22.0" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

### Clone project repository and Build Binary
```
cd && rm -rf 0g-chain
git clone https://github.com/0glabs/0g-chain
cd 0g-chain
git checkout v0.1.0
make install
```

### Node Configuration
```
0gchaind config chain-id zgtendermint_16600-1
0gchaind config keyring-backend test
0gchaind config node tcp://localhost:26657
```

### Change your Moniker Name
```
0gchaind init "MONIKER_NAME" --chain-id zgtendermint_16600-1
```

### Genesis
```
rm ~/.0gchain/config/genesis.json
curl -Ls https://github.com/0glabs/0g-chain/releases/download/v0.1.0/genesis.json > $HOME/.0gchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/0G-Newton/addrbook.json > $HOME/.0gchain/config/addrbook.json
```

### Set Minimum Gas Prices
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0025ua0gi"|' $HOME/.0gchain/config/app.toml
```

### Peers Seeds
```
PEERS="3285d00b18ffeceeeaaa68e7540d751849fc6b83@62.171.146.126:16656,e224629bf2b905628647b3cb725e20c2182e6e2f@158.220.114.24:26656,fba9b40beef7a2f9d5d10cd458df5499ad6706d7@138.201.201.96:26666,cbcc15defbfbcc0b55e0e66ebbd55dcccb954938@147.45.66.177:26656,b1b5a0999cb6e810886ceb95655e15308093bfe1@195.26.247.160:26656,0f2be6b6c5db0edc217b599e9f2f9800048c4394@37.27.91.167:26656,bf668d127a52b8543c3b5f2a3b01f8bb79eb05a7@109.199.112.123:26656,2cc75d1951d3d6172aee420b713c5b2153bd3402@185.103.103.77:26656,928f42a91548484f35a5c98aa9dcb25fb1790a70@65.109.155.238:26656,4c61ca4605bc6f9a114dcbec4e83c2ac28ead591@204.93.241.110:30397,3c7279de55529ef9a345632c77e66e8e4b0c518c@212.41.9.211:26656,d377f1bee9d0ba4c8d6a8ce3ba68d21803074355@5.78.97.34:26656,068ff19398729c6ab0b40ce5f71e1c7606d00e3e@193.233.75.53:26656,fb575bf0f3f09d927ffbba75ff86317ba4c8c0ec@75.119.151.217:16656"
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.0gchain/config/config.toml
sed -i -e 's|^seeds *=.*|seeds = "c4d619f6088cb0b24b4ab43a0510bf9251ab5d7f@54.241.167.190:26656,44d11d4ba92a01b520923f51632d2450984d5886@54.176.175.48:26656,f2693dd86766b5bf8fd6ab87e2e970d564d20aff@54.193.250.204:26656"|' $HOME/.0gchain/config/config.toml
```

### Pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.0gchain/config/app.toml
```

### Create a Service
```
sudo tee /etc/systemd/system/0gchaind.service > /dev/null << EOF
[Unit]
Description=0G node service
After=network-online.target
[Service]
User=$USER
ExecStart=$(which 0gchaind) start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### Daemon Reload and Start Service
```
sudo systemctl daemon-reload
sudo systemctl enable 0gchaind.service
sudo systemctl start 0gchaind.service
```

### Check the logs
```
sudo journalctl -u 0gchaind.service -f --no-hostname -o cat

```

### Create a new Wallet
DON'T FORGET TO SAVE YOUR KEYS
```
0gchaind keys add wallet --eth
```

### Recovery Wallet
IF YOU REINSTALL NEW MACHINE WITH OLD WALLET USE THIS CODE FOR RECOVERY
```
0gchaind keys add wallet --eth --recover
```

### Get Evm Address For Faucet
```
echo "0x$(0gchaind debug addr $(0gchaind keys show wallet -a) | grep hex | awk '{print $3}')"
```

### Get Fauce 
```
https://faucet.0g.ai/
```

### Wallet List
```
0gchaind keys list
```

### Check Sync
Before Create validator command, check your sync with this code. 
If shows false, run the create validator command.
```
0gchaind status 2>&1 | jq
```

### Wallet Balance Check
```
0gchaind q bank balances $(0gchaind keys show wallet -a)
```

### Create Validator (Change Moniker_Name)
```
0gchaind tx staking create-validator \
--amount=1000000ua0gi \
--pubkey=$(0gchaind tendermint show-validator) \
--moniker="Moniker_Name" \
--identity= \
--details="Details" \
--chain-id=zgtendermint_16600-1 \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1 \
--from=wallet \
--gas-prices=0.0025ua0gi \
--gas-adjustment=1.5 \
--gas=auto \
-y 
```
IF YOU ARE USING DIFFERNT PORT YOU NEED TO ADD THIS LINE CREATE VALIDATOR COMMAND. 
```
--node=http://localhost:PORTNUMBER \
```
FOR EXAMPLE:
--node=http://localhost:15657 \
!! PLEASE BACKUP YOUR PRIV VALID JSON

### Fill the Form
```
https://www.shorturl.at/fgkoR
```
### Edit Validator 
```
0gchaind tx mstaking edit-validator \
--moniker "NEW MONIKER NAME" \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.15uinit \
-y
```

### Validator Info
```
0gchaind tx mstaking validator $(0gchaind keys show wallet --bech val -a)
```

### Delegation
```
0gchaind tx staking delegate $(0gchaind keys show wallet --bech val -a) 1000000ua0gi --from wallet -y
```

### Unjail
```
0gchaind tx slashing unjail --from wallet --gas=500000 --gas-prices=99999neuron -y
```

### Restart Service
```
sudo systemctl restart 0gchaind
```

### Stop Service
```
sudo systemctl stop 0gchaind
```

### Check Log
```
sudo journalctl -u 0gchaind -f -o cat
```

### Port Update
```
CUSTOM_PORT=110
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.0gchaind/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.0gchaind/config/app.toml
```


### Removing Node
DON'T FORGET TO BACKUP YOUR PRIV VALIDATOR JSON KEY FROM CONFIG FOLDER.
```
sudo systemctl stop 0gchaind && sudo systemctl disable 0gchaind && sudo rm /etc/systemd/system/0gchaind.service && sudo systemctl daemon-reload && rm -rf $HOME/.0gchain && rm -rf 0g-chain && sudo rm -rf $(which 0gchaind) 
```

