# TrackMySpend

A production-grade personal finance tracker built with **React + Vite** (frontend), **Node.js + Express** (backend), **MongoDB** (database), and **Gemini AI** (for natural language expense parsing).

---

## Quick Start

### Backend
```bash
cd backend
cp .env.example .env          # Update MONGO_URI, JWT_SECRET, and GEMINI_API_KEY
npm install
npm run dev                   # Starts backend server using node --env-file
```

### Frontend
```bash
cd frontend
cp .env.example .env          # Adjust VITE_API_URL if needed
npm install
npm run dev
```

Open `http://localhost:5173` in your browser.

---

## Project Structure

```
TrackMySpend/
├── backend/
│   ├── server.js             # Express app (Auth, Expenses, & AI Parse routes)
│   ├── db.js                 # MongoDB connection handler via Mongoose
│   ├── middleware/           # Auth and validation middleware
│   ├── models/               # Mongoose schemas (User, Expense)
│   ├── .env                  # Private environment variables (gitignored)
│   └── .env.example          # Safe configuration template
└── frontend/
    ├── src/
    │   ├── App.jsx           # Root orchestrator — global state only
    │   ├── constants.js      # Categories list + backend URL
    │   ├── api/
    │   │   └── expenseApi.js # API communication layer
    │   └── components/
    │       ├── Header.jsx            # Network-mode selector, theme toggle
    │       ├── ExpenseForm.jsx       # Manual expense entry & AI parsing interface
    │       ├── NetworkConsole.jsx    # Observability log for transactions
    │       ├── SummaryCards.jsx      # Metrics overview
    │       ├── CategoryBreakdown.jsx # Per-category visualization
    │       └── ExpenseLedger.jsx     # Filtering, table listing, and deletion
    ├── .env                  # Local environment vars (gitignored)
    └── .env.example          # Safe template to copy
```

---

## Key Design Decisions

### 1. Integer-Cent Money Storage
Amounts are stored as **integer cents** in MongoDB (e.g. `$15.50` is stored as `1550`). This prevents floating-point rounding errors that accumulate during math operations in JavaScript. The frontend formats the display values back to decimals only at presentation time using `Intl.NumberFormat`.

### 2. User Authentication
A full auth lifecycle is built in. Users register and log in to receive a JWT (JSON Web Token). The frontend stores this token and sends it via `Authorization: Bearer <token>` headers on all API requests, ensuring every user's data is private, secure, and securely query-scoped.

### 3. AI Expense Parsing
Users can type plain English phrases (e.g., *"bought dinner for 450 rupees yesterday"*). The backend uses the `gemini-2.5-flash` model (`@google/genai`) to parse the text, resolve relative dates based on the server's current date, categorize the item (defaulting to *"Others"* if ambiguous), and return structured data to the UI.

### 4. Client-Side Idempotency Keys
Before submitting an expense, the client generates a unique UUID `idempotencyKey`. If a connection drop occurs after the server saves the database record but before the browser receives confirmation, retrying the request with the same key returns the existing record instead of inserting a duplicate.

### 5. Automatic Retry with Backoff
When the API reports temporary errors, the frontend automatically retries failed `POST` requests up to **3 times** with a delay of 1.5 seconds. Every single transaction event (retries, database actions, error states) is transparently rendered in the real-time **Network Console** panel.

### 6. MongoDB Atlas Database
Replaced the flat JSON file database with a real MongoDB integration using **Mongoose**. Schemas define indexes (like compound indexes for sorting history, and sparse indexes for fast idempotency lookups) to ensure database queries perform optimally as volumes scale.

---

## Trade-offs and Architecture

| Area | Solution | Benefit |
|---|---|---|
| **Storage** | MongoDB Atlas (via Mongoose) | Robust, production-grade schema validation and fast indexing. |
| **Auth** | Password Hashing (Salt) + JWT | Secure, stateless authentication matching standard web patterns. |
| **AI Processing** | Google Gen AI SDK (`@google/genai`) | Lightweight, modern library directly calling Google AI Studio. |
| **State Management** | React `useState` & Prop Drilling | Shallow component tree keeps state logic unified and avoids complex store boilerplate. |
| **Resilience** | Network Console & Simulations | Injected errors let developers verify client retry/idempotency behavior instantly. |

---

## API Reference

See the full routes specification, request parameters, and response signatures inside [`backend/README.md`](./backend/README.md).