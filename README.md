# Zakat-AI-Assistant
# 🕌 ZakatPro AI — Intelligent Zakat Calculator

> A full-stack, Shariah-compliant Zakat calculator with AI chatbot, multi-currency support, and PDF reports. Built for the Muslim community.

---

## 🌟 Features

- **8-Step Guided Form** — Gold & Silver, Cash & Bank, Investments, Business, Property, Agriculture, Animals, Liabilities
- **AI Chatbot** — Answers Zakat-related questions using Claude AI
- **Multi-Currency** — USD and PKR with admin-managed live exchange rates
- **PDF Reports** — Detailed Shariah-compliant Zakat summary with explanations per asset
- **Nisab Threshold** — Auto-calculated from silver standard (612.36g × $0.93)
- **Special Zakat Rates** — Agricultural (10%/5%/7.5%) and Animal (1 per 40) calculated separately
- **Admin Panel** — Manage exchange rates and formula rates (gold price, agri rates, etc.)
- **Input Validation** — Real-time validation with error messages
- **Shariah Notes** — Per-asset Islamic jurisprudence explanations in PDF

---

## 🏗️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React (Vite), Tailwind CSS |
| Backend | FastAPI (Python) |
| Database | Supabase (PostgreSQL) |
| AI Chatbot | Anthropic Claude API |
| Frontend Hosting | Vercel |
| Backend Hosting | Railway |
| PDF Generation | Client-side HTML → Print |

---

## 📁 Project Structure

```
zakat-app/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Calculator.jsx      # Main 8-step calculator + PDF
│   │   │   ├── AdminPanel.jsx      # Exchange rate & formula management
│   │   │   └── Chatbot.jsx         # AI Zakat assistant
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── .env
│   └── package.json
├── backend/
│   ├── main.py                     # FastAPI app + all endpoints
│   ├── requirements.txt
│   └── .env
└── README.md
```

---

## ⚙️ Environment Variables

### Frontend (`frontend/.env`)
```env
VITE_API_URL=https://your-backend.railway.app
VITE_ANTHROPIC_API_KEY=sk-ant-xxxxx
```

### Backend (`backend/.env`)
```env
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_KEY=your-service-role-key
```

---

## 🚀 Local Development

### Backend
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev
```

---

## 🗄️ Supabase Database Schema

### `rate_history` — Exchange & formula rates
```sql
CREATE TABLE rate_history (
  id SERIAL PRIMARY KEY,
  usd_to_pkr DECIMAL DEFAULT 279.5,
  gold_price_per_gram DECIMAL DEFAULT 89.0,
  silver_price_per_gram DECIMAL DEFAULT 0.93,
  zakat_rate DECIMAL DEFAULT 2.5,
  agri_rain_rate DECIMAL DEFAULT 10.0,
  agri_irrigated_rate DECIMAL DEFAULT 5.0,
  agri_mixed_rate DECIMAL DEFAULT 7.5,
  animal_per INTEGER DEFAULT 40,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### `chat_logs` — AI chatbot history
```sql
CREATE TABLE chat_logs (
  id SERIAL PRIMARY KEY,
  message TEXT,
  response TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### `calculations` — User zakat records
```sql
CREATE TABLE calculations (
  id SERIAL PRIMARY KEY,
  total_assets DECIMAL,
  total_liabilities DECIMAL,
  net_zakatable DECIMAL,
  zakat_amount DECIMAL,
  currency TEXT DEFAULT 'USD',
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 📊 Zakat Calculation Logic

### Standard Zakat
```
Net Zakatable = Total Assets − Total Liabilities
Zakat = Net Zakatable × 2.5%   (if Net ≥ Nisab)
```

### Nisab Threshold
```
Nisab = min(87.48g × gold_price, 612.36g × silver_price)
      = min(87.48 × $89, 612.36 × $0.93)
      = $569.49 USD
```

### Gold Values
```
24k Gold = grams × gold_price
22k Gold = grams × gold_price × 91.67%
18k Gold = grams × gold_price × 75%
```

### Business Net
```
Net = Saleable Stock + Damaged Stock + Credit Receivables
      − Payables to Suppliers − Bad Debts
```

### Partnership Net
```
Net = Capital Balance + Loans to Firm + Accumulated Profit
      − Withdrawals
```

### Agricultural Zakat (calculated separately)
```
Rain-fed crops   → 10%
Irrigated crops  → 5%
Mixed crops      → 7.5%
```

### Animal Zakat (calculated separately)
```
Zakat = floor(count ÷ 40) × (total_value ÷ count)
```

### PKR Conversion
```
Backend always works in USD
User enters PKR → toUSD = value ÷ rate → backend
Backend returns USD → conv = USD × rate → display PKR
Gold/Silver excluded from toUSD (auto-calculated in USD)
```

---

## 🔌 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/calculate` | Calculate Zakat |
| `GET` | `/api/rates` | Get current rates |
| `POST` | `/api/rates` | Update rates (admin) |
| `POST` | `/api/chat` | AI chatbot message |
| `GET` | `/health` | Health check / keep-alive |

### POST `/api/calculate` — Request Body
```json
{
  "gold_24k": 1958.0,
  "gold_22k": 0,
  "gold_18k": 0,
  "silver": 30.69,
  "precious_stones": 0,
  "cash": 123.0,
  "savings": 11.0,
  "current_account": 2322.0,
  "stocks": 1233.0,
  "receivables": 222.0,
  "business_stock": 444.0,
  "damaged_stock": 222.0,
  "business_payables": 0,
  "bad_debts": 0,
  "investment_property": 3333.0,
  "partnership_capital": 5555.0,
  "agri_rain_fed": 4444.0,
  "agri_irrigated": 5555.0,
  "agri_mixed": 1111.0,
  "animals_count": 44,
  "animals_value": 55555.0,
  "friends_loans": 3444.0,
  "utility_arrears": 4444.0,
  "rent_arrears": 2222.0
}
```

### POST `/api/calculate` — Response
```json
{
  "total_assets": 2033.09,
  "total_liabilities": 36.15,
  "net_zakatable": 1996.94,
  "nisab": 569.49,
  "zakat_due": true,
  "zakat_amount": 49.92,
  "standard_zakat": 49.92,
  "agri_zakat": 2.88,
  "animal_zakat": 4.52,
  "total_zakat": 57.32
}
```

---

## 🔒 Security

- Row Level Security (RLS) enabled on all Supabase tables
- Backend uses `service_role` key (bypasses RLS for server operations)
- No user PII stored — calculations are anonymous
- Environment variables for all secrets

---

## 📱 Screenshots

| Step | Screen |
|---|---|
| Step 1 | Gold & Silver input with gram-based calculation |
| Step 4 | Business Net = Stock + Receivables − Payables |
| Step 9 | Results with Zakat breakdown + PDF download |
| Admin | Exchange rate and formula rate management |

---

## 🛠️ Deployment

### Backend → Railway
1. Connect GitHub repo to Railway
2. Set root directory to `backend/`
3. Add environment variables
4. Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`

### Frontend → Vercel
1. Connect GitHub repo to Vercel
2. Set root directory to `frontend/`
3. Add environment variables (`VITE_API_URL`, `VITE_ANTHROPIC_API_KEY`)
4. Build command: `npm run build`



## 🤝 Shariah Compliance

- Zakat calculations follow **Hanafi fiqh** as per Mufti Taqi Usmani's guidelines
- Nisab based on **silver standard** (lower threshold, more inclusive)
- Agricultural and animal Zakat calculated at **separate rates** from standard 2.5%
- Fixed assets (machinery, vehicles, home) are **excluded** from zakatable wealth
- All Shariah notes are verified

---

## 📄 License

Built with ❤️ for the Muslim Community  
