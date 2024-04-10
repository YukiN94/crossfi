# How To Install Full Node Crossfi Testnet
**Setting up vars
Your Nodename (validator) that will shows in explorer**

`NODENAME=<Your_Nodename_Moniker>`

**Save variables to system**

`echo "export NODENAME=$NODENAME" >> $HOME/.bash_profile
if [ ! $WALLET ]; then
    echo "export WALLET=wallet" >> $HOME/.bash_profile
fi
echo "export CROSSFI_CHAIN_ID=crossfi-evm-testnet-1" >> $HOME/.bash_profile
source $HOME/.bash_profile`

**Update packages**

`sudo apt update && sudo apt upgrade -y`

**Install dependencies**

`sudo apt install curl build-essential git wget jq make gcc tmux net-tools ccze -y`


**Install go**

`if ! [ -x "$(command -v go)" ]; then
  ver="1.20.2"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi`

**Download and build binaries**

`cd $HOME
wget https://github.com/crossfichain/crossfi-node/releases/download/v0.3.0-prebuild3/crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz
tar -xvf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz
chmod +x $HOME/bin/crossfid
mv $HOME/bin/crossfid $HOME/go/bin
rm -rf crossfi-node_0.3.0-prebuild3_linux_amd64.tar.gz $HOME/bin`

**Init app**

`crossfid init $NODENAME --chain-id $CROSSFI_CHAIN_ID
rm -rf testnet ~/.mineplex-chain
git clone https://github.com/crossfichain/testnet.git
mv $HOME/testnet/ $HOME/.mineplex-chain/`


**Download configuration**

`wget  http://crossfi-toolkit.coinsspor.com/genesis.json -O $HOME/.mineplex-chain/config/genesis.json
wget  http://crossfi-toolkit.coinsspor.com/addrbook.json -O $HOME/.mineplex-chain/config/addrbook.json
`


**Set the minimum gas price**

`sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "10000000000000mpx"|g' $HOME/.mineplex-chain/config/app.toml`

**Disable indexing**

`sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.mineplex-chain/config/config.toml
`


**Config pruning**

`pruning="custom" && \
pruning_keep_recent="100" && \
pruning_keep_every="0" && \
pruning_interval="10" && \
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" ~/.mineplex-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" ~/.mineplex-chain/config/app.toml && \
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" ~/.mineplex-chain/config/app.toml && \
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" ~/.mineplex-chain/config/app.toml`


**Create service**

`sudo tee /etc/systemd/system/crossfid.service > /dev/null <<EOF
[Unit]
Description=Crossfi node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.mineplex-chain
ExecStart=$(which crossfid) start --home $HOME/.mineplex-chain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
`

**Register and start service**

`sudo systemctl daemon-reload`

`sudo systemctl enable crossfid`

`sudo systemctl restart crossfid && sudo journalctl -u crossfid -f -o cat
`


**Guidance for Validator**

`crossfid keys add $WALLET`


**To recover your wallet using seed phrase**

`crossfid keys add $WALLET --recover
`

**Show your wallet list**

`crossfid keys list`


**Save wallet info**

`
CROSSFI_WALLET_ADDRESS=$(crossfid keys show $WALLET -a)
CROSSFI_VALOPER_ADDRESS=$(crossfid keys show $WALLET --bech val -a)
echo 'export CROSSFI_WALLET_ADDRESS='${CROSSFI_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export CROSSFI_VALOPER_ADDRESS='${CROSSFI_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile`


**Create validator**


`crossfid tx staking create-validator \
  --amount 1000000mpx \
  --from $WALLET \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(crossfid tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $CROSSFI_CHAIN_ID
  --gas auto \
  --gas-adjustment 1.5 \
  --gas-prices 10000000000000mpx

`


**Edit validator**

crossfid tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$CROSSFI_CHAIN_ID \
  --from=$WALLET
  --gas auto \
  --gas-adjustment 1.5 \
  --gas-prices 10000000000000mpx



**Unjail validator**

`crossfid tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$CROSSFI_CHAIN_ID \
  --gas auto \
  --gas-adjustment 1.5 \
  --gas-prices 10000000000000mpx

`














