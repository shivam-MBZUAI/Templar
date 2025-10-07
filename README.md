# Templar Test Environment Setup

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Bittensor](https://img.shields.io/badge/Bittensor-000000?style=for-the-badge&logo=bitcoin&logoColor=white)](https://bittensor.com/)

## The Big Picture
You're setting up a test environment for an AI training system called Templar. Think of it as a mini version of what runs in production. Below are the components involved:
- Blockchain (Subtensor): Manages rewards and coordination between AI models
- Validator: Checks the quality of AI models' work
- Miners: AI models that do the actual training/learning
- You're testing: Whether they all work together correctly
- **BLOCKCHAIN (Subtensor)**
  - **Subnet 1** (Image Recognition)
    - Validators (5 of them)
    - Miners (100 of them)
  - **Subnet 2** (Text Generation) ← Your test
    - Validators (3 of them)
    - Miners (50 of them)
  - **Subnet 3** (Translation)
    - Validators (10 of them)
    - Miners (200 of them)

## Clone the repository
```bash
# Clone the project
git clone https://github.com/one-covenant/templar.git
cd templar
```
## Install Dependencies
```bash
# Install UV (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh
source "$HOME/.local/bin/env"

# Install project dependencies
uv sync --all-extras
# Install Just (task runner)
cargo install just
# Install PM2 (process manager) 
npm install -g pm2
# Install dotenv
npm install dotenv
```

## What Each Component Does
- Blockchain (Subtensor): A local blockchain that tracks who does what work
  - Keeps track of who's participating
  - Records who do good work, then gives out rewards
  - The blockchain ensures fairness; nobody can lie about their score.
  - The blockchain is the foundation, and subnets are the specialized spaces inside the blockchain where actual work happens.
### Start the Blockchain (Subtensor)
```bash
# Use tmux to keep it running in the background
tmux new -s subtensor

# Pull and run the blockchain
docker pull ghcr.io/opentensor/subtensor-localnet:v2.0.11
docker run --rm --name subtensor-local \
           -p 9944:9944 -p 9945:9945 \
           ghcr.io/opentensor/subtensor-localnet:v2.0.11

# Press Ctrl+B then D to detach (keep it running in background)
```
- Wallets: Digital identities for each participant (like usernames)
  - Each participant needs to be recognized
  - Contains their identity and their "bank account" for rewards
  - We create fake ones for testing (like practice IDs)
### Create Fake Wallets (participants)
```bash
for name in owner validator miner{1..3}; do
  uvx --from bittensor-cli btcli wallet new_coldkey \
      --wallet.name "$name" -p ~/.bittensor/wallets \
      --n-words 24 --no-use-password
  uvx --from bittensor-cli btcli wallet new_hotkey \
      --wallet.name "$name" --wallet.hotkey default --n-words 24
done
```
### Give Wallets Test Money(tokens)
```bash
for name in owner validator miner{1..3}; do
  uvx --from bittensor-cli --with torch btcli wallet faucet \
      --wallet.name "$name" --network local -y
done
```

- Subnet: A specific "channel" where your AI models operate
  - Different subnets can focus on different types of learning
- Validator: The "teacher" that grades the miners' work
  - Checks if the miners (students) are actually learning
  - Has the power to give good or bad scores
- Miners: The "students" doing AI training
  - They're AI models trying to learn and improve
  - They compete to give the best answers
  - Get rewards (grades/money) when they do well
