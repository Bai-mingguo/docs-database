# Database Schema

This document outlines the schema design for the AMQ CFD trading platform database.

## Tables

### Users
- id (primary key)
- username
- email
- password_hash
- created_at
- last_login

### Trades
- id (primary key)
- user_id (foreign key to Users)
- instrument_id (foreign key to Instruments)
- quantity
- price
- timestamp

### Instruments
- id (primary key)
- name
- type
- current_price

## Relationships

- Users have many Trades
- Instruments have many Trades

## Indexes

- Users: email (unique)
- Trades: user_id, instrument_id
- Instruments: name
