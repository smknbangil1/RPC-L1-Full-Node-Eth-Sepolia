# RPC-L1-Full-Node-Eth-Sepolia
# Membuat RPC Full Node Ethereum L1 (Sepolia Testnet) dengan Resource Minimal

Berikut panduan lengkap untuk membuat full node Ethereum Sepolia Testnet dengan resource minimal, termasuk instalasi, konfigurasi, dan pengujian.

## Persyaratan Hardware Minimal

Untuk menjalankan full node Sepolia Testnet dengan performa dasar:
- **CPU**: 4-core (Intel/AMD 64-bit)
- **RAM**: 8GB (16GB lebih baik)
- **Storage**: SSD 500GB (1TB direkomendasikan untuk ruang pertumbuhan)
- **Bandwidth**: 10 Mbps upload/download stabil
- **OS**: Linux (Ubuntu 22.04 LTS direkomendasikan)

## Langkah 1: Persiapan Sistem

```bash
# Update sistem
sudo apt update && sudo apt upgrade -y

# Install dependensi
sudo apt install -y build-essential git curl wget jq

# Tambahkan swap jika RAM < 16GB (opsional tapi direkomendasikan)
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## Langkah 2: Install Geth (Go Ethereum)

```bash
# Download versi stabil terbaru
wget https://gethstore.blob.core.windows.net/builds/geth-linux-amd64-1.13.5-3475992e.tar.gz

# Ekstrak
tar xvf geth-linux-amd64-1.13.5-3475992e.tar.gz
cd geth-linux-amd64-1.13.5-3475992e/

# Install ke /usr/local/bin
sudo cp geth /usr/local/bin/
```

## Langkah 3: Konfigurasi dan Sync Node

Buat direktori untuk data Ethereum:

```bash
mkdir ~/sepolia-data
```

Buat file layanan systemd (`/etc/systemd/system/geth-sepolia.service`):

```ini
[Unit]
Description=Geth Sepolia Client
After=network.target

[Service]
User=$USER
ExecStart=/usr/local/bin/geth \
  --sepolia \
  --datadir /home/$USER/sepolia-data \
  --http \
  --http.addr 0.0.0.0 \
  --http.port 8545 \
  --http.api eth,net,web3,debug \
  --http.corsdomain "*" \
  --http.vhosts "*" \
  --authrpc.port 8551 \
  --authrpc.addr 0.0.0.0 \
  --authrpc.vhosts "*" \
  --syncmode snap \
  --maxpeers 50
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

Ganti `$USER` dengan username Anda.

## Langkah 4: Jalankan Node

```bash
# Reload systemd
sudo systemctl daemon-reload

# Start service
sudo systemctl start geth-sepolia

# Enable startup otomatis
sudo systemctl enable geth-sepolia

# Cek status
sudo systemctl status geth-sepolia
```

## Langkah 5: Monitor Sync Progress

```bash
# Cek log sync
journalctl -fu geth-sepolia -o cat

# Atau gunakan console Geth
geth attach http://localhost:8545
# Di dalam console Geth:
> eth.syncing
```

Proses sync bisa memakan waktu 12-48 jam tergantung hardware dan koneksi internet.

## Langkah 6: Pengujian Node

### Tes 1: Cek Status Sync

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' http://localhost:8545
```

Jika sudah fully synced, response akan mengembalikan `false`.

### Tes 2: Dapatkan Block Terbaru

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' http://localhost:8545
```

### Tes 3: Query Balance (Contoh alamat Sepolia)

```bash
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x6B2b4D2d1FdD2145aD5D5aC0aB5A8b5a5E5a5E5a","latest"],"id":1}' http://localhost:8545
```

### Tes 4: Kirim Transaksi Test (Butuh ETH Testnet)

Anda bisa dapatkan ETH testnet dari faucet Sepolia: https://sepoliafaucet.com

```bash
# Contoh menggunakan cast (bagian dari foundry)
cast send --rpc-url http://localhost:8545 --private-key $YOUR_PRIVATE_KEY 0x6B2b4D2d1FdD2145aD5D5aC0aB5A8b5a5E5a5E5a "0x"
```

## Optimasi untuk Resource Minimal

1. **Sync Mode**: Gunakan `--syncmode snap` (default) untuk sync lebih cepat
2. **Cache**: Tambahkan `--cache 1024` untuk mengurangi penggunaan RAM
3. **Max Peers**: Kurangi `--maxpeers 25` untuk mengurangi bandwidth
4. **Prune**: Setelah sync selesai, jalankan `geth snapshot prune-state` untuk mengurangi ukuran storage

## Troubleshooting

1. **Sync stuck**: Coba restart service dengan `sudo systemctl restart geth-sepolia`
2. **Disk penuh**: Pastikan memiliki cukup space, atau pertimbangkan pruning
3. **Port tidak terbuka**: Pastikan firewall membolehkan port 8545 (HTTP) dan 30303 (P2P)

## Kesimpulan

Dengan konfigurasi di atas, Anda telah membuat full node Ethereum Sepolia Testnet yang berfungsi sebagai RPC endpoint. Node ini dapat digunakan untuk:
- Mengembangkan dan menguji smart contract
- Membuat transaksi tanpa bergantung pada provider pihak ketiga
- Membaca data blockchain secara langsung
- Panduan RPC L2 Fullnode mandiri, menyusul, tanpa sewa pihak ketiga (alchemy dsb...) lebih hemat biaya untuk keperluan testnet maupun mainnet

Untuk mainnet Ethereum, persyaratan hardware akan lebih tinggi (minimal 2TB SSD, CPU 8 core dan 16GB RAM).
