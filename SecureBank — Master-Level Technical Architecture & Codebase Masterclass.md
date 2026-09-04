# SecureBank — Complete Technical Masterclass

## 0. Executive Assessment

**SecureBank is a React + Express banking/payment prototype whose defining feature is offline-resilient payments.**

The application attempts to solve a real distributed-systems problem:

> **How can a user initiate a payment when their browser temporarily has no network connection, while still giving the user an immediate experience and synchronizing the transaction with the backend when connectivity returns?**

The project combines:

- **React 19** for the frontend
- **Vite** for frontend development/building
- **Express 5** for the backend API
- **SQLite** for the server-side relational database
- **IndexedDB** for client-side persistent storage
- **Service Worker** for PWA/background-sync concepts
- **JWT** for authentication
- **Twilio** for OTP delivery
- **bcryptjs** for PIN hashing
- **CryptoJS AES** for local browser-data encryption
- **Axios** for API communication

Architecturally, the project is best understood as:

```text
                    SECUREBANK
                       |
          +------------+------------+
          |                         |
       FRONTEND                  BACKEND
       React                    Express
          |                         |
   +------+-------+          +------+-------+
   |      |       |          |      |       |
 Auth   Local   Network     Auth  Banking  DB
        DB      Manager          Logic    SQLite
   |      |       |          |      |       |
   +------+-------+          +------+-------+
          |                         |
       IndexedDB                 SQLite
          |
    Offline Queue
          |
     Connectivity
       Restored
          |
          +--------> /sync/transactions
```

### Most important architectural idea

The project does **not** make the banking server itself offline.

Instead:

```text
Browser
  |
  | offline
  v
IndexedDB
  |
  | transaction queued
  v
Sync Queue
  |
  | internet returns
  v
Express API
  |
  v
SQLite
```

Therefore, the system implements **offline transaction intent + eventual synchronization**, rather than true offline settlement.

That distinction is extremely important in a technical interview.

---

# 1. Repository Structure

The uploaded repository contains approximately 53 files.

The meaningful application structure is:

```text
SecureBank-main/
│
├── README.md
│
└── banking-app/
    │
    ├── client/
    │   ├── package.json
    │   ├── vite.config.js
    │   ├── index.html
    │   │
    │   └── src/
    │       ├── App.jsx
    │       ├── main.jsx
    │       ├── index.css
    │       ├── sw.js
    │       │
    │       ├── contexts/
    │       │   ├── AuthContext.jsx
    │       │   └── NetworkContext.jsx
    │       │
    │       ├── services/
    │       │   ├── api.js
    │       │   ├── database.js
    │       │   └── -- SQLite.sql
    │       │
    │       └── components/
    │           ├── AuthForm.jsx
    │           ├── BankAccountSetup.jsx
    │           ├── Dashboard.jsx
    │           ├── EncryptedVaultCard.jsx
    │           ├── InstantSyncCard.jsx
    │           ├── LandingPage.jsx
    │           ├── NetworkStatus.jsx
    │           ├── OfflineModeCard.jsx
    │           ├── PayXEcosystem.jsx
    │           ├── PaymentStatusOverlay.jsx
    │           ├── PhoneMockUp.jsx
    │           ├── QRPaymentsCard.jsx
    │           ├── SecurityBadge.jsx
    │           ├── StackedFeatures.jsx
    │           ├── TransactionForm.jsx
    │           ├── TransactionList.jsx
    │           └── UserProfileDrawer.jsx
    │
    └── server/
        ├── package.json
        ├── index.js
        ├── migrate.js
        ├── jsconfig.json
        └── bank.db
```

---

# 2. What Problem Does SecureBank Solve?

## 2.1 Normal banking application

A traditional payment application behaves approximately like:

```text
User
 |
 | payment request
 v
Frontend
 |
 | HTTP
 v
Backend
 |
 | SQL
 v
Database
 |
 v
Success
```

If the network disappears:

```text
User
 |
 v
Frontend
 |
 X
Internet unavailable
```

The user cannot complete the operation.

---

# 2.2 SecureBank's approach

SecureBank introduces a local persistence layer.

```text
                    ONLINE
                      |
                      v
User -> React -> Express -> SQLite
                      ^
                      |
                   response


                    OFFLINE
                      |
                      v
User -> React -> IndexedDB
                      |
                      v
                 Sync Queue
                      |
                 wait...
                      |
                network returns
                      |
                      v
                 Express API
                      |
                      v
                   SQLite
```

This creates an **eventual consistency architecture**.

The frontend can immediately record:

```text
Transaction:
    id = tx_...
    amount = ₹500
    sender = account A
    recipient = account B
    status = syncing
```

and later ask the server to permanently apply it.

---

# 3. Critical Terminology: Offline Payment vs Offline Transaction

This is one of the most important concepts to understand.

The project calls itself an offline payment system.

Technically, however, the implementation is closer to:

> **Offline transaction queuing with eventual server-side settlement.**

Why?

Because when offline:

```text
Server database
    |
    | does not know about transaction
    v
No actual server-side balance transfer
```

The browser merely records the user's intention.

Only after synchronization:

```text
IndexedDB
    |
    | POST /sync/transactions
    v
Express
    |
    v
SQLite
    |
    +--> debit sender
    |
    +--> credit recipient
    |
    +--> record transaction
```

Therefore:

### User experience

Offline:

> "Payment initiated."

### Actual financial state

Offline:

> "Payment has not yet been settled by the authoritative server."

This distinction would be important in production banking architecture.

---

# 4. Architectural Style

The project is primarily a:

## Layered Client-Server Architecture

with elements of:

- SPA architecture
- Offline-first architecture
- Event-driven browser communication
- Eventually consistent synchronization
- REST API architecture
- Local persistence
- Optimistic UI updates

It is **not** actually microservices.

There is one Express application:

```text
Express
 |
 +-- Authentication
 +-- Account management
 +-- Bank account management
 +-- Transaction processing
 +-- Synchronization
 +-- Health check
```

This is essentially a **modular monolith**.

---

# 5. High-Level Architecture

```text
┌───────────────────────────────────────────────────────────┐
│                     Browser / Client                      │
│                                                           │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                  React Application                  │  │
│  │                                                     │  │
│  │  Landing Page                                       │  │
│  │       │                                             │  │
│  │       ├── Authentication                            │  │
│  │       │                                             │  │
│  │       └── Dashboard                                 │  │
│  │              │                                      │  │
│  │              ├── Bank Accounts                      │  │
│  │              ├── Transactions                       │  │
│  │              ├── QR Scanner                         │  │
│  │              └── Network Status                     │  │
│  │                                                     │  │
│  └──────────────────────┬──────────────────────────────┘  │
│                         │                                 │
│              ┌──────────┴──────────┐                      │
│              │                     │                      │
│         ApiService            LocalDatabase               │
│              │                     │                      │
│          Axios               IndexedDB                    │
│              │                     │                      │
│              │               ┌─────┴──────┐               │
│              │               │            │               │
│              │          Transactions   SyncQueue           │
│              │               │            │               │
│              │               └────────────┘               │
│              │                                            │
└──────────────┼────────────────────────────────────────────┘
               │
               │ HTTPS / REST
               ▼
┌───────────────────────────────────────────────────────────┐
│                    Express Backend                        │
│                                                           │
│ Helmet                                                    │
│ CORS                                                      │
│ Rate Limiting                                             │
│ JWT Authentication                                        │
│                                                           │
│ /auth/*                                                   │
│ /account/*                                                │
│ /bank-accounts/*                                          │
│ /transactions                                             │
│ /sync/transactions                                        │
│ /health                                                   │
│                                                           │
│                         │                                 │
│                         ▼                                 │
│                     SQLite                                │
│                                                           │
│       users                                               │
│       bank_accounts                                       │
│       transactions                                        │
└───────────────────────────────────────────────────────────┘
```

---

# 6. Frontend Architecture

## 6.1 Entry point

`client/src/main.jsx`

This is the browser bootstrap.

Conceptually:

```javascript
createRoot(document.getElementById('root'))
    .render(
        <StrictMode>
            <App />
        </StrictMode>
    );
```

It also registers the Service Worker.

Therefore:

```text
Browser
  |
  v
main.jsx
  |
  +--> Service Worker registration
  |
  +--> React root
          |
          v
        App.jsx
```

---

# 7. App.jsx — Application Composition Root

`App.jsx` establishes the major application contexts.

```text
AuthProvider
    |
    └── NetworkProvider
            |
            └── AppContent
```

This is important.

Instead of passing authentication and network state through dozens of components:

```text
Dashboard
   |
   +--> Auth
   +--> Network
   +--> Transaction
```

React Context provides shared application state.

---

# 8. Authentication Architecture

The authentication flow is:

```text
                    SIGN UP / LOGIN

User enters phone number
          |
          v
POST /auth/request-otp
          |
          v
Express
          |
          v
Generate OTP
          |
          v
otpStore Map
          |
          v
Twilio SMS
          |
          v
User receives OTP
          |
          v
POST /auth/verify-otp
          |
          v
Validate OTP
          |
          v
Find user in SQLite
          |
       +--+--+
       |     |
    Exists  New
       |     |
       |    INSERT
       |     |
       +-----+
          |
          v
Generate JWT
          |
          v
Return token + user
          |
          v
Frontend localStorage
```

---

# 9. AuthContext.jsx

The purpose of `AuthContext.jsx` is to abstract authentication from UI components.

It exposes:

```javascript
user
requestOtp()
verifyOtp()
logout()
isLoading
updateUserBankAccounts()
```

This means `Dashboard.jsx` does not need to know how authentication works.

It simply says:

```javascript
const { user, logout } = useAuth();
```

That is a good separation of concerns.

---

# 10. JWT Authentication

The server generates:

```text
JWT
 |
 +-- user.id
 +-- phone_number
 +-- name
 |
 +-- expiresIn: 24h
```

The browser stores the token:

```text
localStorage
    |
    └── auth_token
```

Axios automatically attaches:

```http
Authorization: Bearer [JWT]
```

The server middleware extracts:

```javascript
req.headers['authorization']
```

then:

```text
Bearer token
     |
     v
JWT verify
     |
     v
req.user
```

Protected endpoints use:

```javascript
authenticateToken
```

---

# 11. Authentication Strengths

The implementation has several good engineering choices.

### JWT verification is centralized

Instead of repeating:

```text
verify token
verify token
verify token
```

the middleware handles it once.

### Token expiry exists

```text
24h
```

is better than an indefinitely valid token.

### PIN is not stored as plaintext in SQLite

The bank account stores:

```text
pin_hash
```

rather than:

```text
pin
```

and uses bcrypt.

---

# 12. Authentication Weaknesses

There are major production concerns.

## 12.1 JWT in localStorage

```text
localStorage.auth_token
```

is vulnerable to token theft through XSS.

A production design would typically consider:

```text
HttpOnly
Secure
SameSite
```

cookies, depending on architecture.

---

## 12.2 No refresh-token architecture

There is only:

```text
24-hour JWT
```

There is no:

```text
refresh token
token rotation
revocation mechanism
device session management
```

---

## 12.3 OTP is stored in process memory

```javascript
const otpStore = new Map();
```

This means:

```text
Server process A
    |
    +--> OTP memory
```

If the process restarts:

```text
OTP data disappears
```

More importantly, if there are multiple backend instances:

```text
              Load Balancer
             /            \
            v              v
        Server A        Server B
        OTP Map         OTP Map
```

OTP created on A may be verified on B.

Therefore distributed deployments require shared storage such as:

```text
Redis
```

or a database.

---

# 13. OTP Security Problems

The implementation generates:

```javascript
Math.random()
```

for the OTP.

For security-sensitive authentication, cryptographically secure randomness should be used.

The backend also logs:

```text
Generated OTP: [OTP]
```

That is dangerous in a production banking environment.

Logs should never contain authentication secrets.

---

# 14. Rate Limiting

The backend includes:

```javascript
express-rate-limit
```

Two limiters exist conceptually:

### Global

```text
100 requests / minute / IP
```

### OTP

```text
3 requests / minute / phone number
```

This is intended to mitigate:

- OTP spam
- brute force
- API abuse
- accidental request storms

The concept is correct, although the implementation contains configuration/comment inconsistencies.

---

# 15. Backend Architecture

`server/index.js` contains essentially the entire backend.

This is both a strength and weakness.

### Strength

Very easy to understand:

```text
index.js
    |
    +-- middleware
    +-- DB
    +-- auth
    +-- bank accounts
    +-- transactions
    +-- synchronization
```

### Weakness

The file has become a large monolithic controller.

A production structure should eventually become:

```text
server/
├── src/
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── account.routes.js
│   │   ├── bank.routes.js
│   │   └── transaction.routes.js
│   │
│   ├── controllers/
│   ├── services/
│   ├── repositories/
│   ├── middleware/
│   ├── validators/
│   └── config/
│
└── app.js
```

---

# 16. Database Design

The SQLite database contains three major tables.

```text
users
  |
  | 1:N
  v
bank_accounts
  |
  | 1:N
  v
transactions
```

More precisely:

```text
users
 |
 +---- bank_accounts
          |
          +---- transactions.from_bank_account_id
          |
          +---- transactions.to_bank_account_id
```

---

# 17. Users Table

Conceptually:

```sql
users (
    id TEXT PRIMARY KEY,
    phone_number TEXT UNIQUE NOT NULL,
    name TEXT,
    created_at DATETIME
)
```

### Responsibility

Represents application users.

Important properties:

```text
id
phone_number
name
created_at
```

The UUID provides a non-sequential identifier.

---

# 18. Bank Accounts Table

Conceptually:

```sql
bank_accounts (
    id TEXT PRIMARY KEY,
    user_id TEXT NOT NULL,
    bank_name TEXT NOT NULL,
    account_number TEXT UNIQUE NOT NULL,
    ifsc_code TEXT NOT NULL,
    branch TEXT NOT NULL,
    pin_hash TEXT NOT NULL,
    balance REAL DEFAULT 0,
    is_primary BOOLEAN DEFAULT 0,
    created_at DATETIME
)
```

This represents a linked financial account.

Relationship:

```text
User
 |
 +--> Account 1
 |
 +--> Account 2
 |
 +--> Account 3
```

---

# 19. Why Store `pin_hash`?

The system uses:

```javascript
bcrypt.hashSync(pin, 10)
```

This transforms:

```text
1234
```

into a salted bcrypt hash.

Conceptually:

```text
PIN
 |
 v
bcrypt
 |
 v
salted hash
```

During payment:

```text
entered PIN
      |
      v
bcrypt.compare
      |
      +---- match ---> authorized
      |
      +---- mismatch -> rejected
```

This is significantly safer than plaintext PIN storage.

---

# 20. Major Banking Data Modeling Problem: `REAL`

Balances are stored as:

```sql
balance REAL
```

This is inappropriate for production monetary calculations.

Floating-point numbers can produce precision problems.

For example:

```text
0.1 + 0.2
```

is not necessarily represented exactly in binary floating point.

A financial system should generally use:

```text
integer minor units
```

For INR:

```text
₹100.50
```

can be represented as:

```text
10050 paise
```

Therefore:

```sql
balance_paise INTEGER
```

is much safer.

---

# 21. Transactions Table

Conceptually:

```sql
transactions (
    id TEXT PRIMARY KEY,
    from_bank_account_id TEXT NOT NULL,
    to_bank_account_id TEXT NOT NULL,
    amount REAL NOT NULL,
    type TEXT NOT NULL,
    description TEXT,
    status TEXT DEFAULT 'pending',
    created_at DATETIME,
    client_timestamp DATETIME,
    synced INTEGER DEFAULT 0
)
```

The transaction table is effectively a ledger/event record.

---

# 22. Why Two Account IDs?

A transfer is fundamentally:

```text
FROM account
      |
      | amount
      v
TO account
```

Therefore:

```text
from_bank_account_id
to_bank_account_id
```

are required.

This also allows transaction history to determine:

```text
outgoing
```

versus:

```text
incoming
```

---

# 23. Normal Online Transaction Flow

When online, the frontend calls:

```http
POST /transactions
```

The server performs:

```text
Authenticate JWT
       |
       v
Validate request
       |
       v
Find sender account
       |
       v
Verify ownership
       |
       v
Verify PIN
       |
       v
Check balance
       |
       v
Find recipient
       |
       v
Prevent self-transfer
       |
       v
BEGIN TRANSACTION
       |
       +--> sender balance -= amount
       |
       +--> recipient balance += amount
       |
       +--> insert transaction
       |
       v
COMMIT
```

This is the correct conceptual shape for a money transfer.

---

# 24. Why Database Transactions Matter

Imagine:

```text
Sender -= ₹500
```

succeeds.

Then:

```text
Recipient += ₹500
```

fails.

Without a database transaction:

```text
Sender = -₹500
Recipient = unchanged
```

Money disappears.

With:

```sql
BEGIN
...
ROLLBACK
```

the entire operation can be undone.

Therefore:

```text
BEGIN
  debit
  credit
  record
COMMIT
```

provides atomicity.

---

# 25. ACID Concepts

The project is implicitly using ACID principles.

## Atomicity

Either:

```text
debit + credit + ledger record
```

all happen, or none happen.

## Consistency

Database constraints and application validation attempt to maintain valid state.

## Isolation

Concurrent transactions should not interfere with each other incorrectly.

## Durability

After commit, SQLite persists the transaction.

---

# 26. Offline Transaction Flow

This is the core innovation.

The frontend creates:

```javascript
const transaction = {
    id,
    fromBankAccountId,
    toAccountNumber,
    toIfscCode,
    toBranch,
    amount,
    type,
    description,
    senderPin,
    status: 'syncing'
}
```

Then:

```text
                createTransaction()
                        |
             +----------+----------+
             |                     |
             v                     v
        IndexedDB              Sync Queue
             |                     |
             +----------+----------+
                        |
                        v
                Optimistic balance
                        |
                        v
                  UI immediately
```

---

# 27. IndexedDB Architecture

The local database is called:

```text
BankingApp
```

Version:

```text
5
```

It contains four object stores.

```text
BankingApp
│
├── transactions
│
├── userData
│
├── syncQueue
│
└── usersCache
```

---

# 28. Transactions Store

Primary key:

```text
id
```

Indexes include:

```text
status
createdAt
fromBankAccountId
toBankAccountId
```

This enables local queries such as:

```text
get transactions
get pending transactions
find by account
```

---

# 29. User Data Store

Key:

```text
key
```

The application stores:

```text
user
```

as encrypted data.

This enables offline account display.

---

# 30. Sync Queue

The sync queue is the most important data structure for offline operation.

Example:

```json
{
  "id": "tx_123",
  "type": "transaction",
  "data": {
    "...": "..."
  },
  "priority": 1,
  "attempts": 0,
  "createdAt": "[TIMESTAMP]"
}
```

Conceptually:

```text
Offline transaction
       |
       v
Queue
       |
       +--> waiting
       |
       +--> network restored
       |
       v
POST /sync/transactions
```

This is essentially a **durable client-side work queue**.

---

# 31. Users Cache

The fourth store:

```text
usersCache
```

contains indexes for:

```text
accountNumber
phoneNumber
```

It appears intended to make recipient lookup possible locally.

However, the current transaction lookup implementation primarily calls the server, so this cache is not fully integrated into the offline payment path.

---

# 32. Local Encryption

The browser database uses:

```javascript
CryptoJS.AES
```

for selected data.

Conceptually:

```text
Sensitive object
       |
       v
JSON.stringify()
       |
       v
AES encryption
       |
       v
IndexedDB
```

Reading:

```text
IndexedDB
   |
   v
AES decrypt
   |
   v
JSON.parse
   |
   v
Object
```

---

# 33. Important Security Caveat

The encryption key is hardcoded:

```text
[your-encryption-key-here-change-in-production]
```

This is **not real secret management**.

Because the JavaScript application must have access to the key:

```text
Browser
  |
  +--> application code
  |
  +--> encryption key
```

an attacker who can execute JavaScript in the page can potentially obtain the key.

Therefore browser-side AES here primarily provides:

> protection against casual/plain local inspection

rather than:

> strong protection against a fully compromised browser environment.

---

# 34. Even More Important: PIN Storage

The offline transaction object includes:

```javascript
senderPin
```

It is encrypted before being persisted.

So the implementation does not store the PIN as plaintext in the IndexedDB transaction object.

However, the design still has a major security issue:

```text
PIN
 |
 v
Browser
 |
 v
Encrypted IndexedDB
 |
 v
Stored until synchronization
```

A banking-grade architecture should avoid retaining reusable authentication secrets in a client-side queue.

The better design is to use a cryptographic transaction authorization mechanism that does not require storing the user's reusable PIN.

---

# 35. Network Detection

The project uses:

```javascript
navigator.onLine
```

and browser:

```text
online
offline
```

events.

Architecture:

```text
Browser
 |
 +--> online event
 |
 +--> offline event
```

The API service updates:

```javascript
isOnline
```

and dispatches custom events.

---

# 36. Why `navigator.onLine` Is Not Enough

This is an important distributed-systems concept.

`navigator.onLine === true` means approximately:

> the browser believes some network connection exists.

It does **not** guarantee:

```text
Internet works
API server works
DNS works
TLS works
backend is healthy
database is healthy
```

The project correctly adds a second concept:

```text
isServerHealthy
```

using `/health`.

Therefore:

```text
Connectivity
+
Server health
=
better decision about synchronization
```

That is a good design direction.

---

# 37. Health Check

The frontend periodically calls:

```http
GET /health
```

Every approximately:

```text
15 seconds
```

when online.

The server returns:

```json
{
  "status": "healthy",
  "timestamp": "[TIMESTAMP]",
  "uptime": "[SECONDS]"
}
```

This provides basic liveness information.

---

# 38. Event-Driven Browser Communication

The application uses custom browser events:

```text
networkStatusChanged
serverHealthChanged
syncStarted
transactionsSynced
syncFailed
transactionCreated
```

Example:

```javascript
window.dispatchEvent(
    new CustomEvent('transactionsSynced')
);
```

Other components listen:

```javascript
window.addEventListener(
    'transactionsSynced',
    handler
);
```

This is effectively a lightweight event bus.

---

# 39. Why This Is Useful

Without events, components would need tightly coupled references:

```text
API service
    |
    +--> Dashboard
    +--> NetworkStatus
    +--> AuthContext
```

Instead:

```text
API service
      |
      v
  Event Bus
   /   |   \
  v    v    v
Auth Dashboard Network
```

This reduces direct coupling.

---

# 40. But Browser Custom Events Have a Limitation

The event architecture is implicit.

There is no central event definition such as:

```javascript
EventBus.emit('TRANSACTION_SYNCED')
```

Therefore:

- event names are string literals
- payload contracts are informal
- debugging becomes harder
- type safety is weak
- dependencies are hidden

A mature architecture could introduce a typed application event system.

---

# 41. Optimistic UI

The offline transaction code immediately deducts the sender's cached balance:

```text
local balance
     |
     - amount
     |
     v
new local balance
```

This gives the user instant feedback.

That is an example of:

> **Optimistic state update**

Instead of:

```text
wait for server
    |
    v
update UI
```

it does:

```text
update UI immediately
    |
    v
server confirmation later
```

This is common in offline-first applications.

---

# 42. Optimistic Updates Introduce Risk

Suppose:

```text
Balance = ₹1000
```

User attempts:

```text
₹800 payment
```

Offline UI becomes:

```text
₹200
```

But later server synchronization could fail because:

```text
PIN invalid
recipient missing
insufficient funds
server conflict
```

The browser then needs to reconcile:

```text
optimistic state
        |
        v
authoritative state
```

The current implementation does not fully implement robust compensation/reconciliation.

This is one of the largest architectural gaps.

---

# 43. Critical Sync Design Problem #1 — Duplicate Transactions

This is probably the most important technical issue in the current implementation.

Consider:

```text
Client
 |
 | POST /sync/transactions
 v
Server
 |
 +--> debit
 +--> credit
 +--> commit
 |
 v
response
```

Suppose the server successfully commits.

But the network fails before the client receives the response.

Client thinks:

```text
sync failed
```

and keeps the transaction in the queue.

Later:

```text
POST same transaction
```

again.

The server can process it again.

Result:

```text
Sender: -₹500
Recipient: +₹500

again:

Sender: -₹500
Recipient: +₹500
```

The same logical payment may settle twice.

---

# 44. Required Solution: Idempotency

The transaction ID must act as an idempotency key.

The server should first check:

```sql
SELECT id
FROM transactions
WHERE id = ?
```

If it already exists:

```text
DO NOT modify balances again.
Return previous result.
```

Correct flow:

```text
POST transaction ID X
       |
       v
Does X exist?
   /       \
 yes        no
 |           |
return      process
existing
result
```

This is mandatory for reliable distributed synchronization.

---

# 45. Critical Sync Design Problem #2 — Batch Rollback

The sync endpoint processes multiple transactions.

Suppose queue contains:

```text
TX1
TX2
TX3
```

and:

```text
TX1 = valid
TX2 = invalid
TX3 = valid
```

The backend can accumulate results but ultimately:

```text
ROLLBACK
```

if one transaction fails.

Therefore:

```text
TX1
TX2
TX3
```

may all be rolled back.

However, the frontend can interpret successful result entries as successful and remove them from its queue.

That can create:

```text
Client:
TX1 = completed

Server:
TX1 = rolled back
```

This is a serious consistency bug.

---

# 46. Better Sync Strategy

Each transaction should have independent atomicity.

Instead of:

```text
BEGIN
   TX1
   TX2
   TX3
COMMIT/ROLLBACK ALL
```

use:

```text
TX1:
 BEGIN
 process
 COMMIT

TX2:
 BEGIN
 process
 ROLLBACK

TX3:
 BEGIN
 process
 COMMIT
```

or use a transaction-by-transaction service boundary.

Then response can safely be:

```json
[
  { "id": "TX1", "status": "success" },
  { "id": "TX2", "status": "failed" },
  { "id": "TX3", "status": "success" }
]
```

---

# 47. Critical Sync Design Problem #3 — Offline Balance Validation

Suppose:

```text
Actual server balance = ₹1000
```

Offline client thinks:

```text
₹1000
```

User performs:

```text
₹700
```

then:

```text
₹400
```

locally.

Client could temporarily allow:

```text
₹700
₹400
```

even though:

```text
700 + 400 = 1100
```

exceeds the actual balance.

The authoritative server must therefore determine the final outcome.

This creates a fundamental offline-payment tradeoff:

```text
Offline convenience
        vs
Financial certainty
```

---

# 48. Service Worker

`sw.js` attempts to provide PWA-style capabilities.

It uses Workbox concepts:

```text
precache
runtime cache
NetworkFirst
CacheFirst
StaleWhileRevalidate
```

This is conceptually good for offline applications.

---

# 49. Service Worker Caching

The service worker attempts to:

```text
precache static assets
```

and cache API responses using:

```text
NetworkFirst
```

for `/api/`.

However, the application's actual API calls include routes such as:

```text
/auth/*
/account/*
/bank-accounts/*
/transactions
/sync/*
```

and the API base URL is an external domain.

Therefore the actual runtime behavior should be tested carefully rather than assuming every API request is covered by the service-worker route.

---

# 50. Background Sync

The service worker listens for:

```text
sync
```

with:

```text
sync-transactions
```

It then sends:

```text
SYNC_TRANSACTIONS
```

to active clients.

This is more of a **background-sync integration scaffold** than a complete independent background transaction processor.

The actual queue lives in IndexedDB and synchronization logic primarily resides in `ApiService`.

---

# 51. Client API Service

`services/api.js` acts as a frontend service/repository layer.

It encapsulates:

```text
HTTP communication
authentication
network state
health checking
offline synchronization
local persistence coordination
```

This is a good abstraction.

Instead of:

```javascript
axios.post(...)
```

everywhere, components call:

```javascript
apiService.createTransaction(...)
```

---

# 52. API Service Responsibilities

The service contains methods for:

```text
requestOtp()
verifyOtp()
getAccountDetails()
linkBankAccount()
getBankAccounts()
lookupBankAccount()
setPrimaryAccount()
createTransaction()
getTransactions()
syncOfflineTransactions()
healthCheck()
```

Therefore it is effectively the frontend's application gateway.

---

# 53. Backend Endpoint Inventory

## Public

```http
GET /health
POST /auth/request-otp
POST /auth/verify-otp
```

## Authenticated

```http
GET /account/details

POST /bank-accounts/link
GET /bank-accounts
POST /bank-accounts/lookup

POST /transactions
POST /sync/transactions
GET /transactions

POST /api/bank-accounts/set-primary
```

---

# 54. Bank Account Linking

The user supplies:

```text
Bank
Account Number
IFSC
Branch
4-digit PIN
```

The server:

```text
validate fields
     |
     v
validate PIN
     |
     v
bcrypt hash
     |
     v
create UUID
     |
     v
initial balance = ₹1000
     |
     v
INSERT
```

The initial balance is clearly a development/demo behavior.

A real banking system would never arbitrarily create:

```text
₹1000
```

upon linking an external bank account.

---

# 55. Recipient Lookup

The frontend asks for:

```text
account number
IFSC
branch
```

The server queries:

```sql
WHERE account_number = ?
AND ifsc_code = ?
AND branch = ?
```

This is a simple identity-resolution mechanism.

The frontend then displays:

```text
Verified: [bank/name]
```

before payment.

---

# 56. QR Architecture

The user profile generates a QR payload containing recipient information.

Conceptually:

```text
User Profile
    |
    v
Primary bank account
    |
    v
JSON
    |
    v
QRCodeCanvas
```

The scanner:

```text
camera
  |
  v
QR reader
  |
  v
text
  |
  v
JSON.parse()
  |
  v
recipient data
  |
  v
TransactionForm
```

This is a clean UI flow.

---

# 57. QR Security Problem

The QR payload appears to contain sensitive financial/account information.

QR codes are bearer-like data.

Anyone who scans it can potentially obtain:

```text
name
account number
IFSC
bank
```

A production design could use:

```text
opaque payment identifier
```

instead:

```text
QR:
    payment://recipient/[opaque-id]
```

Then the backend resolves the ID.

---

# 58. Dashboard Architecture

`Dashboard.jsx` is the primary authenticated application screen.

It handles:

```text
user identity
balance display
bank accounts
transaction history
payment form
QR scanner
profile drawer
network status
logout
```

It is therefore becoming a relatively large orchestration component.

A future refactor could extract:

```text
BalanceCard
QuickActions
TransactionSection
QrScannerModal
AccountSelector
```

---

# 59. TransactionForm

This component manages payment input.

Flow:

```text
Select sender account
        |
        v
Enter recipient details
        |
        v
Lookup recipient
        |
        v
Enter amount
        |
        v
Enter PIN
        |
        v
Submit
```

Online mode requires recipient verification.

Offline mode can bypass server recipient lookup if QR data is already available.

That is an important offline usability decision.

---

# 60. TransactionList

The transaction list determines:

```text
incoming vs outgoing
```

by checking whether one of the user's bank accounts is the sender.

Conceptually:

```text
transaction.fromBankAccountId
             |
             v
is this one of my accounts?
        /          \
      yes           no
       |             |
   outgoing       incoming
```

Then it displays:

```text
-₹500
```

or:

```text
+₹500
```

---

# 61. Landing Page and Feature Cards

Several components are primarily presentation/demo components:

```text
LandingPage
InstantSyncCard
OfflineModeCard
EncryptedVaultCard
QRPaymentsCard
PayXEcosystem
PayXFeatures
StackedFeatures
SecurityBadge
PhoneMockUp
```

These communicate the product concept visually.

They should be distinguished from actual business logic.

---

# 62. Important Example: QRPaymentsCard

`QRPaymentsCard.jsx` contains a generated grid using:

```javascript
Math.random()
```

This means it is a **visual mock QR**, not necessarily a functional payment QR.

That is an important distinction.

The actual functional QR mechanism is in:

```text
UserProfileDrawer
Dashboard
react-qr-reader
```

---

# 63. Technology Stack

## Frontend

| Technology | Purpose |
|---|---|
| React 19 | UI framework |
| Vite 7 | Build/dev server |
| Tailwind CSS 4 | Styling |
| Axios | HTTP client |
| Framer Motion | Animations |
| Lucide React | Icons |
| qrcode.react | QR generation |
| react-qr-code | QR functionality |
| react-qr-reader | QR scanning |
| html2canvas | QR/image export |
| CryptoJS | local encryption |
| Zod | intended validation |
| lodash.throttle | throttling utility |

---

# 64. Backend

| Technology | Purpose |
|---|---|
| Node.js | runtime |
| Express 5 | REST API |
| SQLite3 | database |
| JWT | authentication |
| bcryptjs | PIN hashing |
| Twilio | SMS OTP |
| Helmet | security headers |
| CORS | cross-origin API |
| express-rate-limit | abuse prevention |
| UUID | identifiers |
| dotenv | configuration |

---

# 65. Suspicious / Unused Backend Dependencies

The server package includes:

```text
bcrypt
connect-session-sequelize
crypto-js
express-session
mysql2
nodemailer
passport
passport-local
plaid
redis
```

but the inspected server implementation does not meaningfully use these.

This strongly suggests the project evolved from multiple architectural experiments.

For example:

```text
Redis
MySQL
Plaid
Passport
Sessions
Nodemailer
```

suggest capabilities that are not actually part of the current implementation.

A production cleanup should remove unused dependencies.

---

# 66. Why Express?

Express is appropriate for this project because:

```text
small API
simple REST endpoints
JavaScript ecosystem
low conceptual overhead
```

Alternatives:

### NestJS

Better for:

```text
large backend
dependency injection
modular architecture
enterprise conventions
```

### Fastify

Better when:

```text
high-performance Node API
schema-driven validation
```

### Spring Boot

Better for:

```text
enterprise banking
strong typing
large organization
mature Java ecosystem
```

For a prototype, Express is reasonable.

---

# 67. Why SQLite?

SQLite is attractive because:

```text
zero external DB server
single file
simple setup
transaction support
excellent for prototypes
```

This is excellent for:

```text
local development
demonstrations
small workloads
```

It is not an appropriate authoritative database for a serious high-volume banking platform.

---

# 68. Production Database Direction

A better production architecture would use:

```text
PostgreSQL
```

or another enterprise-grade relational database.

Potential architecture:

```text
Express / NestJS
       |
       v
Connection Pool
       |
       v
PostgreSQL
```

Benefits:

- concurrency
- replication
- indexes
- operational tooling
- backups
- stronger production deployment patterns
- better horizontal scalability

---

# 69. Why IndexedDB?

IndexedDB is a strong browser technology for this use case.

Unlike:

```text
localStorage
```

IndexedDB provides:

```text
structured objects
larger storage
transactions
indexes
asynchronous APIs
```

Therefore:

```text
offline transaction queue
```

is much better suited to IndexedDB than localStorage.

---

# 70. Why a Service Worker?

A service worker allows code to execute independently from the page lifecycle.

This enables concepts such as:

```text
offline asset loading
background synchronization
push notifications
cache strategies
```

That makes it appropriate for an offline-first PWA.

---

# 71. Current Service Worker Limitation

The service worker expects Workbox-generated:

```text
self.__WB_MANIFEST
```

but the Vite configuration shown in the repository does not configure Workbox generation.

Therefore the project should be tested carefully because:

```text
self.__WB_MANIFEST
```

normally needs a build-time Workbox integration.

Simply placing a service worker file in `src` does not automatically generate that manifest.

This is an important setup/deployment issue.

---

# 72. Setup Guide

## Step 1 — Extract

```bash
unzip SecureBank-main.zip
cd SecureBank-main/banking-app
```

---

# 73. Backend Setup

```bash
cd server
npm install
```

Create:

```text
.env
```

with at minimum:

```env
JWT_SECRET=[STRONG_RANDOM_JWT_SECRET]

TWILIO_ACCOUNT_SID=[TWILIO_ACCOUNT_SID]
TWILIO_AUTH_TOKEN=[TWILIO_AUTH_TOKEN]
TWILIO_PHONE_NUMBER=[TWILIO_PHONE_NUMBER]
```

Do not commit real secrets.

---

# 74. Start Backend

Development:

```bash
npm run dev
```

or:

```bash
npm start
```

Expected server:

```text
http://localhost:8000
```

Health check:

```bash
curl http://localhost:8000/health
```

Expected concept:

```json
{
  "status": "healthy"
}
```

---

# 75. Database Initialization

The server automatically creates:

```text
users
bank_accounts
transactions
```

if they do not exist.

The repository also contains:

```text
migrate.js
```

which adds:

```text
users.name
bank_accounts.is_primary
```

to older database versions.

Run from `server`:

```bash
node migrate.js
```

if working with the older DB schema.

---

# 76. Frontend Setup

Open another terminal:

```bash
cd banking-app/client
npm install
```

Then:

```bash
npm run dev
```

Vite will provide the development URL, usually something like:

```text
http://localhost:5173
```

---

# 77. Important Configuration Problem

`api.js` currently contains a hardcoded API URL:

```text
https://payx-kaqu.onrender.com
```

Therefore local frontend development may still communicate with the deployed backend instead of:

```text
http://localhost:8000
```

A production-quality configuration should use:

```env
VITE_API_BASE_URL=[API_BASE_URL]
```

and:

```javascript
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL;
```

---

# 78. Recommended Development Configuration

Frontend:

```env
VITE_API_BASE_URL=http://localhost:8000
```

Production:

```env
VITE_API_BASE_URL=[PRODUCTION_API_URL]
```

This avoids changing source code between environments.

---

# 79. End-to-End Local Test

After both processes start:

```text
Terminal 1:
server

Terminal 2:
client
```

Then:

```text
Browser
  |
  v
Landing page
  |
  v
Sign up
  |
  v
Phone number
  |
  v
OTP
  |
  v
JWT
  |
  v
Dashboard
  |
  v
Link account
  |
  v
Create payment
```

---

# 80. Testing Offline Mode

After login and account setup:

1. Create/verify at least two accounts in the database.
2. Open the dashboard.
3. Enable browser DevTools.
4. Switch network to offline.
5. Create a transaction.
6. Observe:
   - local transaction appears
   - status becomes syncing/queued
   - local sender balance changes optimistically
7. Restore network.
8. Observe synchronization.
9. Verify server-side transaction and balances.

---

# 81. How the Offline Queue Works

Detailed sequence:

```text
User clicks Pay
       |
       v
TransactionForm
       |
       v
Dashboard.handleTransactionSubmit()
       |
       v
apiService.createTransaction()
       |
       +--------------------+
       |                    |
       v                    v
saveTransaction()     addToSyncQueue()
       |                    |
       +---------+----------+
                 |
                 v
       optimistic balance
                 |
                 v
           return instantly
```

If online:

```text
syncOfflineTransactions()
```

is triggered.

If offline:

```text
queue remains persistent
```

until connectivity returns.

---

# 82. Synchronization Sequence

```text
Browser detects online
          |
          v
ApiService.isOnline = true
          |
          v
healthCheck()
          |
          v
server healthy
          |
          v
syncOfflineTransactions()
          |
          v
IndexedDB syncQueue
          |
          v
POST /sync/transactions
          |
          v
Express
          |
          v
validate each transaction
          |
          v
debit + credit + ledger
          |
          v
response
          |
          v
IndexedDB status updated
          |
          v
successful queue items deleted
          |
          v
transactionsSynced event
          |
          v
Dashboard reloads
```

---

# 83. Distributed Systems Concepts You Should Learn From This Project

This project is actually an excellent educational vehicle for several senior-level concepts.

## 83.1 Eventual consistency

Client:

```text
state A
```

Server:

```text
state B
```

temporarily.

Eventually:

```text
client -> server
```

and state converges.

---

## 83.2 Optimistic updates

Client assumes:

```text
operation will succeed
```

before authoritative confirmation.

---

## 83.3 Durable queues

IndexedDB provides persistent local work storage.

---

## 83.4 Retry semantics

A failed synchronization can be retried.

But retry requires:

```text
idempotency
```

to be safe.

---

# 84. Idempotency — Senior Engineer Concept

An operation is idempotent when repeating it produces the same final result.

For example:

```text
PUT user/profile/123
```

can often be safely repeated.

A financial transfer:

```text
POST transfer
```

is **not naturally idempotent**.

Therefore the client must provide:

```text
Idempotency-Key: [TRANSACTION_ID]
```

or equivalent.

This project already has a transaction ID.

It should exploit that ID for idempotency.

---

# 85. Concurrency Problems

Imagine two requests:

```text
Request A: pay ₹800
Request B: pay ₹800
```

Current balance:

```text
₹1000
```

If both independently read:

```text
balance = 1000
```

both could pass:

```text
balance >= 800
```

and then debit.

A robust implementation needs atomic conditional updates.

For example conceptually:

```sql
UPDATE bank_accounts
SET balance_paise = balance_paise - ?
WHERE id = ?
AND balance_paise >= ?;
```

Then inspect affected rows.

This is safer than:

```text
SELECT balance
check in JavaScript
UPDATE balance
```

because the check and mutation are combined.

---

# 86. SQLite Concurrency Bottleneck

SQLite is optimized around a file-based architecture.

The biggest concern is concurrent write workload.

Banking transfers are writes:

```text
debit
credit
insert transaction
```

High transaction volume produces contention.

Therefore:

```text
1000 concurrent transfers
```

is not what this architecture is designed for.

---

# 87. SQLite Horizontal Scaling Problem

Suppose we deploy:

```text
Load Balancer
   |
 +---+---+
 |       |
 v       v
API A  API B
 |       |
 +---+---+
     |
   SQLite
```

A local SQLite file does not naturally provide a shared authoritative database across arbitrary server instances.

You therefore need a centralized production database:

```text
API A
  \
   +--> PostgreSQL
  /
API B
```

---

# 88. OTP Horizontal Scaling Problem

Same problem exists with:

```javascript
otpStore = new Map()
```

Each process has separate memory.

Production:

```text
API A --> Redis
API B --> Redis
API C --> Redis
```

All servers share OTP state.

---

# 89. Scalability Bottleneck Summary

| Area | Current | Problem |
|---|---|---|
| Database | SQLite | write/concurrency limitations |
| OTP | memory Map | not distributed |
| Auth | localStorage JWT | XSS exposure |
| Sync | batch rollback | consistency risk |
| Retry | no idempotency | duplicate settlement |
| Money | REAL | precision |
| Validation | ad hoc | inconsistent input validation |
| Backend | one large file | maintainability |
| API URL | hardcoded | environment coupling |
| CORS | broad | excessive access |
| Queue | browser-local | device/browser dependent |
| Observability | console logs | weak production telemetry |

---

# 90. Security Analysis

## Good

The project includes:

```text
Helmet
bcrypt
JWT
rate limiting
input checks
server-side authorization
database transactions
```

These are good foundations.

---

# 91. Security Problems

### 1. Hardcoded local encryption key

Must become:

```text
[SECURE_KEY_STRATEGY]
```

although client-side secret management has fundamental limitations.

### 2. OTP logged

Remove.

### 3. JWT localStorage

Consider secure cookies.

### 4. Broad CORS

Current:

```javascript
cors()
```

should be restricted to:

```text
[FRONTEND_ORIGIN]
```

### 5. No comprehensive schema validation

Although Zod exists in frontend dependencies, the server manually checks fields.

Use:

```text
Zod
Joi
Yup
Ajv
class-validator
```

or another centralized schema validation system.

---

# 92. Serious Bug: `set-primary` Endpoint

This endpoint is inconsistent with the rest of the application.

It uses:

```javascript
db.query(...)
```

and PostgreSQL syntax:

```sql
$1
$2
```

But the actual database is:

```text
sqlite3
```

and other code uses:

```javascript
db.get(...)
db.all(...)
db.run(...)
```

The endpoint also refers to:

```text
account.rows
account.rows[0].pin
```

whereas the actual schema stores:

```text
pin_hash
```

Therefore this endpoint is effectively written for a different database/API abstraction and is incompatible with the current SQLite implementation.

This is a major codebase inconsistency.

---

# 93. Correct Design for Set Primary

For SQLite:

```text
SELECT account
WHERE id = ?
AND user_id = ?
```

then:

```text
bcrypt.compare(pin, pin_hash)
```

then:

```text
BEGIN
UPDATE all accounts -> is_primary = 0
UPDATE selected account -> is_primary = 1
COMMIT
```

This should use the same SQLite API already used everywhere else.

---

# 94. Migration Strategy

`migrate.js` demonstrates a primitive schema migration strategy.

It performs:

```sql
ALTER TABLE users
ADD COLUMN name TEXT
```

and:

```sql
ALTER TABLE bank_accounts
ADD COLUMN is_primary BOOLEAN DEFAULT 0
```

This is useful for a prototype.

For production, use:

```text
Prisma Migrate
Knex migrations
Sequelize migrations
Flyway
Liquibase
```

or equivalent.

---

# 95. Code Quality Assessment

## Strengths

### Clear technology separation

```text
React
Express
SQLite
IndexedDB
```

are easy to identify.

### Service abstraction

`api.js` is a useful boundary.

### Local database abstraction

`database.js` hides IndexedDB's callback/event complexity.

### Authentication context

Good use of React Context.

### Network context

Makes offline state accessible throughout UI.

### Database transactions

The transfer logic correctly attempts atomic operations.

---

# 96. Code Smells

## Giant backend file

```text
823 lines
```

in `server/index.js`.

## Giant frontend components

Dashboard contains many responsibilities.

## Unused dependencies

Suggest architectural leftovers.

## Duplicate comments

Several areas contain comments such as:

```text
UPDATE:
NEW METHOD ADDED:
```

indicating incremental patching rather than clean architectural refactoring.

## Inconsistent API style

The primary-account endpoint uses a completely different DB abstraction.

---

# 97. Data Ownership

One of the most important architectural questions is:

> Which system is authoritative?

The correct answer should be:

```text
Server database
```

IndexedDB is:

```text
cache
offline queue
temporary state
```

It must never become the final authority for financial balances.

---

# 98. Recommended State Model

Instead of only:

```text
syncing
completed
failed
```

a production system could use:

```text
DRAFT
QUEUED
SUBMITTED
PROCESSING
COMPLETED
FAILED
REJECTED
REQUIRES_REVIEW
```

This allows better reconciliation.

---

# 99. Recommended Transaction State Machine

```text
           +-------+
           | DRAFT |
           +---+---+
               |
               v
           +-------+
           | QUEUED|
           +---+---+
               |
          network available
               |
               v
         +-----------+
         | SUBMITTED |
         +-----+-----+
               |
               v
        +--------------+
        |  PROCESSING  |
        +------+-------+
               |
         +-----+------+
         |            |
         v            v
   +----------+  +--------+
   | COMPLETED|  | FAILED |
   +----------+  +--------+
```

---

# 100. Production Architecture

A much stronger future architecture would look like:

```text
                         Internet
                            |
                            v
                    CDN / WAF / TLS
                            |
                            v
                       Load Balancer
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
          API #1         API #2         API #3
             |              |              |
             +--------------+--------------+
                            |
                    Application Services
                            |
       +--------------------+--------------------+
       |                    |                    |
       v                    v                    v
 Authentication       Transaction Service    Account Service
       |                    |                    |
       +--------------------+--------------------+
                            |
                            v
                       PostgreSQL
                            |
              +-------------+-------------+
              |                           |
              v                           v
            Redis                    Event Broker
              |                           |
              v                           v
       OTP / Cache / Locks        Async Processing
```

Client:

```text
React
 |
 +--> IndexedDB
 |
 +--> Service Worker
 |
 +--> Sync Queue
 |
 +--> API
```

---

# 101. Better Offline Architecture

A mature design could use:

```text
IndexedDB
    |
    +--> transaction intents
    +--> local projections
    +--> sync metadata
    +--> retry count
    +--> server version
```

Each queued command contains:

```json
{
  "transactionId": "[UUID]",
  "createdAt": "[TIMESTAMP]",
  "operation": "TRANSFER",
  "payload": "[ENCRYPTED/POLICY-COMPLIANT PAYLOAD]",
  "attemptCount": 0,
  "nextRetryAt": "[TIMESTAMP]",
  "state": "QUEUED"
}
```

---

# 102. Retry Backoff

Current implementation does not fully implement robust retry policy.

A production queue should use exponential backoff:

```text
1 sec
2 sec
4 sec
8 sec
16 sec
...
```

with jitter.

Conceptually:

```text
retryDelay =
    min(
        MAX_DELAY,
        BASE_DELAY * 2^attempt
    )
    + randomJitter
```

This prevents a large population of clients from retrying simultaneously.

---

# 103. Dead-Letter Queue

After repeated failures:

```text
QUEUED
  |
  v
RETRY
  |
  v
RETRY
  |
  v
RETRY
  |
  v
DEAD LETTER
```

The user should then see:

```text
Payment could not be synchronized.
Action required.
```

rather than silently retrying forever.

---

# 104. Reconciliation

A production offline system needs a reconciliation mechanism.

For example:

```text
Client local balance
        |
        v
Server authoritative balance
        |
        v
difference detected
        |
        v
reconciliation event
```

The client then updates its local projection.

---

# 105. Stronger Financial Model

Instead of treating balance as simply mutable state:

```text
balance = 1000
```

a banking system can maintain a ledger:

```text
Account A
-----------------
Opening balance 1000
Transfer       -500
Transfer       -100
-----------------
Current         400
```

The ledger becomes the source of truth.

This improves:

- auditing
- reconciliation
- dispute resolution
- transaction history
- financial traceability

---

# 106. Double-Entry Accounting

A truly banking-oriented design should consider double-entry accounting.

For:

```text
A -> B ₹500
```

record:

```text
A: -500
B: +500
```

as two ledger entries within one atomic transaction.

Conceptually:

```text
Journal Entry
    |
    +--> Debit  A  ₹500
    |
    +--> Credit B  ₹500
```

The invariant is:

```text
sum(debits) = sum(credits)
```

This is a fundamental financial-system concept.

---

# 107. Why Double Entry Is Better

If an accounting system produces:

```text
A -₹500
B +₹500
```

the total is:

```text
0
```

If only one side is recorded:

```text
A -₹500
B +₹0
```

the accounting system becomes unbalanced.

This makes errors easier to detect.

---

# 108. Recommended Backend Refactoring

Break `index.js` into:

```text
src/
│
├── app.js
├── server.js
│
├── config/
│   ├── env.js
│   └── database.js
│
├── middleware/
│   ├── auth.js
│   ├── rateLimit.js
│   └── errorHandler.js
│
├── routes/
│   ├── auth.routes.js
│   ├── accounts.routes.js
│   ├── bankAccounts.routes.js
│   └── transactions.routes.js
│
├── controllers/
│   ├── auth.controller.js
│   ├── bank.controller.js
│   └── transaction.controller.js
│
├── services/
│   ├── otp.service.js
│   ├── auth.service.js
│   ├── transaction.service.js
│   └── sync.service.js
│
├── repositories/
│   ├── user.repository.js
│   ├── account.repository.js
│   └── transaction.repository.js
│
└── validators/
    ├── auth.schema.js
    └── transaction.schema.js
```

---

# 109. Why Service + Repository Layers?

Controller:

```text
HTTP concerns
```

Service:

```text
business logic
```

Repository:

```text
database operations
```

For example:

```text
POST /transactions
        |
        v
TransactionController
        |
        v
TransactionService
        |
        v
TransactionRepository
        |
        v
PostgreSQL
```

This prevents SQL and business logic from becoming mixed together.

---

# 110. Frontend Refactoring

Current:

```text
Dashboard.jsx
```

does too much.

Refactor:

```text
Dashboard
├── Header
├── BalanceCard
├── QuickActions
├── AccountCard
├── TransactionSection
├── QRScanner
└── Modals
```

Application logic can be moved into hooks:

```text
useAuth()
useNetwork()
useTransactions()
useBankAccounts()
useOfflineQueue()
```

---

# 111. Better Offline Hook

For example:

```text
useOfflineQueue()
```

could expose:

```javascript
{
    queue,
    pendingCount,
    enqueue,
    sync,
    retry,
    remove
}
```

Then UI does not need to understand IndexedDB internals.

---

# 112. Observability Roadmap

Current logging is mostly:

```javascript
console.log()
console.error()
```

Production should have:

```text
structured logs
metrics
distributed tracing
error tracking
audit logs
```

For example:

```text
transaction.created
transaction.sync.started
transaction.sync.success
transaction.sync.failed
transaction.duplicate
transaction.reconciled
```

---

# 113. Audit Logging

For financial applications, audit events are extremely important.

Example:

```text
USER_LOGIN
BANK_ACCOUNT_LINKED
PIN_VERIFICATION_FAILED
TRANSFER_INITIATED
TRANSFER_COMPLETED
TRANSFER_FAILED
PRIMARY_ACCOUNT_CHANGED
OTP_REQUESTED
OTP_VERIFIED
```

Each event should contain:

```text
event ID
user ID
timestamp
request ID
device/session information
result
```

without logging secrets.

---

# 114. API Improvements

Every endpoint should have:

```text
request schema
response schema
authentication requirements
authorization requirements
error contract
```

Example:

```http
POST /transactions
```

Response:

```json
{
  "transactionId": "[UUID]",
  "status": "COMPLETED",
  "amount": 50000,
  "currency": "INR"
}
```

---

# 115. Currency Modeling

Instead of:

```json
{
  "amount": 500.50
}
```

prefer:

```json
{
  "amountMinor": 50050,
  "currency": "INR"
}
```

This avoids floating-point ambiguity.

---

# 116. Database Indexing

Queries frequently use:

```text
user_id
account_number
ifsc_code
branch
from_bank_account_id
to_bank_account_id
created_at
```

Production indexes should be evaluated.

For example:

```sql
CREATE INDEX idx_bank_accounts_user
ON bank_accounts(user_id);

CREATE INDEX idx_transactions_from
ON transactions(from_bank_account_id);

CREATE INDEX idx_transactions_to
ON transactions(to_bank_account_id);
```

Composite indexes should be designed according to actual query patterns.

---

# 117. Authorization

Authentication asks:

> Who are you?

Authorization asks:

> Are you allowed to do this?

The project performs an important authorization check:

```text
senderAccount.user_id === req.user.id
```

This prevents a user from simply submitting another user's account ID.

That is a very important security property.

---

# 118. Mass Assignment Risk

A production API should avoid blindly trusting request bodies.

For example:

```javascript
req.body
```

should be explicitly transformed into an allowed DTO:

```text
fromBankAccountId
toAccountNumber
toIfscCode
toBranch
amount
type
description
```

rather than allowing arbitrary fields to influence business logic.

---

# 119. Validation Roadmap

Use centralized validation.

For example:

```text
Phone:
E.164

OTP:
6 digits

PIN:
4 digits

Amount:
positive integer minor units

IFSC:
proper format

Account number:
bank-specific constraints
```

Validation should occur:

```text
frontend
+
backend
```

but the backend is authoritative.

---

# 120. Error Handling

Current errors are mostly:

```json
{
  "error": "..."
}
```

A better production contract could be:

```json
{
  "error": {
    "code": "INSUFFICIENT_FUNDS",
    "message": "Insufficient funds",
    "requestId": "[REQUEST_ID]"
  }
}
```

This lets the frontend distinguish:

```text
retryable error
non-retryable error
authentication error
validation error
business error
```

---

# 121. Retry Classification

This matters enormously for offline synchronization.

### Retryable

```text
network timeout
502
503
temporary database failure
```

### Usually not retryable

```text
invalid PIN
recipient doesn't exist
self-transfer
insufficient funds
invalid amount
```

A queue should automatically retry only retryable failures.

---

# 122. Current Sync Queue Improvement

The queue contains:

```text
attempts
priority
```

but synchronization does not fully exploit them.

A stronger implementation should:

```text
sort by priority
increment attempts
record lastAttemptAt
record error
calculate nextRetryAt
stop retrying permanent failures
```

---

# 123. Offline Security Model

The key security question is:

> What happens if the device is stolen while offline?

Current architecture stores:

```text
cached user data
transaction data
encrypted PIN
JWT
```

on the browser.

Even though some data is encrypted, the overall browser is still a trusted environment.

Production banking apps usually require stronger device-bound security.

Possible improvements:

```text
WebAuthn/passkeys
device-bound credentials
hardware-backed keys
transaction signing
biometric authorization
short-lived sessions
```

---

# 124. Transaction Signing Architecture

A more advanced offline design could create a key pair:

```text
Private Key
   |
   +--> secure device storage

Public Key
   |
   +--> server
```

Offline transaction:

```text
transaction payload
       |
       v
hash
       |
       v
private-key signature
       |
       v
IndexedDB
```

When online:

```text
server
 |
 v
verify signature
 |
 v
validate nonce
 |
 v
execute transaction
```

This is conceptually stronger than storing a reusable PIN.

---

# 125. Nonce / Sequence Number

Offline synchronization also needs replay protection.

Each transaction could contain:

```text
transactionId
deviceId
sequenceNumber
timestamp
signature
```

Server maintains:

```text
lastAcceptedSequence
```

This helps prevent replay.

---

# 126. Device-Level Queues

The current queue is browser-global.

A better model would associate records with:

```text
userId
deviceId
```

Example:

```json
{
  "userId": "[USER_ID]",
  "deviceId": "[DEVICE_ID]",
  "transactionId": "[UUID]"
}
```

This improves multi-account/device management.

---

# 127. Multi-Device Consistency

Suppose the same user logs in on:

```text
Phone A
Laptop B
```

Phone A queues:

```text
₹500 payment
```

Laptop B still displays:

```text
old balance
```

until synchronization.

Therefore production architecture needs:

```text
server-authoritative state
+
versioning
+
push updates
+
reconciliation
```

---

# 128. Potential Use of WebSockets

For live transaction updates:

```text
Server
 |
 v
WebSocket
 |
 +--> Phone
 +--> Laptop
 +--> Tablet
```

Then:

```text
Payment completed
      |
      v
Push event
      |
      v
all clients update
```

This is optional, not necessary for the prototype.

---

# 129. Production Infrastructure

Recommended:

```text
Cloudflare / AWS / GCP / Azure
        |
        v
WAF
        |
        v
Load Balancer
        |
        v
Containerized API
        |
        +--> PostgreSQL
        |
        +--> Redis
        |
        +--> Message Broker
        |
        +--> Observability
```

Docker/Kubernetes would become relevant only when operational scale justifies them.

---

# 130. Testing Strategy

The repository currently has no meaningful automated test suite.

That is a major gap.

Recommended layers:

## Unit

Test:

```text
OTP generation
validation
transaction calculations
state transitions
queue logic
```

## Integration

Test:

```text
API + database
```

## End-to-end

Test:

```text
login
link account
payment
offline payment
reconnection
sync
```

---

# 131. Most Important E2E Scenario

The most valuable test would be:

```text
Given:
    account A = ₹1000
    account B = ₹500

When:
    client goes offline
    user queues ₹300 transfer

Then:
    local queue contains transaction

When:
    client reconnects

Then:
    server transfers ₹300 exactly once

And:
    A = ₹700
    B = ₹800

And:
    retrying same request does NOT transfer again
```

This tests the core architecture.

---

# 132. Failure Injection Testing

A senior-level test suite should intentionally simulate:

```text
network disconnect
network reconnect
server timeout
response lost after commit
database failure
duplicate request
invalid PIN
insufficient balance
recipient deleted
two simultaneous transfers
browser restart
service worker restart
```

This is where distributed systems reveal their real bugs.

---

# 133. Current Project Maturity

I would classify the project as:

```text
Prototype / Educational MVP
```

rather than:

```text
Production banking system
```

That is not a criticism.

The architecture demonstrates several real engineering concepts, especially:

```text
offline-first UX
local persistence
eventual synchronization
JWT auth
transactional database operations
browser event architecture
```

But the system lacks several properties required for real financial infrastructure.

---

# 134. Architecture Scorecard

| Category | Assessment |
|---|---|
| UI architecture | Good prototype |
| API architecture | Good prototype |
| Authentication | Basic/functional |
| Authorization | Basic but important checks exist |
| Offline UX | Strong concept |
| Offline consistency | Needs major improvement |
| Database modeling | Good learning model |
| Financial correctness | Not production-grade |
| Security | Basic foundation |
| Scalability | Low |
| Observability | Low |
| Testing | Very low |
| Maintainability | Medium-low |
| Production readiness | Low |

---

# 135. Top 10 Improvements

If you were asked in an interview:

> "What would you improve?"

answer in this order:

### 1. Add idempotency

Prevent duplicate transfers.

### 2. Replace SQLite

Move to PostgreSQL.

### 3. Replace floating-point money

Use integer minor units.

### 4. Fix sync atomicity

Process each transaction independently.

### 5. Add reconciliation

Server remains authoritative.

### 6. Remove PIN from offline queue

Use stronger transaction authorization.

### 7. Move OTP state to Redis

Enable horizontal scaling.

### 8. Add centralized validation

Use schemas.

### 9. Split backend into services/modules

Improve maintainability.

### 10. Add automated tests

Especially offline/retry/failure scenarios.

---

# 136. Suggested Development Roadmap

## Phase 1 — Stabilize

```text
Fix set-primary endpoint
Fix API base URL
Fix service-worker build integration
Remove unused dependencies
Add environment configuration
```

---

## Phase 2 — Correctness

```text
Add idempotency
Use integer money
Fix batch rollback
Add transaction state machine
Add reconciliation
```

---

## Phase 3 — Security

```text
Remove OTP logs
Improve OTP generation
Secure CORS
Improve session handling
Remove reusable PIN from offline storage
Add audit logging
```

---

## Phase 4 — Scalability

```text
SQLite -> PostgreSQL
Map -> Redis
single API -> horizontally scalable API
```

---

## Phase 5 — Reliability

```text
retry backoff
dead-letter queue
request IDs
metrics
tracing
failure injection tests
```

---

## Phase 6 — Advanced Banking Architecture

```text
double-entry ledger
transaction signing
device identity
reconciliation
fraud detection
risk engine
```

---

# 137. How to Explain This Project in an Interview

A strong answer would be:

> "SecureBank is an offline-first payment prototype built using React, Express, SQLite and IndexedDB. The key architectural challenge is maintaining a good user experience when the browser loses connectivity. Instead of requiring the backend for every transaction, the client persists transaction intents in IndexedDB and places them in a durable synchronization queue. When connectivity returns, the client sends the queued transactions to the Express backend, which validates ownership, verifies the sender PIN, validates the recipient and performs the debit, credit and ledger insertion inside a database transaction."

Then immediately demonstrate senior-level awareness:

> "However, I would not consider the current implementation production-ready for financial transactions. The most important missing property is idempotency. If the server commits a transaction but the response is lost, the client may retry and potentially apply the transfer twice. I would use the transaction UUID as an idempotency key and make settlement transactionally idempotent. I would also replace SQLite with PostgreSQL, use integer minor currency units instead of REAL, move OTP state into Redis, and implement proper reconciliation between the local client projection and the authoritative server ledger."

That answer demonstrates much more than simply saying:

> "It is a React banking application."

---

# 138. Mental Model for the Entire Codebase

The easiest way to remember the architecture is:

```text
                    USER
                     |
                     v
                React UI
                     |
          +----------+----------+
          |                     |
          v                     v
     AuthContext          NetworkContext
          |                     |
          v                     v
      ApiService <-------- Network Events
          |
     +----+-------------------------+
     |                              |
     v                              v
 HTTP API                     LocalDatabase
     |                              |
     v                       +------+------+
 Express                     |             |
     |                   IndexedDB      SyncQueue
     |
     v
 SQLite
     |
 +---+---+
 |       |
Users  Accounts
          |
          v
     Transactions
```

---

# 139. The Five Most Important Files

If learning the project from scratch, study these first:

## 1. `server/index.js`

Understand:

```text
API
authentication
database
transactions
sync
```

This is the backend brain.

## 2. `client/src/services/api.js`

Understand:

```text
frontend ↔ backend
offline behavior
sync
health checking
```

This is the communication brain.

## 3. `client/src/services/database.js`

Understand:

```text
IndexedDB
local persistence
encryption
queue
```

This is the offline storage brain.

## 4. `client/src/contexts/NetworkContext.jsx`

Understand:

```text
network state
sync triggering
browser events
```

This is the connectivity brain.

## 5. `client/src/components/Dashboard.jsx`

Understand:

```text
actual user workflow
```

This is the UI orchestration brain.

---

# 140. Recommended Learning Order

Do **not** study the components alphabetically.

Use this sequence:

```text
1. README
       ↓
2. App.jsx
       ↓
3. AuthContext
       ↓
4. api.js
       ↓
5. database.js
       ↓
6. NetworkContext
       ↓
7. server/index.js
       ↓
8. SQLite schema
       ↓
9. Dashboard
       ↓
10. TransactionForm
       ↓
11. TransactionList
       ↓
12. QR flow
       ↓
13. Service Worker
       ↓
14. Failure scenarios
       ↓
15. Scalability
```

---

# 141. What You Should Be Able to Draw on a Whiteboard

For mastery, you should be able to draw this without looking at the code:

```text
             ┌──────────────┐
             │    React     │
             └──────┬───────┘
                    │
            ┌───────┴────────┐
            │                │
            v                v
        ApiService       IndexedDB
            │                │
            │           ┌────┴─────┐
            │           │          │
            │       Transactions Queue
            │                      │
            │                 reconnect
            │                      │
            └──────────┬───────────┘
                       v
                  Express API
                       |
              ┌────────+────────┐
              │                 │
              v                 v
           JWT Auth        Transaction
                                |
                         BEGIN TRANSACTION
                                |
                       +--------+--------+
                       |        |        |
                     Debit    Credit   Ledger
                       |        |        |
                       +--------+--------+
                                |
                              COMMIT
                                |
                                v
                             SQLite
```

If you can explain every arrow, you understand the project.

---

# 142. Final Architectural Verdict

SecureBank is a **good educational implementation of an offline-first financial application**, particularly because it exposes several difficult concepts that are often hidden in ordinary CRUD applications.

Its strongest architectural idea is:

```text
Local durable transaction queue
+
optimistic UI
+
connectivity detection
+
server-side synchronization
```

The most important engineering lesson is also its biggest weakness:

> **Offline systems are not primarily a UI problem; they are a consistency, retry, idempotency, authorization, and reconciliation problem.**

The current project demonstrates the first half very well:

```text
offline persistence
queueing
reconnection
synchronization
```

but the second half needs significant engineering:

```text
idempotency
replay protection
conflict resolution
financial correctness
reconciliation
distributed coordination
```

For an interview, the strongest way to present this project is **not** to claim that it is a production banking architecture.

Instead, present it as:

> **An offline-first payment prototype that demonstrates client-side durable transaction queuing and eventual synchronization, while identifying the additional consistency, security, and scalability mechanisms required to evolve it into a production-grade financial system.**

That framing demonstrates architectural maturity.

---

# 143. Final Cheat Sheet

```text
PROJECT
SecureBank / PayX-style offline banking prototype

FRONTEND
React 19
Vite
Tailwind
Axios
IndexedDB
Service Worker

BACKEND
Node
Express
JWT
bcryptjs
Twilio
Helmet
Rate limiting

DATABASE
SQLite

AUTH
Phone + OTP
JWT

LOCAL STORAGE
IndexedDB
localStorage for JWT

OFFLINE MODEL
Transaction intent
      ↓
IndexedDB
      ↓
Sync Queue
      ↓
Network restored
      ↓
POST /sync/transactions

ONLINE TRANSFER
Authenticate
      ↓
Verify account ownership
      ↓
Verify PIN
      ↓
Check funds
      ↓
Find recipient
      ↓
Debit
      ↓
Credit
      ↓
Insert transaction
      ↓
Commit

MAIN RISKS
No idempotency
SQLite scalability
REAL monetary values
OTP in memory
OTP logging
PIN retained client-side
Hardcoded encryption key
Broad CORS
Weak reconciliation
Batch sync rollback
Large monolithic server file
No meaningful automated tests

BEST FUTURE ARCHITECTURE
React
+
IndexedDB
+
Service Worker
+
PostgreSQL
+
Redis
+
Idempotent transaction service
+
Double-entry ledger
+
Audit logging
+
Reconciliation
+
Observability
```

## Bottom line

**The core engineering lesson of SecureBank is the boundary between local optimism and server authority.**

The browser can provide:

```text
availability
instant feedback
offline persistence
```

but the server must ultimately provide:

```text
authorization
financial correctness
atomic settlement
idempotency
auditability
```

Understanding that boundary is what takes your understanding of this project from **"I know how the code works"** to **"I understand why the architecture exists, where it fails, and how I would redesign it at senior-engineer level."**