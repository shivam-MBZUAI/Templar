# 🚀 Templar Test Environment Setup

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Bittensor](https://img.shields.io/badge/Bittensor-000000?style=for-the-badge&logo=bitcoin&logoColor=white)](https://bittensor.com/)

A comprehensive guide for setting up a local Bittensor test environment with validators and miners for integration testing.

## 📋 Table of Contents
- [Overview](#overview)
- [System Requirements](#system-requirements)
- [Quick Start](#quick-start)
- [Detailed Setup](#detailed-setup)
  - [Linux/Ubuntu Setup](#linuxubuntu-setup)
  - [macOS Setup](#macos-setup)
- [Configuration](#configuration)
- [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

## 🎯 Overview

This setup creates a local test environment with:

| Component | Count | Purpose |
|-----------|-------|---------|
| **Local Subtensor Node** | 1 | Private blockchain at `ws://127.0.0.1:9944` |
| **Subnet** | 1 | Sandboxed economics on `netuid 2` |
| **Validator** | 1 | Reference validator logic |
| **Miners** | 1+ | Traffic generation for testing |

## 💻 System Requirements

### Minimum Requirements
- **RAM**: 16GB minimum (32GB recommended)
- **Storage**: 50GB available space
- **OS**: Ubuntu 20.04+ or macOS 12+
- **GPU**: Optional (CUDA-compatible for Linux, Apple Silicon for Mac)

### Software Prerequisites
- Docker or Docker Desktop
- Git
- Node.js 16+
- Python 3.9+

## ⚡ Quick Start

```bash
# Clone the repository
git clone https://github.com/one-covenant/templar.git
cd templar

# Run the automated setup (Linux/Ubuntu)
./scripts/quick-setup.sh

# Or use Docker Compose
docker-compose up -d
```

## 📖 Detailed Setup

### Linux/Ubuntu Setup

<details>
<summary>Click to expand Linux instructions</summary>

#### 1. Install System Dependencies

```bash
sudo apt update
sudo apt install --assume-yes \
  make build-essential git clang curl llvm \
  libssl-dev libudev-dev protobuf-compiler pkg-config \
  tmux nodejs npm neovim
```

#### 2. Start Local Subtensor Node

Choose one of the following methods:

**Option A: Docker (Recommended)**
```bash
docker pull ghcr.io/opentensor/subtensor-localnet:v2.0.11
docker run --rm --name subtensor-local \
           -p 9944:9944 -p 9945:9945 \
           ghcr.io/opentensor/subtensor-localnet:v2.0.11
```

**Option B: Docker Compose**
```bash
git clone https://github.com/opentensor/subtensor.git
cd subtensor && git checkout v2.0.11
docker compose up --build --detach
```

**Option C: Build from Source**
```bash
git clone https://github.com/opentensor/subtensor.git
cd subtensor && git checkout v2.0.11
./scripts/init.sh
./scripts/localnet.sh False --build-only
tmux new -d -s localnet \
     'cd subtensor && ./scripts/localnet.sh False --no-purge'
```

#### 3. Verify Connection

```bash
btcli subnet list --network ws://127.0.0.1:9944
```

</details>

### macOS Setup

<details>
<summary>Click to expand macOS instructions</summary>

#### 1. Install Homebrew & Dependencies

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install required packages
brew install git curl llvm protobuf pkg-config tmux node neovim
brew install --cask docker
```

#### 2. Install Docker Desktop

1. Download [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop/)
2. Choose **Apple Silicon** version for M1/M2 Macs
3. Start Docker Desktop from Applications

#### 3. Start Local Subtensor

```bash
docker pull ghcr.io/opentensor/subtensor-localnet:v2.0.11
docker run --rm --name subtensor-local \
           -p 9944:9944 -p 9945:9945 \
           ghcr.io/opentensor/subtensor-localnet:v2.0.11
```

> **Note for Apple Silicon Users**: CUDA is not available on Mac. The system will use CPU or Metal Performance Shaders (MPS) instead.

</details>

### 🔧 Common Setup Steps (All Platforms)

#### 4. Clone and Install Project Dependencies

```bash
# Clone the repository
git clone https://github.com/one-covenant/templar.git
cd templar

# Install UV (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh
source "$HOME/.local/bin/env"

# Sync Python dependencies
uv sync --all-extras

# Install additional tools
cargo install just  # Requires Rust - install from https://rustup.rs
sudo npm install -g pm2
npm install dotenv
```

#### 5. Create Wallets

```bash
# Create coldkeys and hotkeys for all participants
for name in owner validator miner{1..3}; do
  uvx --from bittensor-cli btcli wallet new_coldkey \
      --wallet.name "$name" -p ~/.bittensor/wallets \
      --n-words 24 --no-use-password
  uvx --from bittensor-cli btcli wallet new_hotkey \
      --wallet.name "$name" --wallet.hotkey default --n-words 24
done
```

#### 6. Fund Wallets

```bash
# Add test tokens from local faucet
for name in owner validator miner{1..3}; do
  uvx --from bittensor-cli --with torch btcli wallet faucet \
      --wallet.name "$name" --max-successes 1 --network local -y
done
```

#### 7. Create and Setup Subnet

```bash
# Create subnet
uvx --from bittensor-cli btcli subnet create \
    --wallet.name owner --wallet.hotkey default --network local -y

# Register validator
uvx --from bittensor-cli btcli subnet register \
    --wallet.name validator --wallet.hotkey default --netuid 2 --network local -y

# Register miners (wait 360 blocks if registering >3 per epoch)
for i in {1..3}; do
  uvx --from bittensor-cli btcli subnet register \
      --wallet.name "miner$i" --wallet.hotkey default \
      --netuid 2 --network local -y
done

# Stake to validator
uvx --from bittensor-cli btcli stake add \
    --wallet.name validator --wallet.hotkey default \
    --netuid 2 --network local --unsafe -y
```

## ⚙️ Configuration

### Environment Variables

Create a `.env` file in the project root:

```env
# R2 Storage Configuration
R2_GRADIENTS_ACCOUNT_ID=your_account_id_here
R2_GRADIENTS_BUCKET_NAME=$R2_GRADIENTS_ACCOUNT_ID
R2_GRADIENTS_READ_ACCESS_KEY_ID=your_read_key_here
R2_GRADIENTS_READ_SECRET_ACCESS_KEY=your_read_secret_here
R2_GRADIENTS_WRITE_ACCESS_KEY_ID=your_write_key_here
R2_GRADIENTS_WRITE_SECRET_ACCESS_KEY=your_write_secret_here

R2_AGGREGATOR_ACCOUNT_ID=$R2_GRADIENTS_ACCOUNT_ID
R2_AGGREGATOR_BUCKET_NAME=$R2_AGGREGATOR_ACCOUNT_ID
R2_AGGREGATOR_READ_ACCESS_KEY_ID=your_aggregator_key_here
R2_AGGREGATOR_READ_SECRET_ACCESS_KEY=your_aggregator_secret_here

# Dataset Configuration
DATASET_BINS_PATH=/absolute/path/to/your/dataset_bins
```

> ⚠️ **Important**: Contact your team for actual R2 credentials and dataset paths.

### Process Configuration

Edit `ecosystem.config.js` to configure:
- Wallet names (coldkey/hotkey)
- Number of processes (`nproc_per_node`)
- GPU settings (`CUDA_VISIBLE_DEVICES`)

Edit `hparams/hparams.json` to adjust:
- `blocks_per_window`: Lower for faster testing loops
- `model_size`: Choose between `"150M"`, `"1B"`, or `"8B"`
- `minimum_peers`, `target_batch_size`, etc.

## 🚀 Running the Test

```bash
# Start all processes
just test-run

# Monitor logs
pm2 logs --lines 100

# Check process status
pm2 status

# Stop all processes
pm2 stop all
```

## 📊 Monitoring

### Health Indicators

| Metric | Healthy Signal | Warning Sign |
|--------|---------------|--------------|
| **Process Status** | PM2 shows `online` ≥3 windows | Recurring NCCL/OOM errors |
| **Validator Sync** | `sync_score` ≈ 1.0, `avg_steps_behind` < 1 | Sudden jumps indicate checkpoint issues |
| **Gather Success** | `miner/gather/success_rate` > 90% | Drop indicates tensor shape mismatches |
| **Binary Indicator** | Trends toward +1 | Flips to -1 indicate data problems |
| **Loss Curves** | Validator random loss decreasing | Spikes indicate exploding gradients |

### Useful Commands

```bash
# Check subnet status
btcli subnet list --network local

# View wallet balance
btcli wallet balance --wallet.name validator --network local

# Monitor specific miner
pm2 logs miner1

# Restart specific process
pm2 restart validator
```

## 🔨 Troubleshooting

### Common Issues

<details>
<summary>Docker permission denied</summary>

```bash
# Add user to docker group
sudo usermod -aG docker $USER
# Log out and back in for changes to take effect
```
</details>

<details>
<summary>Port 9944 already in use</summary>

```bash
# Find and kill the process using the port
sudo lsof -i :9944
sudo kill -9 <PID>
```
</details>

<details>
<summary>CUDA not found (Linux)</summary>

```bash
# Install CUDA drivers
sudo apt install nvidia-cuda-toolkit
# Verify installation
nvidia-smi
```
</details>

<details>
<summary>Process crashes with OOM</summary>

- Reduce `model_size` in `hparams/hparams.json`
- Lower `target_batch_size`
- Reduce number of miners
</details>

<details>
<summary>Apple Silicon Compatibility</summary>

For M1/M2 Macs:
- CUDA operations will fallback to CPU/MPS
- Install Rust manually: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`
- Use Docker Desktop instead of Docker CLI
- Consider using Rosetta 2 for x86 compatibility if needed
</details>

### Scaling Tips

- **Add more miners**: Duplicate entries in `ecosystem.config.js` with pre-registered wallets
- **Faster feedback**: Lower `blocks_per_window` and use smaller model sizes
- **Resource optimization**: 
  - Use `"150M"` model size for quick testing
  - Reduce `target_batch_size` for lower memory usage
  - Limit concurrent miners based on available RAM

## ❓ FAQ

**Q: Can I run this without a GPU?**  
A: Yes, but performance will be limited. Set appropriate CPU-only configurations in `hparams.json`.

**Q: How much disk space do I need?**  
A: Minimum 50GB for blockchain data, model checkpoints, and datasets. Recommended 100GB for comfortable operation.

**Q: Can I run this on Apple Silicon (M1/M2)?**  
A: Yes, but CUDA features won't be available. The system will use CPU or Metal Performance Shaders (MPS).

**Q: How do I reset everything and start fresh?**  
A: 
```bash
pm2 delete all
rm -rf ~/.bittensor/wallets
docker stop subtensor-local
docker rm subtensor-local
# Then restart from step 5
```

**Q: What if I get rate limited by the faucet?**  
A: Wait 360 blocks (approximately 90 seconds with 250ms blocks) before requesting more funds.

**Q: Can I change the subnet ID from 2?**  
A: Yes, but you'll need to update all registration commands and configuration files accordingly.

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Development Guidelines

- Follow PEP 8 for Python code
- Use meaningful commit messages
- Add tests for new features
- Update documentation as needed

## 📄 License

This project is part of the Bittensor ecosystem. See [LICENSE](LICENSE) for details.

## 🆘 Support

- **Documentation**: [Bittensor Docs](https://docs.bittensor.com)
- **Discord**: [Join our Discord](https://discord.gg/bittensor)
- **GitHub Issues**: [Report bugs](https://github.com/one-covenant/templar/issues)
- **Team Contact**: Contact your team lead for R2 credentials and internal resources

## 📚 Additional Resources

- [Bittensor Official Documentation](https://docs.bittensor.com)
- [Subtensor Repository](https://github.com/opentensor/subtensor)
- [PM2 Documentation](https://pm2.keymetrics.io/)
- [Docker Documentation](https://docs.docker.com)

---

<p align="center">
  <strong>Built with ❤️ by the One Covenant Team</strong>
  <br>
  <sub>Last updated: October 2025</sub>
</p>
