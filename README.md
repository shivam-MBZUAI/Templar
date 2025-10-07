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

## Clone the repository
```bash
# Clone the project
git clone https://github.com/one-covenant/templar.git
cd templar
```

## What Each Component Does
- Blockchain (Subtensor): A local blockchain that tracks who does what work
  - Keeps track of who's participating
  - Records who do good work, then gives out rewards
  - The blockchain ensures fairness; nobody can lie about their score.
  - The blockchain is the foundation, and subnets are the specialized spaces inside the blockchain where actual work happens.

- Wallets: Digital identities for each participant (like usernames)
  - Each participant needs to be recognized
  - Contains their identity and their "bank account" for rewards
  - We create fake ones for testing (like practice IDs)
- Subnet: A specific "channel" where your AI models operate
  - Different subnets can focus on different types of learning
- Validator: The "teacher" that grades the miners' work
  - Checks if the miners (students) are actually learning
  - Has the power to give good or bad scores
- Miners: The "students" doing AI training
  - They're AI models trying to learn and improve
  - They compete to give the best answers
  - Get rewards (grades/money) when they do well
