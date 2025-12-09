# Collateral_Vault_Management_System
# GoQuant Collateral Vault

📘 GoQuant Collateral Vault Management System

Assignment Submission — Anu

This repository contains a complete Collateral Vault Management System built for the GoQuant recruitment assignment.

The project demonstrates secure custody of user collateral (USDT) on Solana using Anchor, SPL Token, Program Derived Addresses (PDAs), and Cross-Program Invocations (CPIs).

The solution includes:

✅ Fully functional Anchor smart contract

✅ Backend service (Rust + Actix Web)

✅ PostgreSQL schema

✅ Anchor tests with a simulated USDT mint

✅ Architecture, CPI flow, security notes

✅ Demo script, PDF, and submission files



🏗 1. Project Overview
In a decentralized perpetual futures exchange, user funds must be held in non-custodial vaults that the protocol controls programmatically. This project implements such a vault system with support for:

Creating user-specific vaults (PDA-derived)

Depositing collateral into the vault

Withdrawing from available balance

Locking/unlocking collateral for margin requirements

Internal transfers for settlement/liquidation

Secure authority management

On-chain/off-chain state synchronization

The backend provides REST APIs and a Postgres store for transaction history and analysis.

🔐 2. Key Features

Smart Contract (Anchor)

Initialize Vault — Creates a PDA vault + associated token account

Deposit — Validates amount, executes SPL token transfer via CPI

Withdraw — Uses PDA authority to sign token transfer back to user

Lock Collateral — Reserves funds for open positions

Unlock Collateral — Releases margin when positions close

Transfer Collateral — For settlement/liquidation engines

Events emitted for indexing: DepositEvent, WithdrawEvent, LockEvent, UnlockEvent, TransferEvent

Security Guarantees

Only vault owner can withdraw

Only authorized programs can lock/unlock

Overflows prevented using checked arithmetic

PDA-signed CPIs ensure program-controlled custody

Vault data is immutable except by program logic


Backend Service



Written in Rust (Actix Web)



Exposes REST endpoints:



POST /vault/initialize



POST /vault/deposit



POST /vault/withdraw



GET /vault/balance/:owner



GET /vault/transactions/:owner



Stores historical transactions in PostgreSQL



Useful for dashboards, analytics, risk checks



🧪 3. How to Run the Project Locally

Prerequisites



Install:



Rust \& Cargo



Anchor CLI



Solana CLI



Node.js (for anchor tests)



PostgreSQL



Step 1 — Start Solana Local Validator

solana-test-validator --reset \&

solana config set --url http://127.0.0.1:8899



Step 2 — Build \& Test Anchor Program

cd programs/collateral\_vault

anchor build

anchor test





This runs:



fake USDT mint creation



ATA creation



initialize vault



deposit → withdraw flow



prints vault balances



Step 3 — Setup PostgreSQL

psql -U postgres -c "CREATE DATABASE vault;"

psql -U postgres -d vault -f db/migrations.sql



Step 4 — Run Backend API

cd backend

export DATABASE\_URL=postgres://postgres:postgres@localhost/vault

cargo run





Backend starts at:

👉 http://127.0.0.1:8080



Step 5 — API Usage Examples

Initialize Vault

curl -X POST http://127.0.0.1:8080/vault/initialize \\

-H "Content-Type: application/json" \\

-d '{"owner\_pubkey":"<PUBKEY>", "vault\_token\_account":"<ATA>"}'



Deposit

curl -X POST http://127.0.0.1:8080/vault/deposit \\

-H "Content-Type: application/json" \\

-d '{"owner\_pubkey":"<PUBKEY>", "amount":1000}'



Withdraw

curl -X POST http://127.0.0.1:8080/vault/withdraw \\

-H "Content-Type: application/json" \\

-d '{"owner\_pubkey":"<PUBKEY>", "amount":500}'



Balance

curl http://127.0.0.1:8080/vault/balance/<PUBKEY>



🏛 4. Project Architecture

Vault PDA

PDA = Pubkey::find\_program\_address(\["vault", user\_pubkey])





Stores:



owner



token account



total / locked / available balances



stats: deposited, withdrawn



bump



Vault Authority PDA

vault\_auth = Pubkey::find\_program\_address(\["vault\_auth", vault\_pubkey])





Used to sign withdrawal CPI.



Event Emission



Events allow:



Realtime balance indexing



Transaction logging



Off-chain analytics



CPI Flows



Deposit: User signs token transfer



Withdraw: PDA signs token transfer



Lock/Unlock: Called by position manager



🗄 5. Database Schema (PostgreSQL)

vault\_accounts

Field	Type	Description

owner\_pubkey	text	user wallet

vault\_pubkey	text	PDA

token\_account\_pubkey	text	ATA

total\_balance	bigint	total funds

locked\_balance	bigint	margin reserved

available\_balance	bigint	free balance

total\_deposited	bigint	lifetime

total\_withdrawn	bigint	lifetime

transactions

Field	Type

vault\_pubkey	text

tx\_type	text (deposit/withdraw/lock/unlock)

amount	bigint

timestamp	datetime

🔍 6. Testing



The included Anchor test:



Creates local mint



Mints USDT to user



Derives vault PDA



Calls initialize → deposit → withdraw



Verifies balances



Logs events



Command:



anchor test



🎥 7. Demo Script Summary



Use the included docs/demo-script.md.



Flow:



Start validator



Run anchor tests



Show vault logs \& events



Start backend



Run curl commands



Brief architecture explanation



Security summary



🔒 8. Security Considerations



PDA authority for withdrawals



No direct user control of vault token account



Checked arithmetic prevents overflow



Input validation for all amounts



Events for auditability



Ready for advanced features (multisig, timelocks, yield routing)

