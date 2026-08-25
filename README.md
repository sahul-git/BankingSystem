# Backend Ledger

A Node.js and Express backend for managing user accounts, ledger balances, and money transfers with MongoDB persistence. The service supports user authentication, account creation, balance queries, and transaction processing with idempotency checks and ledger-based accounting.

## Features

- User registration, login, and logout
- JWT-based authentication with cookie support
- Account creation and per-user account listing
- Account balance retrieval based on ledger entries
- Transaction processing with idempotency keys
- Ledger entry tracking for debit and credit operations
- System-user funding flow for initial account credits
- Email notifications on registration and successful transactions
- MongoDB transactions and session-based processing for financial safety

## Tech Stack

- Node.js
- Express.js
- MongoDB with Mongoose
- JWT for authentication
- bcryptjs for password hashing
- Nodemailer for email alerts
- dotenv for environment configuration

## Project Structure

```bash
backend-ledger-main/
├── server.js
├── package.json
├── .env
├── src/
│   ├── app.js
│   ├── config/
│   │   └── db.js
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── account.controller.js
│   │   └── transaction.controller.js
│   ├── middleware/
│   │   └── auth.middleware.js
│   ├── models/
│   │   ├── account.model.js
│   │   ├── blackList.model.js
│   │   ├── ledger.model.js
│   │   ├── transaction.model.js
│   │   └── user.model.js
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── account.routes.js
│   │   └── transaction.routes.js
│   └── services/
│       └── email.service.js
└── README.md
```

## Prerequisites

Before running the project, make sure you have:

- Node.js installed
- MongoDB running locally or a valid MongoDB connection string
- A Gmail account with SMTP OAuth2 credentials for sending emails (if email notifications are enabled)

## Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd backend-ledger-main
```

2. Install dependencies:

```bash
npm install
```

3. Create a `.env` file in the project root:

```env
MONGO_URI
JWT_SECRET=your_jwt_secret_here
EMAIL_USER=your_email@gmail.com
CLIENT_ID=your_google_client_id
CLIENT_SECRET=your_google_client_secret
REFRESH_TOKEN=your_google_refresh_token
```

4. Start the server:

```bash
npm run dev
```

Or in production mode:

```bash
npm start
```

The server runs on:

```bash
http://localhost:3000
```

## API Endpoints

### Authentication

#### Register a user

```http
POST /api/auth/register
```

Request body:

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "secret123"
}
```

#### Login a user

```http
POST /api/auth/login
```

Request body:

```json
{
  "email": "john@example.com",
  "password": "secret123"
}
```

#### Logout a user

```http
POST /api/auth/logout
```

### Accounts

#### Create an account

```http
POST /api/accounts
```

Requires authentication.

#### Get all accounts for the logged-in user

```http
GET /api/accounts
```

Requires authentication.

#### Check account balance

```http
GET /api/accounts/balance/:accountId
```

Requires authentication.

### Transactions

#### Create a transfer transaction

```http
POST /api/transactions
```

Request body:

```json
{
  "fromAccount": "ACCOUNT_ID",
  "toAccount": "ACCOUNT_ID",
  "amount": 500,
  "idempotencyKey": "txn-123456"
}
```

Requires authentication.

#### Create initial funds transaction for system user

```http
POST /api/transactions/system/initial-funds
```

Request body:

```json
{
  "toAccount": "ACCOUNT_ID",
  "amount": 1000,
  "idempotencyKey": "initial-funds-001"
}
```

Requires system-user authorization.

## How the Ledger Works

The application tracks money movements using ledger entries rather than just updating a single balance field.

- Each debit or credit creates a ledger record
- Account balance is derived from the sum of all ledger entries
- A transaction can be marked as `PENDING`, `COMPLETED`, `FAILED`, or `REVERSED`
- The system uses idempotency keys to prevent duplicate transaction processing
- MongoDB transactions ensure that transaction and ledger records are committed consistently

## Database Models

### User

Stores user credentials and account metadata:

- `email`
- `name`
- `password`
- `systemUser`

### Account

Stores user-owned bank-like accounts:

- `user`
- `status` (`ACTIVE`, `FROZEN`, `CLOSED`)
- `currency` (default `INR`)

### Transaction

Records financial transfers:

- `fromAccount`
- `toAccount`
- `amount`
- `status`
- `idempotencyKey`

### Ledger

Stores each account movement:

- `account`
- `amount`
- `transaction`
- `type` (`DEBIT` or `CREDIT`)

## Authentication Flow

- User logs in or registers
- A JWT token is generated and returned
- The token is stored in a cookie and/or Authorization header
- Protected routes validate the token and blacklist status before allowing access

## Notes

- The app currently uses a hardcoded port of `3000` in [server.js](server.js).
- Email notifications require valid OAuth2 credentials in the environment.
- The app assumes MongoDB is available before startup and will exit if the database connection fails.
- Some behavior, such as the initial funds system route, depends on the user being marked as a system user in the database.

## Default Health Check

```http
GET /
```

Response:

```text
Ledger Service is up and running
```


