# **Installation Guide for Namada Validator Node**

# **Introduction**

**Chain ID**: public-testnet-15.0dacadb8d663

**Hardware requirements**:

CPU: x86_64 or arm64, 8GB DDR4, 1TB of storage

# **First steps**

Update packages and install dependencies:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y make git-core libssl-dev pkg-config libclang-12-dev build-essential protobuf-compiler
sudo apt-get install unzip
```

Install **go**:

```bash
if ! [ -x "$(command -v go)" ]; then
  ver="1.20.5"
  cd $HOME
  wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"
  sudo rm -rf /usr/local/go
  sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"
  rm "go$ver.linux-amd64.tar.gz"
  echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile
  source ~/.bash_profile
fi
```

Check **go** version

```bash
go version
```

Install **Rust**:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
```

Check Rust version:

```bash
cargo --version
```

Install **Protocol** Buffers:

```bash
cd $HOME && rustup update
PROTOC_ZIP=protoc-23.3-linux-x86_64.zip
curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v23.3/$PROTOC_ZIP
sudo unzip -o $PROTOC_ZIP -d /usr/local bin/protoc
sudo unzip -o $PROTOC_ZIP -d /usr/local 'include/*'
rm -f $PROTOC_ZIP
```

Check **Protoc**:

```bash
protoc --version
```

Save and import variables into system

```bash
sed -i '/public-testnet/d' "$HOME/.bash_profile"
sed -i '/NAMADA_TAG/d' "$HOME/.bash_profile"
sed -i '/WALLET_ADDRESS/d' "$HOME/.bash_profile"
sed -i '/CBFT/d' "$HOME/.bash_profile"
```

Setting up vars:

```bash
echo "export NAMADA_TAG=v0.28.0" >> ~/.bash_profile
echo "export CBFT=v0.37.2" >> ~/.bash_profile
echo "export NAMADA_CHAIN_ID=public-testnet-15.0dacadb8d663" >> ~/.bash_profile
echo "export KEY_ALIAS=wallet" >> ~/.bash_profile
echo "export BASE_DIR=$HOME/.local/share/namada" >> ~/.bash_profile
```

Change the value of parameters

```bash
echo "export VALIDATOR_ALIAS=YOUR VALIDATOR ALIAS" >> ~/.bash_profile
echo "export EMAIL=your@email.com" >> ~/.bash_profile
source ~/.bash_profile
```

Install **CometBFT**:

```bash
cd $HOME && git clone https://github.com/cometbft/cometbft.git && cd cometbft && git checkout $CBFT
make build

cd $HOME && sudo cp $HOME/cometbft/build/cometbft /usr/local/bin/cometbft
```

Check **CometBFT** version:

```bash
cometbft version
```

# **Install Namada**

Download and build **Namada binaries**:

```bash
cd $HOME
rm -rf $HOME/namada
git clone https://github.com/anoma/namada
cd $HOME/namada
wget https://github.com/anoma/namada/releases/download/v0.28.0/namada-v0.28.0-Linux-x86_64.tar.gz
tar -xvf namada-v0.28.0-Linux-x86_64.tar.gz
rm namada-v0.28.0-Linux-x86_64.tar.gz
cd namada-v0.28.0-Linux-x86_64
sudo mv namada namadan namadac namadaw /usr/local/bin/
if [ ! -d "$HOME/.local/share/namada" ]; then
    mkdir -p "$HOME/.local/share/namada"
fi
```

Check **Namada** version:

```bash
namada --version
```

Create **Service** file:

```bash
sudo tee /etc/systemd/system/namadad.service > /dev/null <<EOF
[Unit]
Description=namada
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.local/share/namada
Environment=TM_LOG_LEVEL=p2p:none,pex:error
Environment=NAMADA_CMT_STDOUT=true
ExecStart=/usr/local/bin/namada node ledger run
StandardOutput=syslog
StandardError=syslog
Restart=always
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

Enable **Service**:

```bash
sudo systemctl daemon-reload
sudo systemctl enable namadad
```

# **Join-network**

## **Join-network as Pre-Genesis Validator**

IF YOU ARE NOT A PRE GEN VALIDATOR SKIP THIS SECTION

```bash
namada client utils join-network --chain-id $NAMADA_CHAIN_ID --genesis-validator $VALIDATOR_ALIAS
```

## **Join-network as Post-Genesis Validator**

IF YOU ARE A PRE GEN VALIDATOR SKIP THIS SECTION

**Create wallet:**

```bash
cd $HOME
namada wallet key gen --alias $KEY_ALIAS --unsafe-dont-encrypt
namada wallet address find --alias $KEY_ALIAS
```

Access [Namada’s Discord](https://discord.gg/KsZKs486) faucet to fund your wallet

Check your balance:

```bash
namada client balance --owner $ALIAS --token NAM
```

Init your validator:

```bash
cd $HOME
namada client init-validator \
--alias $VALIDATOR_ALIAS \
--account-keys $KEY_ALIAS \
--signing-keys $KEY_ALIAS \
--commission-rate 0.05 \
--max-commission-rate-change 0.01 \
--email $EMAIL \
--unsafe-dont-encrypt
```

Stake your funds:

```bash
namada client bond \
--validator $VALIDATOR_ALIAS \
--amount 1002 \
--gas-limit 10000000
```

Waiting 2 epoch and check your status:

```bash
namada client bonds --owner $VALIDATOR_ALIAS
```

# **Start Node**

## **Start Service and check logs:**

```bash
sudo systemctl start namadad && sudo journalctl -u namadad -f -o cat
```

## **Check full synchronization**

If `“catching_up”: false` , node is synchronized

```bash
curl http://127.0.0.1:26657/status | jq .result.sync_info.catching_up
```

# **Update Node**

Stop the node:

```bash
sudo systemctl stop namadad
```

Update:

```bash
cd namada
git pull
git checkout tomas/db-rm-recursion && git pull
make install
```

Restart node:

```bash
sudo systemctl restart namadad && sudo journalctl -u namadad -f -o cat
```

In case of an error when restarting the node, follow these steps:

```bash
sudo systemctl stop namadad.service
cp ~/.local/share/namada/public-testnet-15.0dacadb8d663/cometbft/data/priv_validator_state.json ~/priv_validator_state.jsonBAK
rm -rf ~/.local/share/namada/public-testnet-15.0dacadb8d663/db/
rm -rf ~/.local/share/namada/public-testnet-15.0dacadb8d663/cometbft/data/
mkdir -p ~/.local/share/namada/public-testnet-15.0dacadb8d663/cometbft/data
cp ~/priv_validator_state.jsonBAK ~/.local/share/namada/public-testnet-15.0dacadb8d663/cometbft/data/priv_validator_state.json
rm -rf ~/.local/share/namada/public-testnet-15.0dacadb8d663/tx_wasm_cache
rm -rf ~/.local/share/namada/public-testnet-15.0dacadb8d663/vp_wasm_cache
sudo systemctl restart namadad.service
```
