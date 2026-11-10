# EasyRupee — Personal Finance Management Platform

EasyRupee is a full-stack digital wallet and personal finance web app. Users can register, hold a wallet balance in PKR, send money to other wallets, pay merchant bills, repay loans, and track their expenses, all from one dashboard.

> *"Your finances simplified. Send money, pay bills, and save securely."*

## Features

- **Authentication**: user and merchant registration, login and logout with database-backed sessions (1-day expiry) and cookies.
- **Wallet dashboard**: live balance in PKR (`en-PK` formatting) and the 10 most recent transactions, with incoming and outgoing amounts colour-coded.
- **Transactions**: transfer funds to another wallet through a MySQL stored procedure (`transfer_funds`), and deposit funds into your own wallet.
- **Bills**: browse merchant bills sorted by due date and pay them in full or in part from a wallet. Payments run in a locked SQL transaction that debits the payer, credits the merchant wallet, updates the bill status and records the transaction.
- **Loans**: view your loans (principal, interest rate, dates, status), make repayments that reduce the principal, and mark the loan paid off when it reaches zero.
- **Expenses**: record expenses by category (Rent, Utilities, Salary, Entertainment, Other, Food) and view the history.
- **Profile**: view and update name, email, phone and password.
- **Merchant accounts**: separate merchant registration that creates a linked `Merchants` record by business category (Retail, Food, Entertainment, Utility).

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | [Next.js 15](https://nextjs.org/) (App Router, Turbopack), React 19 |
| Language | TypeScript |
| Styling | Tailwind CSS 4, DaisyUI 5, `tw-animate-css`, shadcn/ui config |
| Icons | lucide-react |
| Database | MySQL / MariaDB via `mysql2` (connection pool) |
| Linting | ESLint 9 with `eslint-config-next` |

## Project Structure

```
├── components/            # Shared UI (Navbar, Sidebar, Card)
├── lib/
│   ├── db.ts              # MySQL connection pool + row types
│   ├── auth.ts            # Session create/destroy and cookie helpers
│   ├── getCurrentUser.ts  # Resolve the current user from the session
│   └── utils.ts           # cn() class-name helper
├── middleware.ts          # Redirects unauthenticated users away from /dashboard
├── src/app/
│   ├── page.tsx           # Landing page
│   ├── login/ register/ registerm/   # Auth pages (registerm = merchant)
│   ├── dashboard/         # Balance + recent transactions
│   ├── transactions/      # Transfers and deposits
│   ├── bills/             # Merchant bills and payment modal
│   ├── loans/             # Loans and repayments
│   ├── expenses/          # Expense tracking
│   ├── profile/           # Account settings
│   └── api/               # Route handlers (see below)
└── public/                # Static assets
```

## API Routes

| Route | Methods | Purpose |
|---|---|---|
| `/api/auth/register` | POST | Register a user |
| `/api/auth/registerm` | POST | Register a merchant (user and merchant record) |
| `/api/auth/login` | POST | Log in and set session cookies |
| `/api/auth/logout` | POST | Destroy the session |
| `/api/wallet` | GET | Current wallet ID from the cookie |
| `/api/transactions` | GET, POST | List the 20 latest transactions, or transfer funds |
| `/api/deposit` | POST | Add funds to the current wallet |
| `/api/bills` | GET, POST | List bills, or pay a bill |
| `/api/loans` | GET, POST | List loans, or record a repayment |
| `/api/expenses` | GET, POST | List (latest 200) or add expenses |
| `/api/profile` | GET, PUT | Read or update the user profile |

## Getting Started

### Prerequisites

- Node.js 18.18+ (20+ recommended)
- A running MySQL or MariaDB server

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment variables

Create a `.env.local` file in the project root:

```env
DB_HOST=localhost
DB_USER=your_db_user
DB_PASS=your_db_password
DB_NAME=your_db_name
DB_PORT=3306

# Optional: base URL used by server-side fetches (e.g. the expenses page)
NEXT_PUBLIC_SITE_URL=http://localhost:3000
```

### 3. Set up the database

The repository does not include a SQL schema file, so you need to create the database yourself. The code expects these tables, plus one stored procedure:

| Table | Used for |
|---|---|
| `Users` | `user_id`, `name`, `email`, `password`, `phone`, `created_at` |
| `Session` | `SessionID`, `UserID`, `Created`, `Expires` |
| `Wallets` | `wallet_id`, `user_id`, `balance` |
| `Transactions` | `transaction_id`, `amount`, `description`, `type_id`, `wallet_id_send`, `wallet_id_recv`, `created_at` |
| `Expenses` | `user_id`, `expense_category_id`, `amount`, `description`, `date_recorded` |
| `Merchants` | `merchant_id`, `merchant_name`, `category_id`, `contact_info` |
| `Merchant_Bills` | `bill_id`, `merchant_id`, `user_id`, `amount`, `due_date`, `status_id` |
| `Bill_Status` | `status_id`, `status_name` |
| `Loans` | `loan_id`, `user_id`, `principal`, `interest_rate`, `start_date`, `end_date`, `status_id` |
| `Loan_Payments` | `loan_id`, `amount_paid`, `payment_date` |

Stored procedure: `transfer_funds(wallet_send, wallet_recv, amount, type_id, description, OUT inserted_id)`.

Every user needs a row in `Wallets` to log in successfully.

### 4. Run the app

```bash
npm run dev      # development server at http://localhost:3000
npm run build    # production build
npm run start    # serve the production build
npm run lint     # lint the codebase
```

## Author

**Abdullah Karim**
