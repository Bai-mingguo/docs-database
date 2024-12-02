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
- role_id (BIGINT, foreign key to Roles.id)

### Roles
- id (BIGINT, primary key, auto-increment)
- name (VARCHAR(50), unique, not null)
- description (TEXT)

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
- status (ENUM('pending', 'executed', 'cancelled', 'rejected'), not null)
- created_at (TIMESTAMP, not null, default: current_timestamp)
- executed_at (TIMESTAMP)

### Positions
- id (BIGINT, primary key, auto-increment)
- user_id (BIGINT, foreign key to Users.id)
- account_id (BIGINT, foreign key to Accounts.id)
- instrument_id (BIGINT, foreign key to Instruments.id)
- quantity (DECIMAL(15,5), not null)
- entry_price (DECIMAL(15,5), not null)
- current_price (DECIMAL(15,5), not null)
- unrealized_pnl (DECIMAL(15,2), not null)
- opened_at (TIMESTAMP, not null)
- last_updated (TIMESTAMP, not null, default: current_timestamp on update current_timestamp)

### Transactions
- id (BIGINT, primary key, auto-increment)
- account_id (BIGINT, foreign key to Accounts.id)
- type (ENUM('deposit', 'withdrawal', 'trade_profit', 'trade_loss', 'fee', 'adjustment'), not null)
- amount (DECIMAL(15,2), not null)
- description (TEXT)
- created_at (TIMESTAMP, not null, default: current_timestamp)
- related_order_id (BIGINT, foreign key to Orders.id)
- related_position_id (BIGINT, foreign key to Positions.id)

### UserRiskParameters
- id (BIGINT, primary key, auto-increment)
- user_id (BIGINT, foreign key to Users.id)
- max_leverage (DECIMAL(5,2), not null)
- max_position_size (DECIMAL(15,2), not null)
- max_daily_loss (DECIMAL(15,2), not null)
- last_updated (TIMESTAMP, not null, default: current_timestamp on update current_timestamp)

### MarginUsage
- id (BIGINT, primary key, auto-increment)
- account_id (BIGINT, foreign key to Accounts.id)
- used_margin (DECIMAL(15,2), not null)
- available_margin (DECIMAL(15,2), not null)
- margin_level (DECIMAL(5,2), not null)  # (Equity / Used Margin) * 100
- timestamp (TIMESTAMP, not null, default: current_timestamp)

### AuditLogs
- id (BIGINT, primary key, auto-increment)
- user_id (BIGINT, foreign key to Users.id)
- action (VARCHAR(100), not null)
- details (TEXT)
- ip_address (VARCHAR(45), not null)  # IPv6 compatible
- timestamp (TIMESTAMP, not null, default: current_timestamp)

### KYC_AML_Data
- id (BIGINT, primary key, auto-increment)
- user_id (BIGINT, foreign key to Users.id, unique)
- id_type (ENUM('passport', 'national_id', 'drivers_license'), not null)
- id_number (VARCHAR(100), not null)
- id_expiry_date (DATE, not null)
- proof_of_address_type (ENUM('utility_bill', 'bank_statement', 'government_letter'), not null)
- proof_of_address_date (DATE, not null)
- verification_status (ENUM('pending', 'approved', 'rejected'), not null, default: 'pending')
- verification_date (TIMESTAMP)
- verified_by (BIGINT, foreign key to Users.id)  # The admin who verified the documents
- additional_notes (TEXT)

## Relationships

- Users have one Role (many-to-one)
- Users have many Accounts (one-to-many)
- Users have one UserRiskParameters (one-to-one)
- Users have one KYC_AML_Data (one-to-one)
- Accounts have many Orders (one-to-many)
- Accounts have many Positions (one-to-many)
- Accounts have many Transactions (one-to-many)
- Accounts have many MarginUsage records (one-to-many)
- Instruments have many Prices (one-to-many)
- Instruments have many Orders (one-to-many)
- Instruments have many Positions (one-to-many)
- Orders can have many Transactions (one-to-many)
- Positions can have many Transactions (one-to-many)

## Indexes

- Users: email (unique), username (unique), role_id
- Roles: name (unique)
- Accounts: user_id
- Instruments: symbol (unique)
- Prices: instrument_id, timestamp
- Orders: user_id, account_id, instrument_id, status
- Positions: user_id, account_id, instrument_id
- Transactions: account_id, type, created_at, related_order_id, related_position_id
- UserRiskParameters: user_id (unique)
- MarginUsage: account_id, timestamp
- AuditLogs: user_id, action, timestamp
- KYC_AML_Data: user_id (unique)

## Notes

1. All DECIMAL fields are used for precise financial calculations.
2. Timestamps are used to track creation and update times for auditing purposes.
3. ENUM types are used where applicable to ensure data integrity.
4. Foreign keys are used to maintain referential integrity between related tables.
5. Indexes are created on frequently queried columns to improve performance.
6. The new Roles table allows for flexible user role management.
7. UserRiskParameters table stores individual risk settings for each user.
8. MarginUsage table tracks real-time margin usage for each account.
9. AuditLogs table provides a comprehensive audit trail for all important actions.
10. KYC_AML_Data table stores detailed Know Your Customer and Anti-Money Laundering information.
11. Transactions table has been expanded to include more details and relationships to orders and positions.

This updated schema provides a more comprehensive foundation for the AMQ CFD trading platform. It covers user management, role-based access control, account handling, instrument details, real-time pricing, order management, position tracking, risk management, compliance, and financial transactions. As the project evolves, we may need to add additional tables or modify existing ones to meet new requirements.
