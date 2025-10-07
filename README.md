# Templar Test Environment Setup

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Bittensor](https://img.shields.io/badge/Bittensor-000000?style=for-the-badge&logo=bitcoin&logoColor=white)](https://bittensor.com/)

## The Big Picture
You're setting up a test environment for an AI training system called Templar. Think of it as a mini version of what runs in production. Here's what the system does:
- Blockchain (Subtensor): Manages rewards and coordination between AI models
-- Think of blockchain like a scoreboard that:
--- Keeps track of who's participating
--- Records who do good work
--- Gives out rewards

The blockchain ensures fairness - nobody can lie about their score.
- Validator: Checks the quality of AI models' work
- Miners: AI models that do the actual training/learning
- You're testing: Whether they all work together correctly

## What Each Component Does
- Subtensor Node: A local blockchain that tracks who does what work
- Wallets: Digital identities for each participant (like usernames)
- Subnet: A specific "channel" where your AI models operate
- Validator: The "teacher" that grades the miners' work
- Miners: The "students" doing AI training
