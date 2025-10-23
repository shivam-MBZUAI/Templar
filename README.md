# Templar Test Environment Setup Guide

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Bittensor](https://img.shields.io/badge/Bittensor-000000?style=for-the-badge&logo=bitcoin&logoColor=white)](https://bittensor.com/)

---

## The Big Picture

You're setting up a **local test environment** for Templar, a decentralized AI training system. Here's what each component does:

- **Blockchain (Subtensor)**: Coordinates rewards and synchronization between participants
- **Validators**: Verify the quality of AI model updates
- **Miners**: Perform actual AI model training and share gradients
- **Your Goal**: Test that all components work together correctly

```
BLOCKCHAIN (Subtensor Basic Setup) - Local Chain
    │
    └── Subnet 2 (Your Test Subnet)
        ├── Validator (1)
        └── Miners (2)
```

---

## Step 1: Clone Repository

```bash
# Clone the templar project
git clone https://github.com/one-covenant/templar.git
cd templar
```

---

## Step 2: Install Dependencies

### 2.1 Install UV (Python Package Manager)
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source "$HOME/.local/bin/env"
```

### 2.2 Install Project Dependencies
```bash
# This installs all Python packages needed
uv sync --all-extras
```

### 2.3 Install Rust & Just (Task Runner)
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
cargo install just
just --version  # Verify installation
```

### 2.4 Install Node.js & PM2 (Process Manager)
```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# Load nvm and install Node.js LTS
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
nvm install --lts

# Install PM2 globally
npm install -g pm2
pm2 --version  # Verify installation
```

### 2.5 Install dotenv
```bash
npm install dotenv
```

** Checkpoint:** All tools should now be installed. You can verify by running:
```bash
uv --version
just --version
pm2 --version
node --version
```

---

## Step 3: Hugging Face Setup

Templar uses the Gemma tokenizer from Hugging Face (a gated/restricted model).

### 3.1 Create Account
1. Go to https://huggingface.co/join
2. Create a free account

### 3.2 Accept Gemma Model Terms
1. Go to https://huggingface.co/google/gemma-3-270m
2. Click **"Agree and access repository"**
3. Wait for approval (usually instant)

### 3.3 Create Access Token
1. Go to https://huggingface.co/settings/tokens
2. Click **"Create new token"**
3. Name it: `templar-read`
4. Type: **Read** access
5. Click **"Generate token"**
6. **Copy the token** (looks like: `hf_xxxxxxxxxxxxx...`)
7. **Save it somewhere safe** - you'll need it for the `.env` file

---

## Step 4: Cloudflare R2 Setup

Templar uses Cloudflare R2 (cloud storage) to share AI model updates between miners and validators.

### 4.1 Create Cloudflare Account
1. Go to https://dash.cloudflare.com/sign-up
2. Create a free account
3. **Add a payment method** (required for R2, but you get 10GB free/month)

### 4.2 Enable R2 Storage
1. In Cloudflare dashboard, click **"R2"** in the left sidebar
2. Click **"Overview"**
3. Click **"Get started with R2"**
4. Fill in payment details and click **"Add R2 subscription to my account"**

### 4.3 Note Your Account ID
After enabling R2, you'll see your **Account ID** (a 32-character hex string like `dc1ac72b395ec87f6caa8ef3df3d56a4`).

** Write it down** - you'll need it multiple times.

### 4.4 Create R2 Bucket
1. Click **"Create bucket"**
2. **Bucket name:** Use your Account ID (same string from above)
3. **Location:** Choose a region close to you (e.g., ENAM for North America East)
4. Click **"Create bucket"**

### 4.5 Create API Tokens

You need **3 API tokens** with different permissions.

#### Token 1: Gradients Read
1. Click **"Manage R2 API Tokens"** (or similar)
2. Click **"Create User API token"**
3. **Token name:** `gradients-read`
4. **Permissions:** Select **"Object Read only"**
5. **Bucket scope:** **"Apply to specific buckets only"** → Select your bucket
6. **TTL:** Forever
7. Click **"Create User API token"**
8. ** COPY BOTH:**
   - Access Key ID
   - Secret Access Key
   - **You can't see the secret again!**

#### Token 2: Gradients Write
1. Click **"Create User API token"** again
2. **Token name:** `gradients-write`
3. **Permissions:** Select **"Object Read & Write"**
4. **Bucket scope:** **"Apply to specific buckets only"** → Select your bucket
5. **TTL:** Forever
6. Click **"Create User API token"**
7. ** COPY BOTH:** Access Key ID and Secret Access Key

#### Token 3: Aggregator Read
1. Click **"Create User API token"** again
2. **Token name:** `aggregator-read`
3. **Permissions:** Select **"Object Read only"**
4. **Bucket scope:** **"Apply to specific buckets only"** → Select your bucket
5. **TTL:** Forever
6. Click **"Create User API token"**
7. ** COPY BOTH:** Access Key ID and Secret Access Key

** Checkpoint:** You should now have:
- Your Account ID
- 3 pairs of API credentials (6 keys total)

---

## Step 5: Create .env File

Create a `.env` file in the `templar` directory with all your credentials.

```bash
cd ~/templar
nano .env  # or use your preferred text editor
```

### Copy this template and replace the values:

```env
# ============================================
# Cloudflare R2 Storage Configuration
# ============================================

# Your Cloudflare Account ID (same for all)
R2_GRADIENTS_ACCOUNT_ID=YOUR_ACCOUNT_ID_HERE
R2_GRADIENTS_BUCKET_NAME=YOUR_ACCOUNT_ID_HERE

# Gradients - Read Access (from Token 1)
R2_GRADIENTS_READ_ACCESS_KEY_ID=YOUR_GRADIENTS_READ_KEY_ID
R2_GRADIENTS_READ_SECRET_ACCESS_KEY=YOUR_GRADIENTS_READ_SECRET

# Gradients - Write Access (from Token 2)
R2_GRADIENTS_WRITE_ACCESS_KEY_ID=YOUR_GRADIENTS_WRITE_KEY_ID
R2_GRADIENTS_WRITE_SECRET_ACCESS_KEY=YOUR_GRADIENTS_WRITE_SECRET

# Aggregator Storage (same account ID)
R2_AGGREGATOR_ACCOUNT_ID=YOUR_ACCOUNT_ID_HERE
R2_AGGREGATOR_BUCKET_NAME=YOUR_ACCOUNT_ID_HERE

# Aggregator - Read Access (from Token 3)
R2_AGGREGATOR_READ_ACCESS_KEY_ID=YOUR_AGGREGATOR_READ_KEY_ID
R2_AGGREGATOR_READ_SECRET_ACCESS_KEY=YOUR_AGGREGATOR_READ_SECRET

# ============================================
# Dataset Configuration (Shared Credentials)
# ============================================
# These are provided by the Templar team - use as-is
R2_DATASET_ACCOUNT_ID=8af7f92a8a0661cf7f1ac0420c932980
R2_DATASET_BUCKET_NAME=gemma-migration
R2_DATASET_READ_ACCESS_KEY_ID=a733fac6c32a549e0d48f9f7cf67d758
R2_DATASET_READ_SECRET_ACCESS_KEY=f50cab456587f015ad21c48c3e23c7ff0e6f1ad5a22c814c3a50d1a4b7c76bb9
DATASET_BINS_PATH=tokenized/

# ============================================
# Hugging Face Token
# ============================================
HF_TOKEN=hf_YOUR_HUGGINGFACE_TOKEN_HERE
```

### Example (with fake credentials):
```env
R2_GRADIENTS_ACCOUNT_ID=dc1ac72b395ec87f6caa8ef3df3d56a4
R2_GRADIENTS_BUCKET_NAME=dc1ac72b395ec87f6caa8ef3df3d56a4
R2_GRADIENTS_READ_ACCESS_KEY_ID=a1cc9c4c3ab594df05acd963af331abc
R2_GRADIENTS_READ_SECRET_ACCESS_KEY=5a0ef2b0fb438fbf789d78f877208cfe53876cf6ce1b4cb2ff8135f51dbe76b8
R2_GRADIENTS_WRITE_ACCESS_KEY_ID=482dcc88d3caa6ef9b771f61b3694d69
R2_GRADIENTS_WRITE_SECRET_ACCESS_KEY=0d9594a6a9e051455d8677602ecc1f489fc3761a85eb44c4a61a026fd8c13528
R2_AGGREGATOR_ACCOUNT_ID=dc1ac72b395ec87f6caa8ef3df3d56a4
R2_AGGREGATOR_BUCKET_NAME=dc1ac72b395ec87f6caa8ef3df3d56a4
R2_AGGREGATOR_READ_ACCESS_KEY_ID=ef9fd25b54ea1b9b16a697e2a5ba3a13
R2_AGGREGATOR_READ_SECRET_ACCESS_KEY=4762b5e65a00115d5545776f641cfe56065befc3ac62691a68c8d1cc271fb567
R2_DATASET_ACCOUNT_ID=8af7f92a8a0661cf7f1ac0420c932980
R2_DATASET_BUCKET_NAME=gemma-migration
R2_DATASET_READ_ACCESS_KEY_ID=a733fac6c32a549e0d48f9f7cf67d758
R2_DATASET_READ_SECRET_ACCESS_KEY=f50cab456587f015ad21c48c3e23c7ff0e6f1ad5a22c814c3a50d1a4b7c76bb9
DATASET_BINS_PATH=tokenized/
HF_TOKEN=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
```

** Save the file** (Ctrl+X, then Y, then Enter if using nano)

---

## Step 6: (Optional) Pre-download Dataset

The training dataset lives in the cloud and downloads automatically during training. However, you can **pre-download** the first two shards (~860GB) to save time later.

### What is a "shard"?
The dataset is split into multiple ~430GB chunks called "shards". The system automatically:
- Downloads the first shard when you start training
- Downloads the next shard in the background
- Deletes old shards to save disk space

### To pre-download shards 0 and 1:
```bash
cd ~/templar
source .venv/bin/activate
python scripts/download_dataset.py
```

---

## Step 7: Start Local Subtensor Blockchain

The Subtensor blockchain coordinates the network. For local testing, we run it in Docker.

### 7.1 Verify Docker is Installed
```bash
docker --version
# Should show: Docker version 28.x.x or higher
```

### 7.2 Pull the Subtensor Image
```bash
cd ~/templar
docker pull ghcr.io/opentensor/subtensor-localnet:v2.0.11
```

### 7.3 Start the Local Blockchain

** Important:** Use **12-second blocks** (not the default 250ms) to avoid faucet issues.

```bash
docker run -d --name subtensor-localnet \
  -p 9944:9944 -p 9945:9945 \
  ghcr.io/opentensor/subtensor-localnet:v2.0.11 False
```

**Why `False`?** 
- Default (no parameter) = 250ms blocks → Too fast for faucet PoW 
- `False` parameter = 12-second blocks → Faucet works reliably 

### 7.4 Verify It's Running
```bash
docker ps --filter name=subtensor-localnet
# You should see the container with status "Up"

# Check blockchain is producing blocks
docker logs subtensor-localnet --tail 10
```

---

## Step 8: Create Wallets

Templar uses one **coldkey** (like a bank account) with multiple **hotkeys** (like debit cards) for different miners/validators.

### 8.1 Create Coldkey
```bash
cd ~/templar
.venv/bin/btcli wallet new_coldkey --wallet.name templar_test -p ~/.bittensor/wallets --n-words 12 --no-use-password
```

** Important:** Save the mnemonic phrase shown! It's needed to recover your wallet.

### 8.2 Create Hotkeys

Create 3 hotkeys (M1, M2 for miners, V1 for validator):

```bash
# Miner 1 hotkey
.venv/bin/btcli wallet new_hotkey --wallet.name templar_test --wallet.hotkey M1 --n-words 12 --no-use-password

# Miner 2 hotkey
.venv/bin/btcli wallet new_hotkey --wallet.name templar_test --wallet.hotkey M2 --n-words 12 --no-use-password

# Validator hotkey
.venv/bin/btcli wallet new_hotkey --wallet.name templar_test --wallet.hotkey V1 --n-words 12 --no-use-password
```

** Checkpoint:** You should now have:
- 1 coldkey: `templar_test`
- 3 hotkeys: `M1`, `M2`, `V1`

---

## Step 9: Fund Wallets from Local Faucet

Each wallet needs TAO tokens to interact with the blockchain. The local faucet provides free tokens for testing.

** If you used the `False` parameter in Step 7.3, the faucet will work smoothly!**

### 9.1 Fund Each Hotkey

Each command takes **30-90 seconds** to complete proof-of-work:

```bash
# Fund M1
.venv/bin/btcli wallet faucet --wallet.name templar_test --wallet.hotkey M1 --max-successes 1 --network local -y

# Fund M2
.venv/bin/btcli wallet faucet --wallet.name templar_test --wallet.hotkey M2 --max-successes 1 --network local -y

# Fund V1
.venv/bin/btcli wallet faucet --wallet.name templar_test --wallet.hotkey V1 --max-successes 1 --network local -y
```

**Expected output for each:**
```
Balance: ‎0.0000 τ‎ ➡ ‎1,000.0000 τ‎
```

### 9.2 Verify All Balances
```bash
.venv/bin/pip install bittensor-cli==9.7.1
.venv/bin/btcli wallet balance --wallet.name templar_test --subtensor.chain_endpoint ws://127.0.0.1:9945
```
---

## Step 10: Create Subnet and Register Hotkeys

Now we create a test subnet and register all participants.

### 10.1 Create Subnet 2
```bash
.venv/bin/btcli subnet create --wallet.name templar_test --wallet.hotkey V1 --subtensor.chain_endpoint ws://127.0.0.1:9945 -y
```
Subnet name (optional) (): 
GitHub repository URL (optional) (): 
Contact email (optional) (): 
Subnet URL (optional) (): 
Discord handle (optional) (): 
Description (optional) (): 
Additional information (optional) (): 
⠋ Connecting to Substrate: Network: custom, Chain: ws://127.0.0.1:9945...
⠸ Connecting to Substrate: Network: custom, Chain: ws://127.0.0.1:9945...
⠴ Connecting to Substrate: Network: custom, Chain: ws://127.0.0.1:9945...
Subnet burn cost: ‎1,000.0000 τ‎
✅ Registered subnetwork with netuid: 2


### 10.2 Register All Hotkeys to Subnet 2
```bash
# Register Validator
.venv/bin/btcli subnet register --wallet.name templar_test --wallet.hotkey V1 --netuid 2 --subtensor.chain_endpoint ws://127.0.0.1:9945 -y

# Register Miner 1
.venv/bin/btcli subnet register --wallet.name templar_test --wallet.hotkey M1 --netuid 2 --subtensor.chain_endpoint ws://127.0.0.1:9945 -y

Balance:
  ‎1,000.0893 τ‎ ➡ ‎999.0893 τ‎
✅ Registered on netuid 2 with UID 1

# Register Miner 2
.venv/bin/btcli subnet register --wallet.name templar_test --wallet.hotkey M2 --netuid 2 --subtensor.chain_endpoint ws://127.0.0.1:9945 -y

Balance:
  ‎1,000.0893 τ‎ ➡ ‎999.0893 τ‎
✅ Registered on netuid 2 with UID 2
```

To register on a specific netuid
```bash
.venv/bin/btcli subnet register --wallet.name templar_test --wallet.hotkey V1 --netuid 2 --subtensor.chain_endpoint ws://127.0.0.1:9945 -y
```

### 10.3 Stake TAO to Validator on netuid 2
```bash
.venv/bin/btcli stake add --wallet.name templar_test --wallet.hotkey V1 --netuid 2 --subtensor.chain_endpoint ws://127.0.0.1:9945 --amount 100 --allow-partial-stake -y
```


### 10.4 Verify Registration
```bash
.venv/bin/btcli subnet list --subtensor.chain_endpoint ws://127.0.0.1:9945
```

### 10.5 Check the metagraph for subnet 2 to see all registered neurons
```bash
.venv/bin/btcli subnet metagraph --netuid 2 --subtensor.chain_endpoint ws://127.0.0.1:9945
```
---

## Step 11: Run Your Tests!

You've now completed the **one-time blockchain and wallet setup**! 

### 11.1 Quick Test Run
```bash
cd ~/templar
just test-run
```

This starts all processes (2 miners + 1 validator) using PM2.

### 11.2 Monitor Processes
```bash
# View all running processes
pm2 status

# View logs from all processes
pm2 logs --lines 50

# View specific process logs
pm2 logs TM1    # Miner 1
pm2 logs TM2    # Miner 2
pm2 logs TV1    # Validator
```

### 11.3 Stop/Restart for Testing Changes
```bash
# Stop all processes
pm2 stop all

# Make your code changes...

# Restart with new changes
pm2 start ecosystem.config.js

# Or just restart everything
just test-run
```

## Important Notes

###  **One-Time Setup (Done!):**
- Docker blockchain (keeps running)
- Wallets (persistent)
- Subnet registration (permanent on local chain)

###  **For Each Test Run:**
- Edit config files (`hparams.json`, `ecosystem.config.js`)
- Run `pm2 stop all` then `just test-run`
- Check WandB for metrics

###  **Key Metrics to Track:**
- `sync_score` (should be ~1.0)
- `tokens_per_sec` (throughput)
- `loss` (should decrease)
- Process stability (no crashes)

---

## Troubleshooting

### If blockchain stops:
```bash
docker restart subtensor-localnet
```

### If you need to reset everything:
```bash
# Stop and remove blockchain
docker stop subtensor-localnet
docker rm subtensor-localnet

# Remove wallets
rm -rf ~/.bittensor/wallets/templar_test

# Start from Step 7 again
```
