# Tellor Layer Mainnet Node Kurulum Rehberi

## Sistem Gereksinimleri

- **İşletim Sistemi**: Ubuntu 20.04/22.04 LTS
- **CPU**: 8+ cores
- **RAM**: 32 GB + 16 GB swap
- **Depolama**: 1 TB NVMe SSD
- **Go**: 1.23.3

## 1. Sistem Hazırlığı

```bash
# Sistem güncellemesi
sudo apt update && sudo apt upgrade -y

# Gerekli paketleri kur
sudo apt install -y build-essential jq sed curl wget git make gcc g++ unzip snapd lz4
```

## 2. Go Kurulumu

```bash
cd $HOME
wget https://go.dev/dl/go1.23.3.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.23.3.linux-amd64.tar.gz
rm go1.23.3.linux-amd64.tar.gz

# PATH ayarları
cat << 'EOF' >> ~/.bashrc
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
export GOPATH=$HOME/go
EOF

source ~/.bashrc
go version
```

## 3. Port Konfigürasyonu

```bash
# Port değişkenleri (default: 26, özel port için değiştirin)
cat << 'EOF' >> ~/.bashrc
# Tellor Port Konfigürasyonu
export CUSTOM_PORT=26
export PORT_P2P="${CUSTOM_PORT}656"
export PORT_RPC="${CUSTOM_PORT}657"
export PORT_API="1${CUSTOM_PORT}17"
export PORT_GRPC="90${CUSTOM_PORT}"
export PORT_GRPC_WEB="91${CUSTOM_PORT}"
EOF

source ~/.bashrc
```

## 4. Binary İndirme ve Kurulum

```bash
# Binary dizini oluştur
mkdir -p ~/layer/binaries/v5.1.1 && cd ~/layer/binaries/v5.1.1

# Binary indir
wget https://github.com/tellor-io/layer/releases/download/v5.1.1/layer_Linux_x86_64.tar.gz
tar -xvzf layer_Linux_x86_64.tar.gz

# Sistem geneline kopyala
sudo cp layerd /usr/local/bin/
sudo chmod +x /usr/local/bin/layerd

# Versiyon kontrolü
layerd version
```

## 5. Node Başlatma

```bash
# Node'u başlat (YOUR_NODE_NAME yerine kendi isminizi yazın)
layerd init "YOUR_NODE_NAME" --chain-id tellor-1

# Genesis dosyasını indir
curl -Ls https://ss.tellor.nodestake.org/genesis.json > $HOME/.layer/config/genesis.json
```

## 6. Ethereum RPC Konfigürasyonu

```bash
# RPC değişkenlerini ayarla (YOUR_API_KEY yerine kendi anahtarınızı yazın)
cat << 'EOF' >> ~/.bashrc
# Ethereum RPC
export ETH_RPC_URL="wss://eth-mainnet.g.alchemy.com/v2/YOUR_API_KEY"
export ETH_RPC_URL_PRIMARY="wss://eth-mainnet.g.alchemy.com/v2/YOUR_API_KEY"
export ETH_RPC_URL_FALLBACK="https://eth-mainnet.g.alchemy.com/v2/YOUR_API_KEY"
export TOKEN_BRIDGE_CONTRACT="0x5589e306b1920F009979a50B88caE32aecD471E4"
EOF

source ~/.bashrc
```

## 7. Konfigürasyon Dosyaları

### app.toml
```bash
# Gas prices ve API ayarları
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0loya\"/" ~/.layer/config/app.toml
sed -i -e "s|tcp://localhost:1317|tcp://0.0.0.0:${PORT_API}|g" ~/.layer/config/app.toml
sed -i -e "s|:9090|:${PORT_GRPC}|g" ~/.layer/config/app.toml
sed -i -e "s|:9091|:${PORT_GRPC_WEB}|g" ~/.layer/config/app.toml

# API etkinleştir
sed -i -e "s/^enable *=.*/enable = true/" ~/.layer/config/app.toml
sed -i -e "s/^swagger *=.*/swagger = true/" ~/.layer/config/app.toml

# Pruning
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" ~/.layer/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" ~/.layer/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" ~/.layer/config/app.toml
```

### config.toml
```bash
# Peer listesi
PEERS="3037a8c239cdcdcf7fbc0ed050a11ecdc0397374@91.99.194.56:26656,17355981bc61dc3c4169158e3d73f22099a5f9c0@152.53.254.219:41767,5ef1ed1fec8700bf9ee16625db2718997ceb499d@157.180.52.245:41656,23a9da592ee6688eac45c82a256ef302a661469b@195.3.223.78:51656,95e55a6cfb850db8c23e969ddd461eac28b98702@3.91.103.4:26656,7fd4d34f3b19c41218027d3b91c90d073ab2ba66@54.221.149.61:26656,2737f23b2223ab1673ce682afdf50d34633f5f7c@69.250.123.126:26656,9358c72aa8be31ce151ef591e6ecf08d25812993@18.143.181.83:26656,2904aa32501548e127d3198c8f5181fb4d67bbe6@18.116.23.104:26656,2b8af463a1f0e84aec6e4dbf3126edf3225df85e@13.52.231.70:26656,f2644778a8a2ca3b55ec65f1b7799d32d4a7098e@54.149.160.93:26656,5a9db46eceb055c9238833aa54e15a2a32a09c9a@54.67.36.145:26656"

sed -i -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.layer/config/config.toml

# Port ayarları
sed -i -e "s|tcp://127.0.0.1:26657|tcp://0.0.0.0:${PORT_RPC}|g" ~/.layer/config/config.toml
sed -i -e "s|tcp://127.0.0.1:26658|tcp://127.0.0.1:${CUSTOM_PORT}658|g" ~/.layer/config/config.toml
sed -i -e "s|:26656|:${PORT_P2P}|g" ~/.layer/config/config.toml
sed -i -e "s|:26658|:${CUSTOM_PORT}658|g" ~/.layer/config/config.toml
sed -i -e "s|proxy_app = \"tcp://127.0.0.1:26658\"|proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}658\"|g" ~/.layer/config/config.toml

# Diğer ayarlar
sed -i -e "s/^cors_allowed_origins *=.*/cors_allowed_origins = [\"*\"]/" ~/.layer/config/config.toml
sed -i -e "s/^timeout_commit *=.*/timeout_commit = \"1s\"/" ~/.layer/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"kv\"/" ~/.layer/config/config.toml
```

### client.toml
```bash
sed -i -e "s/^chain-id *=.*/chain-id = \"tellor-1\"/" ~/.layer/config/client.toml
sed -i -e "s/^keyring-backend *=.*/keyring-backend = \"test\"/" ~/.layer/config/client.toml
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${PORT_RPC}\"|" ~/.layer/config/client.toml
```

## 8. Cüzdan Oluşturma

```bash
# Yeni cüzdan oluştur (seed phrase'i kaydedin!)
layerd keys add wallet --keyring-backend test

# Adresi görüntüle
layerd keys show wallet -a
```

## 9. Cosmovisor Kurulumu

```bash
# Cosmovisor'ı kur
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest

# Environment değişkenleri
cat << 'EOF' >> ~/.bashrc
# Cosmovisor
export DAEMON_NAME=layerd
export DAEMON_HOME=$HOME/.layer
export DAEMON_RESTART_AFTER_UPGRADE=true
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
export DAEMON_POLL_INTERVAL=300ms
export UNSAFE_SKIP_BACKUP=true
export DAEMON_PREUPGRADE_MAX_RETRIES=0
EOF

source ~/.bashrc

# Dizin yapısını oluştur
mkdir -p ~/.layer/cosmovisor/genesis/bin
cp /usr/local/bin/layerd ~/.layer/cosmovisor/genesis/bin/
chmod +x ~/.layer/cosmovisor/genesis/bin/layerd

# Test
cosmovisor version
```

## 10. Snapshot İndirme

```bash
# Veri dizinini temizle
rm -rf ~/.layer/data
layerd tendermint unsafe-reset-all --home ~/.layer/ --keep-addr-book

# Snapshot indir (Nodestake)
echo "Snapshot indiriliyor... (30+ GB, zaman alabilir)"
SNAP_NAME=$(curl -s https://ss.tellor.nodestake.org/ | egrep -o ">20.*\.tar.lz4" | tr -d ">")
curl -o - -L https://ss.tellor.nodestake.org/${SNAP_NAME} | lz4 -c -d - | tar -x -C $HOME/.layer

echo "Snapshot başarıyla yüklendi!"
```

## 11. Systemd Servis Oluşturma

```bash
sudo tee /etc/systemd/system/tellor.service > /dev/null <<EOF
[Unit]
Description=Tellor Layer Node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME
ExecStart=/root/go/bin/cosmovisor run start --home /root/.layer --keyring-backend test --key-name wallet --api.enable --api.swagger
Restart=always
RestartSec=3
LimitNOFILE=65535
Environment="DAEMON_NAME=layerd"
Environment="DAEMON_HOME=/root/.layer"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="ETH_RPC_URL=$ETH_RPC_URL"
Environment="ETH_RPC_URL_PRIMARY=$ETH_RPC_URL_PRIMARY"
Environment="ETH_RPC_URL_FALLBACK=$ETH_RPC_URL_FALLBACK"
Environment="TOKEN_BRIDGE_CONTRACT=0x5589e306b1920F009979a50B88caE32aecD471E4"

[Install]
WantedBy=multi-user.target
EOF

# Servisi başlat
sudo systemctl daemon-reload
sudo systemctl enable tellor
sudo systemctl start tellor
```

## 12. Node Durumu Kontrol

```bash
# Logları izle
sudo journalctl -u tellor -f

# Status kontrolü (30 saniye bekledikten sonra)
curl -s localhost:26657/status | jq '.result.sync_info'

# Peer sayısı
curl -s localhost:26657/net_info | jq -r '.result.n_peers'

# Senkronizasyon durumu
layerd status 2>&1 | jq -r '.SyncInfo.catching_up'
```

## Validator Oluşturma

Node tamamen senkronize olduktan sonra (catching_up: false):

```bash
# Minimum 1 TRB gerekli
layerd tx staking create-validator \
  --amount=1000000loya \
  --pubkey=$(layerd tendermint show-validator) \
  --moniker="YOUR_VALIDATOR_NAME" \
  --chain-id=tellor-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --gas="auto" \
  --gas-adjustment="1.5" \
  --gas-prices="0loya" \
  --from=wallet \
  --keyring-backend test -y
```

## Faydalı Komutlar

```bash
# Servis yönetimi
sudo systemctl stop tellor
sudo systemctl start tellor
sudo systemctl restart tellor
sudo systemctl status tellor

# Cüzdan işlemleri
layerd keys list
layerd keys show wallet -a
layerd query bank balances $(layerd keys show wallet -a)

# Validator bilgileri
layerd query staking validator $(layerd keys show wallet --bech val -a)

# Unjail
layerd tx slashing unjail --from wallet --chain-id tellor-1 --keyring-backend test -y
```

## Notlar

- Snapshot indirme 30+ GB boyutunda, hızınıza göre 15-30 dakika sürebilir
- Senkronizasyon tamamlanması birkaç saat alabilir
- Port değiştirmek için CUSTOM_PORT değişkenini düzenleyin
- Ethereum RPC için Alchemy veya Infura kullanın
- Validator için minimum 1 TRB (1000000 loya) gerekli
