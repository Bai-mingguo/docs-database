# Database Schema

This document outlines the comprehensive schema design for the AMQ CFD trading platform database.

## Tables

### Users
- id (BIGINT, primary key, auto-increment)
- username (VARCHAR(50), unique, not null)
- email (VARCHAR(100), unique, not null)
- password_hash (VARCHAR(255), not null)
- full_name (VARCHAR(100), not null)
- date_of_birth (DATE, not null)
- country (VARCHAR(50), not null)
- created_at (TIMESTAMP, not null, default: current_timestamp)
- last_login (TIMESTAMP)
- account_status (ENUM('active', 'suspended', 'closed'), not null, default: 'active')
- verification_status (ENUM('unverified', 'pending', 'verified'), not null, default: 'unverified')

### Accounts
- id (BIGINT, primary key, auto-increment)
- user_id (BIGINT, foreign key to Users.id)
- account_type (ENUM('demo', 'live'), not null)
- currency (VARCHAR(3), not null)
- balance (DECIMAL(15,2), not null, default: 0)
- created_at (TIMESTAMP, not null, default: current_timestamp)
- last_updated (TIMESTAMP, not null, default: current_timestamp on update current_timestamp)

### Instruments
- id (BIGINT, primary key, auto-increment)
- symbol (VARCHAR(20), unique, not null)
- name (VARCHAR(100), not null)
- type (ENUM('forex', 'stocks', 'commodities', 'indices', 'cryptocurrencies'), not null)
- pip_value (DECIMAL(10,5), not null)
- min_trade_size (DECIMAL(10,5), not null)
- max_trade_size (DECIMAL(10,5), not null)
- margin_requirement (DECIMAL(5,2), not null)  # Stored as a percentage

### Prices
- id (BIGINT, primary key, auto-increment)
- instrument_id (BIGINT, foreign key to Instruments.id)
- bid (DECIMAL(15,5), not null)
- ask (DECIMAL(15,5), not null)
- timestamp (TIMESTAMP, not null)

### Orders
- id (BIGINT, primary key, auto-increment)
- user_id (BIGINT, foreign key to Users.id)
- account_id (BIGINT, foreign key to Accounts.id)
- instrument_id (BIGINT, foreign key to Instruments.id)
- order_type (ENUM('market', 'limit', 'stop'), not null)
- side (ENUM('buy', 'sell'), not null)
- quantity (DECIMAL(15,5), not null)
- price (DECIMAL(15,5))  # Null for market orders
- status (ENUM('pending
