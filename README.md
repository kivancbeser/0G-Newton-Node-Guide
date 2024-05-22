# 0G-Newton-Node-Guide
0G Newton Node Guide

### Discord Role
Go to Discord and Take Roles
https://discord.gg/Fh5kCkjy

### FAUCET 
https://faucet.0g.ai/

### EXPLORER 
https://explorer.scannerx.net/0G-Testnet/staking

### REQUIREMENTS 

| COMPONENTS | MINIMUM REQUIREMENTS | 
| ------------ | ------------ |
| CPU |	4|
| RAM	| 8+ GB |
| Storage	| +500 GB SSD |

### UPDATE UPGRADE MACHINE
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget build-essential jq make lz4 gcc unzip -y
```

### INSTALLATION GO
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

### GENESIS JSON
```
rm ~/.0gchain/config/genesis.json
curl -Ls https://github.com/0glabs/0g-chain/releases/download/v0.1.0/genesis.json > $HOME/.0gchain/config/genesis.json
curl -Ls https://raw.githubusercontent.com/Core-Node-Team/Testnet-TR/main/0G-Newton/addrbook.json > $HOME/.0gchain/config/addrbook.json
```

### SET MINIMUM GAS PRICE
```
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0025ua0gi"|' $HOME/.0gchain/config/app.toml
```

### PEERS SEEDS
```
sed -i -e 's|^seeds *=.*|seeds = "c4d619f6088cb0b24b4ab43a0510bf9251ab5d7f@54.241.167.190:26656,44d11d4ba92a01b520923f51632d2450984d5886@54.176.175.48:26656,f2693dd86766b5bf8fd6ab87e2e970d564d20aff@54.193.250.204:26656"|' $HOME/.0gchain/config/config.toml
```

### PRUNING
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

### CREATE NEW WALLET
DON'T FORGET TO SAVE YOUR KEYS
```
0gchaind keys add wallet --eth
```

### RECOVERY WALLET
IF YOU REINSTALL NEW MACHINE WITH OLD WALLET USE THIS CODE FOR RECOVERY
```
0gchaind keys add wallet --eth --recover
```

### GET EVM ADDRESS
```
echo "0x$(0gchaind debug addr $(0gchaind keys show wallet -a) | grep hex | awk '{print $3}')"
```

### GET FAUCET 
```
https://faucet.0g.ai/
```

### WALLET LIST
```
0gchaind keys list
```

### CHECK SYNC
Before Create validator command, check your sync with this code. 
If shows false, run the create validator command.
```
0gchaind status 2>&1 | jq
```

### WALLET BALANCE CHECK
```
0gchaind q bank balances $(0gchaind keys show wallet -a)
```

### CREATE VALIDATOR (CHANGE MONIKERNAME)
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

### EDIT VALIDATOR 
```
0gchaind tx mstaking edit-validator \
--moniker "NEW MONIKER NAME" \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.15uinit \
-y
```

### VALIDATOR INFO
```
0gchaind tx mstaking validator $(0gchaind keys show wallet --bech val -a)
```

### DELEGATION TO YOUR VALIDATOR
```
0gchaind tx staking delegate $(0gchaind keys show wallet --bech val -a) 1000000ua0gi --from wallet -y
```

### UNJAIL
```
0gchaind tx slashing unjail --from wallet --gas=500000 --gas-prices=99999neuron -y
```

### RESTART SERVICE
```
sudo systemctl restart 0gchaind
```

### STOP SERVICE
```
sudo systemctl stop 0gchaind
```

### CHECK LOG
```
sudo journalctl -u 0gchaind -f -o cat
```

### PORT UPDATE
```
CUSTOM_PORT=110
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.0gchaind/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.0gchaind/config/app.toml
```


### REMOVING NODE
DON'T FORGET TO BACKUP YOUR PRIV VALIDATOR JSON KEY FROM CONFIG FOLDER.
```
sudo systemctl stop 0gchaind && sudo systemctl disable 0gchaind && sudo rm /etc/systemd/system/0gchaind.service && sudo systemctl daemon-reload && rm -rf $HOME/.0gchain && rm -rf 0g-chain && sudo rm -rf $(which 0gchaind) 
```

