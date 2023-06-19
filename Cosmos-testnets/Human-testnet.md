# Guide for Node installation for Humans
### Links
- https://medium.com/humansdotai
- https://humans.ai/
- https://discord.com/invite/humansdotai

### Minimum hardware requirements
- Bandwidth: 1 Gbps Download / 100 Mbps Upload
- Memory: 8 GB RAM
- Disk: 250 GB SSD Storage
- CPU: Quad-Core

## Instalations
Install Go
```
sudo rm -rf /usr/local/go;
curl https://dl.google.com/go/go1.19.2.linux-amd64.tar.gz | sudo tar -C/usr/local -zxvf - ;
cat <<'EOF' >>$HOME/.profile
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
EOF
source $HOME/.profile
```
Check the version of Go
```
go version
```
Update apt and install packeges
```
sudo apt update && sudo apt full-upgrade -y
sudo apt list --upgradable
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```
Install Humans
```bash
cd \
git clone https://github.com/humansdotai/humans \
cd humans \
git checkout v1.0.0 \
go build -o humansd cmd/humansd/main.go \
mv humansd /usr/local/go/bin
```
### Node Initialization
```
moniker=<YOUR_MONIKER_NAME>
humansd config chain-id testnet-1
humansd init $moniker --chain-id=testnet-1
```
Genesis and adderss book
```
curl -s https://rpc-testnet.humans.zone/genesis | jq -r .result.genesis > genesis.json
mv genesis.json $HOME/.humans/config
wget -O $HOME/.humans/config/addrbook.json "https://raw.githubusercontent.com/StakeTake/guidecosmos/main/Humans/testnet-1/addrbook.json"
```
Peers, seeds, and minimum-gas-prices
```
PEERS="1df6735ac39c8f07ae5db31923a0d38ec6d1372b@45.136.40.6:26656,9726b7ba17ee87006055a9b7a45293bfd7b7f0fc@45.136.40.16:26656,6e84cde074d4af8a9df59d125db3bf8d6722a787@45.136.40.18:26656,eda3e2255f3c88f97673d61d6f37b243de34e9d9@45.136.40.13:26656,4de8c8acccecc8e0bed4a218c2ef235ab68b5cf2@45.136.40.12:26656"
seeds=""
sed -i 's|^persistent_peers *=.*|persistent_peers = "'$PEERS'"|' $HOME/.humans/config/config.toml
sed -i 's/minimum-gas-prices =.*/minimum-gas-prices = "0.025uheart"/g' $HOME/.humans/config/app.toml
```
Config `config.toml`
```
CONFIG_TOML="$HOME/.humans/config/config.toml"
sed -i 's/timeout_propose =.*/timeout_propose = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_propose_delta =.*/timeout_propose_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote =.*/timeout_prevote = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_prevote_delta =.*/timeout_prevote_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit =.*/timeout_precommit = "100ms"/g' $CONFIG_TOML
sed -i 's/timeout_precommit_delta =.*/timeout_precommit_delta = "500ms"/g' $CONFIG_TOML
sed -i 's/timeout_commit =.*/timeout_commit = "1s"/g' $CONFIG_TOML
sed -i 's/skip_timeout_commit =.*/skip_timeout_commit = false/g' $CONFIG_TOML
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.humans/config/config.toml
```
### Pruning settings
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="50"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.humans/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.humans/config/app.toml
```
Service file
```
sudo tee /etc/systemd/system/humansd.service > /dev/null <<EOF
[Unit]
Description=humans node
After=network.target
[Service]
User=root
Type=simple
ExecStart=$(which humansd) start --home $HOME/.humans
Restart=on-failure
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
Run
```
sudo systemctl daemon-reload && \
sudo systemctl enable humansd && \
sudo systemctl start humansd && \
systemctl status humansd && \
humansd status 2>&1 | jq .SyncInfo && \
sudo journalctl -u humansd -f
```



