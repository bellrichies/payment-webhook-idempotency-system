# Payment Webhook Idempotency System

A production-grade backend system that demonstrates how to safely process payment provider webhooks using event validation, signature verification, idempotency enforcement, transactional processing, and duplicate transaction prevention.

## Overview

Payment providers often retry webhook delivery when an endpoint times out, returns a non-2xx response, or encounters temporary network issues. Without proper protection, duplicate webhook deliveries can result in duplicate credits, repeated order fulfillment, inconsistent balances, and corrupted transaction state.

This project solves that by implementing:

- webhook endpoint simulation
- event storage and tracking
- signature verification
- provider event idempotency
- business-action idempotency
- duplicate transaction prevention
- safe retry handling
- audit logs for observability

## Key Features

- Webhook endpoint simulation
- Raw event storage for auditability
- Idempotency key enforcement
- Duplicate webhook detection
- Duplicate transaction prevention
- Retry-safe event processing
- Signature verification (mock or real)
- Provider abstraction for multi-gateway support
- Transaction-safe processing with MySQL
- Admin endpoints to inspect and retry failed events

## Tech Stack

- PHP OOP with MVC architecture
- MySQL
- Composer with PSR-4 autoloading
- vlucas/phpdotenv
- REST-style JSON API

## Why This Project Matters

Real-world payment systems must assume:
- webhooks will be retried
- events may arrive more than once
- events may arrive out of order
- processing may fail midway
- financial side effects must never be duplicated

This repository demonstrates the correct architectural approach for building reliable webhook consumers.

## Core Reliability Guarantees

This system enforces duplicate protection at multiple layers:

1. **Event idempotency**
   - prevents the same external provider event from being processed twice

2. **Transaction uniqueness**
   - prevents duplicate payment transaction records for the same provider reference

3. **Business-effect idempotency**
   - prevents the same payment effect from being applied twice even during retries

4. **Transactional processing**
   - ensures payment status updates and ledger writes succeed or fail together

## Project Structure

```text
payment-webhook-idempotency-system/
├── app/
├── bootstrap/
├── database/
├── public/
├── routes/
├── storage/
├── tests/
├── .env.example
├── composer.json
└── README.md
````

## Main Modules

### Webhook Module

Handles:

* receiving webhook requests
* payload parsing
* signature verification
* raw event persistence
* duplicate detection

### Payment Processing Module

Handles:

* payment transaction creation/update
* business effect application
* safe state transitions
* ledger entry creation

### Retry Module

Handles:

* retrying failed or interrupted events
* preserving idempotency during reprocessing

### Admin Inspection Module

Handles:

* viewing stored events
* inspecting failures
* retrying failed events
* listing transaction records

## Example Use Case

A payment provider sends `charge.success` for transaction `TXN_123`.

The webhook arrives once and is processed successfully:

* raw event is stored
* transaction is marked successful
* ledger entry is inserted
* event is marked processed

If the provider retries the same event:

* the duplicate is detected using provider event uniqueness
* no second ledger entry is written
* no second transaction effect occurs

## API Endpoints

### Health Check

`GET /api/v1/health`

### Receive Payment Webhook

`POST /api/v1/webhooks/payments/{provider}`

### Simulate Webhook

`POST /api/v1/simulator/webhooks/{provider}`

### List Webhook Events

`GET /api/v1/admin/webhook-events`

### View Webhook Event

`GET /api/v1/admin/webhook-events/{id}`

### Retry Failed Webhook Event

`POST /api/v1/admin/webhook-events/{id}/retry`

### List Payment Transactions

`GET /api/v1/admin/payment-transactions`

### View Payment Transaction

`GET /api/v1/admin/payment-transactions/{id}`

## Database Tables

* `webhook_events`
* `payment_transactions`
* `ledger_entries`
* `webhook_processing_logs`
* `processed_business_actions`

## Event Lifecycle

Webhook events move through these states:

* `received`
* `processing`
* `processed`
* `failed`
* `duplicate`
* `invalid`
* `ignored`

## Environment Variables

Create a `.env` file from `.env.example`.

Example:

```env
APP_NAME=Payment Webhook Idempotency System
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=payment_webhook_system
DB_USERNAME=root
DB_PASSWORD=

WEBHOOK_DEFAULT_PROVIDER=mock
WEBHOOK_MOCK_SECRET=super-secret-key
WEBHOOK_PAYSTACK_SECRET=your-paystack-secret
WEBHOOK_FLUTTERWAVE_SECRET=your-flutterwave-secret
```

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/payment-webhook-idempotency-system.git
cd payment-webhook-idempotency-system
```

### 2. Install dependencies

```bash
composer install
```

### 3. Copy environment file

```bash
cp .env.example .env
```

### 4. Configure database

Update your `.env` with MySQL credentials.

### 5. Run migrations

Import SQL files from `database/migrations`.

### 6. Start local server

Point your web server to the `public/` directory.

## How Idempotency Works

This project uses multi-layer idempotency:

### Layer 1: Provider event uniqueness

Each webhook must have a unique `(provider, provider_event_id)` pair.

### Layer 2: Payment transaction uniqueness

Each provider payment reference must be unique.

### Layer 3: Business action uniqueness

Each business effect is recorded using a unique action key.

### Layer 4: Ledger entry uniqueness

Financial entries can be protected by unique constraints to stop duplicate postings.

## Signature Verification

Incoming requests are verified using HMAC signatures. The raw request body is hashed with the configured provider secret and compared using constant-time comparison.

## Recommended Test Scenarios

* valid webhook processes successfully
* duplicate webhook is ignored safely
* invalid signature is rejected
* missing required fields returns validation error
* failed event can be retried safely
* retry does not duplicate financial effects
* same payment reference cannot be inserted twice

## Future Enhancements

* queue-based asynchronous processing
* dead-letter retry mechanism
* provider callback IP allowlisting
* Prometheus metrics / monitoring
* OpenAPI / Swagger docs
* Docker setup
* admin dashboard UI
* webhook replay console
* Redis locking for distributed workers

