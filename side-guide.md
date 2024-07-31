## **Install Dependencies**

Update system package and install build tools

```bash
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential fail2ban ufw
sudo apt -qy upgrade
```

## **Configure Moniker**

Replace <your-moniker-name> with your own validator name

```bash
MONIKER="<your-moniker-name>"
```

## **Install Go**

install go version 1.21.7

```bash
sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.21.7.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)
```

## **Build Binaries**

Cloning project repository & Compile binaries

```bash
cd $HOME
rm -rf side
git clone https://github.com/sideprotocol/side.git
cd side
git checkout v0.8.1
make build
```

Prepare binaries for cosmovisor

```bash
mkdir -p $HOME/.side/cosmovisor/genesis/bin
mv build/sided $HOME/.side/cosmovisor/genesis/bin/
rm -rf build
```

Create symlinks

```bash
sudo ln -s $HOME/.side/cosmovisor/genesis $HOME/.side/cosmovisor/current -f
sudo ln -s $HOME/.side/cosmovisor/current/bin/sided /usr/local/bin/sided -f
```

## **Cosmovisor Setup**

Install cosmovisor

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

## **Create Service**

Create a systemd service

```bash
sudo tee /etc/systemd/system/side.service > /dev/null << EOF
[Unit]
Description=Side node service
After=network-online.target
 
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.side"
Environment="DAEMON_NAME=sided"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.side/cosmovisor/current/bin"
 
[Install]
WantedBy=multi-user.target
EOF
```

## **Enable Service**

Enable side systemd service

```bash
sudo systemctl daemon-reloadsudo systemctl enable side
```

## **Initialize Node**

Setting node configuration

```bash
sided config chain-id S2-testnet-2	
sided config keyring-backend test
sided config node tcp://localhost:23857
```

Initialize node

```bash
sided init $MONIKER --chain-id S2-testnet-2
```

## **Download Genesis & Addrbook**

Download genesis & addrbook file

```bash
curl -Ls https://snap.wailchi.cfd/side-testnet/genesis.json > $HOME/.side/config/genesis.json
curl -Ls https://snap.wailchi.cfd/side-testnet/addrbook.json > $HOME/.side/config/addrbook.json
```

## **Configure Seeds**

Setting up a seed peers

```bash
sed -i -e "s|^seeds *=.*|seeds = \"d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.wailchi.cfd:23810\"|" $HOME/.side/config/config.toml
```

## **Configure Gas Prices**

Setting up a gas prices

```bash
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.005uside\"|" $HOME/.side/config/app.toml
```

## **Pruning Setting**

Configure pruning setting

```bash
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.side/config/app.toml
```

## **Custom Port**

Please note that updating the port is optional!

Configure a custom port

```bash
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:23858\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:23857\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:23860\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:23856\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":23866\"%" $HOME/.side/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:23817\"%; s%^address = \":8080\"%address = \":23880\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:23890\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:23891\"%; s%:8545%:23845%; s%:8546%:23846%; s%:6065%:23865%" $HOME/.side/config/app.toml
```

## **Download Snapshots**

Download latest chain snapshot

```bash
curl -L https://snap.wailchi.cfd/side-testnet/side-latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.side
[[ -f $HOME/.side/data/upgrade-info.json ]] && cp $HOME/.side/data/upgrade-info.json $HOME/.side/cosmovisor/genesis/upgrade-info.json
```

## **Start Service**

```bash
sudo systemctl start side
```
