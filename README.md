# MongoDB vs Relational: E-commerce Schema Design Exercise

## Overview

This repository contains an exercise to explore the differences between relational (PostgreSQL) and MongoDB paradigms through an e-commerce application schema design. This exercise explores the different approaches to schema design between relational and document-based databases, focusing on how MongoDB's document model can provide flexibility advantages particularly useful when dealing with evolving application requirements.

E-commerce applications frequently evolve with changing business requirements - new product types, promotion strategies, customer attributes, and order processing workflows. In traditional relational databases like PostgreSQL, these changes often require schema modifications, migration scripts, and application code updates. MongoDB's flexible document model offers a different approach that can significantly reduce development friction during these evolutionary phases.

## Learning Objectives

By completing this exercise, you will:
1. Gain practical experience setting up both MongoDB and PostgreSQL locally
2. Learn how to translate relational schemas to MongoDB schemas
3. Understand MongoDB schema design patterns and when to apply them
4. Experience firsthand how MongoDB handles schema evolution compared to relational (PostgreSQL)

## Repository Contents

- `mongo/` 
  - `docker-compose.yml` - Configuration to set up MongoDB container
- `psql/`
  - `docker-compose.yml` - Configuration to set up PostgreSQL container
- `LICENSE` - License information
- `README.md` - This file
  
## Project set-up

### Installing Docker

#### macOS
1. Download Docker Desktop for Mac from [Docker Hub](https://docs.docker.com/desktop/install/mac-install/)
2. Double-click the downloaded `.dmg` file and drag the Docker app to your Applications folder
3. Open Docker from your Applications folder
4. When prompted, authorize Docker with your system password
5. Once Docker Desktop is running, you should see the Docker icon in the status bar

#### Windows
1. Download Docker Desktop for Windows from [Docker Hub](https://docs.docker.com/desktop/install/windows-install/)
2. Double-click the installer to run it
3. Follow the installation wizard
4. When prompted, ensure the "Use WSL 2 instead of Hyper-V" option is selected (recommended)
5. Click Apply & Restart after installation
6. Start Docker Desktop from the Windows Start menu

#### Verifying Installation
Open a terminal or command prompt and run:

```bash
docker --version
docker-compose --version
```

## Exercise Steps and Guides

Links to individual set-up steps and their corresponding guides:


1. [Understanding the E-commerce Data Model](docs/ecommerce.md) - Overview of the e-commerce application structure using the LucidChart diagrams and relational schema.

2. [MongoDB Schema Solution](docs/mongodb-schema-solution.md) - A sample MongoDB schema design for the e-commerce application with explanations of design decisions and pattern usage.
  
3. [Setting Up MongoDB and PostgreSQL Locally](database-setup.md) - Complete instructions for setting up both databases using Docker containers.

4. [Schema Evolution Scenarios](docs/schema-evolution.md) - Practical scenarios for testing schema flexibility in both databases.
