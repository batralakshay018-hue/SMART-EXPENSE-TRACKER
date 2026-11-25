# FinWise AI – Smart Expense Tracker with Gemini 3.0 Chatbot

A full-stack personal finance management SaaS platform built with **React + Vite**, **Tailwind CSS**, **Recharts**, **Express.js**, **Supabase (PostgreSQL + RLS)**, **jsPDF**, and **Google Gemini 3.0 (`@google/genai`)**.

---

## Project Folder Structure

```text
finwise-ai/
├── package.json
├── .gitignore
├── README.md
├── supabase/
│   └── schema.sql                   # PostgreSQL tables, triggers & Row Level Security (RLS)
├── server/
│   ├── package.json
│   ├── .env.example
│   ├── tests/
│   │   └── financeCalculator.test.js
│   └── src/
│       ├── index.js                 # Express API server entry point
│       ├── config/
│       │   ├── constants.js         # Categories, system prompt & demo seed data
│       │   └── env.js               # Environment variable loader & status flags
│       ├── middleware/
│       │   └── auth.js              # Bearer JWT verification (Supabase or Local JWT)
│       ├── routes/
│       │   ├── authRoutes.js        # /api/auth (register, login, demo, me, logout)
│       │   ├── transactionRoutes.js # /api/transactions (CRUD + search/filters)
│       │   ├── budgetRoutes.js      # /api/budgets (monthly/category limits + 80%/100% alerts)
│       │   ├── goalRoutes.js        # /api/goals (savings goals + contributions)
│       │   ├── analyticsRoutes.js   # /api/analytics (summary + monthly report payload)
│       │   ├── chatRoutes.js        # /api/chat (Gemini 3.0 history & message endpoint)
│       │   └── settingsRoutes.js    # /api/settings (profile, currency, export, reset)
│       ├── services/
│       │   ├── geminiService.js     # Official @google/genai Gemini 3.0 multi-turn chat
│       │   └── storageService.js    # Supabase RLS repository + Local scrypt-hashed store
│       └── utils/
│           └── financeCalculator.js # Deterministic KPI, MoM, budget & health calculations
└── client/
    ├── package.json
    ├── .env.example
    ├── index.html
    ├── tailwind.config.js
    ├── postcss.config.js
    ├── vite.config.js
    └── src/
        ├── App.jsx
        ├── main.jsx
        ├── index.css
        ├── context/
        │   ├── AuthContext.jsx
        │   ├── FinanceContext.jsx
        │   └── ThemeContext.jsx
        ├── services/
        │   └── api.js
        ├── utils/
        │   ├── formatters.js
        │   └── pdfReport.js
        ├── components/
        │   ├── Chat/
        │   │   ├── ChatExperience.jsx
        │   │   └── ChatbotWidget.jsx
        │   ├── Common/
        │   │   └── BrandLogo.jsx
        │   ├── Layout/
        │   │   └── DashboardLayout.jsx
        │   └── Transactions/
        │       └── TransactionModal.jsx
        └── pages/
            ├── LandingPage.jsx
            ├── AuthPage.jsx
            ├── OverviewPage.jsx
            ├── TransactionsPage.jsx
            ├── BudgetsPage.jsx
            ├── AnalyticsPage.jsx
            ├── GoalsPage.jsx
            ├── ReportsPage.jsx
            ├── AssistantPage.jsx
            └── SettingsPage.jsx
```

---

## Technologies Used

- **Frontend:** React 18, Vite, React Router v6, Tailwind CSS (Light + Dark Navy Mode), Lucide React, Recharts, jsPDF
- **Backend:** Node.js, Express.js, Helmet, CORS, Express Rate Limit
- **Database & Authentication:** Supabase (`@supabase/supabase-js`) with PostgreSQL Row Level Security (RLS) + secure local `scrypt` hashed persistent fallback when Supabase keys are not yet configured
- **AI Integration:** Google Gemini 3.0 via the official `@google/genai` SDK (`gemini-3-flash-preview` / `gemini-3-pro-preview` / `gemini-3.1-pro-preview`)

---

## Installation Instructions

1. **Install Node.js LTS (v20+ or v22+)** if not already installed:
   ```powershell
   winget install OpenJS.NodeJS.LTS
   ```
   Restart your terminal after installing so `node` and `npm` are in your `PATH`.

2. **Install dependencies** from the project root (`finwise-ai/`):
   ```bash
   npm run install:all
   ```

---

## Environment Variable Configuration

Create `server/.env` by copying `server/.env.example`:

```env
PORT=5000
CLIENT_URL=http://localhost:5173

# Google Gemini 3.0 API Configuration
GEMINI_API_KEY=your_gemini_api_key_here
GEMINI_MODEL=gemini-3-flash-preview

# Supabase Configuration
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_ANON_KEY=your_supabase_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_key_here

# Local JWT Secret
JWT_SECRET=replace_with_a_long_random_secret
```

*(Optional)* Copy `client/.env.example` to `client/.env` if your backend runs on a non-default URL.

---

## Gemini 3.0 API Setup Instructions

1. Visit **Google AI Studio**: https://aistudio.google.com/apikey
2. Sign in with your Google account and click **Create API key**.
3. Copy the key and set `GEMINI_API_KEY=<your-key>` inside `server/.env`.
4. Ensure `GEMINI_MODEL` is set to a valid Gemini 3.0 model identifier such as:
   - `gemini-3-flash-preview` (default — fast, reasoning-enabled Gemini 3 Flash)
   - `gemini-3-pro-preview` (Gemini 3 Pro preview)
   - `gemini-3.1-pro-preview` (deeper reasoning Gemini 3.1 Pro)
5. Never expose `GEMINI_API_KEY` in `client/`. All Gemini calls execute securely on the Express backend.

---

## Supabase Database Setup Instructions

1. Create a free project at https://supabase.com.
2. Open the **SQL Editor** in your Supabase Dashboard.
3. Paste and run the full contents of [`supabase/schema.sql`](supabase/schema.sql).
   This creates `profiles`, `transactions`, `budgets`, `goals`, and `chat_messages` along with strict **Row Level Security (`auth.uid() = user_id`)** policies and automatic profile creation on signup.
4. Go to **Project Settings → API** and copy your **Project URL** and **anon public key** into `server/.env` (`SUPABASE_URL`, `SUPABASE_ANON_KEY`).
5. *(Note: If Supabase keys are not yet configured, the backend automatically uses an isolated local persistent JSON store with `scrypt` password hashing so you can test registration, login, and data persistence immediately.)*

---

## Running the Application Locally

Open two terminals in `finwise-ai/`:

**Terminal 1 — Start the Express Backend (Port 5000):**
```bash
npm run dev:server
```

**Terminal 2 — Start the React + Vite Frontend (Port 5173):**
```bash
npm run dev:client
```

Open **http://localhost:5173** in your browser.

### Running Automated Backend Tests
```bash
npm run test:server
```
