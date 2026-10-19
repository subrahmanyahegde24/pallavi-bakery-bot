# Pallavi Bakery Bot

An AI-powered Telegram ordering assistant for a bakery, with a web-based admin dashboard and online payments.

Customers browse the menu, build a cart, pick a pickup slot and pay online (Razorpay) or at pickup — all inside Telegram, either by tapping buttons or by chatting naturally with **Sara**, a multi-agent assistant built on Google ADK and Gemini. Bakery staff manage orders and the menu from Telegram admin commands or a Next.js dashboard.

![Python](https://img.shields.io/badge/Python-3.12+-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?logo=fastapi&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-16-000000?logo=nextdotjs&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?logo=mongodb&logoColor=white)
![Telegram](https://img.shields.io/badge/Telegram-Bot-26A5E4?logo=telegram&logoColor=white)
![Vercel](https://img.shields.io/badge/Deploy-Vercel-000000?logo=vercel&logoColor=white)

---

## Table of Contents

- [Features](#features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
- [Environment Variables](#environment-variables)
- [Bot Commands](#bot-commands)
- [API Reference](#api-reference)
- [Order Lifecycle](#order-lifecycle)
- [Testing](#testing)
- [Deployment](#deployment)
- [Contributing](#contributing)
- [License](#license)

---

## Features

### For customers (Telegram)
- **Conversational ordering** — ask in plain language ("do you have croissants?", "add two to my cart") and the AI assistant handles it.
- **Button-driven menu** — browse categories (breads, cakes, pastries, cookies, drinks), view item details and choose customizations.
- **Cart management** — add, remove and review items before checkout.
- **Pickup slot selection** at checkout.
- **Flexible payment** — pay online via Razorpay or pay at pickup.
- **Order tracking** — check current order status and past order history.
- **Payment confirmation** — a Telegram message is sent as soon as an online payment succeeds.

### For bakery staff
- **Telegram admin commands** — list, inspect, confirm, mark ready or cancel orders; toggle item availability.
- **Admin dashboard (web)** — overview stats, order management with auto-refresh, full menu CRUD, and a QR code generator linking customers to the bot.
- **API-key protected** admin API and dashboard login.

---

## Architecture

```
┌──────────────┐      webhook       ┌───────────────────────────────┐
│   Telegram   │ ─────────────────► │        FastAPI backend        │
│   (users &   │ ◄───────────────── │                               │
│    admins)   │    bot replies     │  python-telegram-bot handlers │
└──────────────┘                    │            │                  │
                                    │            ▼                  │
                                    │  Google ADK agents (Gemini)   │
                                    │  orchestrator ─┬─ menu        │
                                    │                ├─ order       │
                                    │                ├─ tracking    │
                                    │                └─ admin       │
                                    │            │                  │
┌──────────────┐   /api proxy       │            ▼                  │      ┌──────────┐
│   Next.js    │ ─────────────────► │   Admin & Payment REST API    │ ───► │ MongoDB  │
│  dashboard   │   (X-Admin-Key)    │                               │      └──────────┘
└──────────────┘                    └───────────────┬───────────────┘
       ▲                                            │
       │      checkout page /pay/[token]            ▼
       └──────────────────────────────────── ┌────────────┐
                                             │  Razorpay  │
                                             └────────────┘
```

- **Backend** (`app/`) — a FastAPI service that receives Telegram updates via webhook, routes free-text messages through a Google ADK multi-agent system, and exposes REST endpoints for the dashboard and payments.
- **Frontend** (`web/`) — a Next.js App Router app for the admin dashboard and the customer payment page. Server-side route handlers proxy requests to the backend so the admin API key is never exposed to the browser.

---

## Tech Stack

| Layer      | Technology |
|------------|------------|
| Backend    | Python 3.12, FastAPI, Uvicorn, Pydantic Settings |
| Bot        | python-telegram-bot (webhook mode) |
| AI         | Google ADK, Google GenAI (Gemini) |
| Database   | MongoDB (Motor async driver) |
| Payments   | Razorpay |
| Frontend   | Next.js 16, React 19, TypeScript, Tailwind CSS 4, Radix UI, Framer Motion, React Three Fiber |
| Tooling    | uv (Python), npm (Node) |
| Deployment | Vercel, GitHub Actions |

---

## Project Structure

```
pallavi-bakery-bot/
├── app/                        # FastAPI backend
│   ├── main.py                 # App entrypoint, lifespan, CORS, routers
│   ├── config.py               # Settings loaded from environment / .env
│   ├── agents/                 # Google ADK agents (orchestrator + sub-agents)
│   ├── tools/                  # Tool functions the agents can call
│   ├── bot/
│   │   ├── setup.py            # Telegram application & handler registration
│   │   └── handlers/           # Customer and admin command/callback handlers
│   ├── api/routes/             # health, webhook, admin, payment endpoints
│   ├── database/
│   │   ├── connection.py       # MongoDB client lifecycle
│   │   └── repositories/       # menu, order, session, user data access
│   ├── models/                 # Pydantic models (menu, order, user)
│   └── services/               # Razorpay payment service
├── scripts/
│   ├── seed_menu.py            # Seed the menu collection with sample items
│   └── set_webhook.py          # Register the Telegram webhook URL
├── web/                        # Next.js admin dashboard + payment page
│   └── src/
│       ├── app/
│       │   ├── dashboard/      # Overview, orders, menu, QR code pages
│       │   ├── login/          # Admin login
│       │   ├── pay/[orderId]/  # Customer checkout page
│       │   └── api/            # Server-side proxies to the backend
│       ├── components/         # UI components
│       ├── hooks/              # Custom React hooks
│       └── lib/                # API client, auth helpers, utils
├── .github/workflows/ci.yml    # Deploy pipeline (Vercel)
├── pyproject.toml              # Python project & dependencies
├── requirements.txt            # Pinned deps for the Vercel Python runtime
└── uv.lock
```

---

## Getting Started

### Prerequisites

- **Python** 3.12+
- **[uv](https://docs.astral.sh/uv/)** package manager
- **Node.js** 20+ and npm
- **MongoDB** instance (local or MongoDB Atlas)
- **Telegram bot token** from [@BotFather](https://t.me/BotFather)
- **Gemini API key** from [Google AI Studio](https://aistudio.google.com/)
- **Razorpay account** (optional — only needed for online payments)
- A public HTTPS URL for the webhook (e.g. a Vercel deployment, or a tunnel such as ngrok for local development)

### Backend Setup

```bash
# 1. Clone the repository
git clone https://github.com/subrahmanyahegde24/pallavi-bakery-bot.git
cd pallavi-bakery-bot

# 2. Install dependencies
uv sync

# 3. Configure environment
#    Create a .env file in the project root (see "Environment Variables")

# 4. Seed the menu (run once)
uv run python scripts/seed_menu.py

# 5. Start the API server
uv run uvicorn app.main:app --reload --port 8000

# 6. Register the Telegram webhook (requires WEBHOOK_URL to be a public HTTPS URL)
uv run python scripts/set_webhook.py
```

Verify the server is running:

```bash
curl http://localhost:8000/health
# {"status":"ok","db":"connected"}
```

### Frontend Setup

```bash
cd web
npm install

# Create web/.env.local (see "Environment Variables")

npm run dev
```

The dashboard is available at <http://localhost:3000>. Log in with the value of `ADMIN_API_KEY`.

---

## Environment Variables

### Backend — `.env` (project root)

| Variable                  | Required | Default                                       | Description |
|---------------------------|:--------:|-----------------------------------------------|-------------|
| `TELEGRAM_TOKEN`          | Yes      | —                                             | Bot token from BotFather |
| `ADMIN_IDS`               | No       | `""`                                          | Comma-separated Telegram user IDs allowed to use admin commands |
| `WEBHOOK_URL`             | No       | `""`                                          | Public base URL of the backend (`/webhook` is appended) |
| `WEBHOOK_SECRET`          | No       | `""`                                          | Secret token Telegram sends with each webhook request |
| `GEMINI_API_KEY`          | Yes      | —                                             | Google Gemini API key |
| `MONGODB_URI`             | Yes      | —                                             | MongoDB connection string |
| `MONGODB_DB_NAME`         | No       | `bakery_bot`                                  | Database name |
| `ADMIN_API_KEY`           | Yes      | —                                             | Key required in the `X-Admin-Key` header for admin endpoints |
| `ENV`                     | No       | `development`                                 | Runtime environment name |
| `DASHBOARD_ORIGINS`       | No       | `http://localhost:3000,http://127.0.0.1:3000` | Comma-separated CORS origins |
| `FRONTEND_URL`            | No       | `http://localhost:3000`                       | Used to build payment links sent to customers |
| `RAZORPAY_KEY_ID`         | No       | `""`                                          | Razorpay key ID |
| `RAZORPAY_KEY_SECRET`     | No       | `""`                                          | Razorpay key secret |
| `RAZORPAY_WEBHOOK_SECRET` | No       | `""`                                          | Secret for verifying Razorpay webhooks |

Example:

```env
TELEGRAM_TOKEN=123456:ABC-your-bot-token
ADMIN_IDS=111111111,222222222
WEBHOOK_URL=https://your-backend.vercel.app
WEBHOOK_SECRET=change-me
GEMINI_API_KEY=your-gemini-key
MONGODB_URI=mongodb+srv://user:pass@cluster.mongodb.net
ADMIN_API_KEY=change-me
FRONTEND_URL=https://your-dashboard.vercel.app
DASHBOARD_ORIGINS=https://your-dashboard.vercel.app
RAZORPAY_KEY_ID=rzp_test_xxx
RAZORPAY_KEY_SECRET=xxx
RAZORPAY_WEBHOOK_SECRET=xxx
```

### Frontend — `web/.env.local`

| Variable              | Description |
|-----------------------|-------------|
| `ADMIN_API_URL`       | Backend base URL used by server-side proxies (preferred) |
| `NEXT_PUBLIC_API_URL` | Fallback backend base URL |
| `ADMIN_API_KEY`       | Same value as the backend `ADMIN_API_KEY` |

> **Never commit `.env` or `.env.local` files.** They are already listed in `.gitignore`.

---

## Bot Commands

### Customer

| Command  | Description |
|----------|-------------|
| `/start` | Greeting and introduction |
| `/menu`  | Browse menu categories |
| `/cart`  | View the current cart |
| `/track` | Check order status |

Any other text message is handled by the AI assistant.

### Admin (only for users listed in `ADMIN_IDS`)

| Command            | Description |
|--------------------|-------------|
| `/admin`           | Show admin help |
| `/orders`          | List active (pending / confirmed) orders |
| `/order <number>`  | Show full order details |
| `/confirm <number>`| Mark an order as confirmed |
| `/ready <number>`  | Mark an order as ready for pickup |
| `/cancel <number>` | Cancel an order |
| `/items`           | List menu items and availability |
| `/toggle <slug>`   | Toggle a menu item's availability |

---

## API Reference

Admin endpoints require the header `X-Admin-Key: <ADMIN_API_KEY>`.

| Method   | Endpoint                        | Auth  | Description |
|----------|---------------------------------|:-----:|-------------|
| `GET`    | `/health`                       | —     | Service and database health check |
| `POST`   | `/webhook`                      | Telegram secret | Telegram update receiver |
| `GET`    | `/admin/orders`                 | Admin | List orders |
| `GET`    | `/admin/orders/{order_id}`      | Admin | Get order details |
| `PATCH`  | `/admin/orders/{order_id}/status` | Admin | Update order status |
| `GET`    | `/admin/menu`                   | Admin | List menu items |
| `POST`   | `/admin/menu`                   | Admin | Create a menu item |
| `PATCH`  | `/admin/menu/{item_id}`         | Admin | Update a menu item |
| `DELETE` | `/admin/menu/{item_id}`         | Admin | Delete a menu item |
| `GET`    | `/payment/order/{payment_token}`| —     | Fetch order details for the checkout page |
| `POST`   | `/payment/verify`               | —     | Verify a Razorpay payment signature |
| `POST`   | `/payment/webhook`              | Razorpay signature | Razorpay webhook receiver |

Interactive API docs are available at `/docs` (Swagger UI) when the server is running.

---

## Order Lifecycle

```
PENDING ──► CONFIRMED ──► PREPARING ──► READY ──► PICKED_UP
   │             │             │           │
   └─────────────┴─────────────┴───────────┴──► CANCELLED
```

Payment status is tracked separately: `PENDING`, `PAID`, `FAILED`, or `COD` (pay at pickup).

---

## Testing

Frontend unit tests use the built-in Node.js test runner:

```bash
cd web
node --test "src/**/*.node.test.mjs"
npm run lint
```

---

## Deployment

Both the backend and the frontend are deployed to **Vercel** as separate projects. The GitHub Actions workflow in `.github/workflows/ci.yml` deploys both on every push to `master`.

Configure these repository secrets in **GitHub → Settings → Secrets and variables → Actions**:

| Secret                       | Description |
|------------------------------|-------------|
| `VERCEL_TOKEN`               | Vercel personal access token |
| `VERCEL_ORG_ID`              | Vercel team / account ID |
| `VERCEL_BACKEND_PROJECT_ID`  | Vercel project ID for the backend |
| `VERCEL_FRONTEND_PROJECT_ID` | Vercel project ID for the frontend |

Set the environment variables listed above in each Vercel project. After the backend is deployed, register the webhook once:

```bash
uv run python scripts/set_webhook.py
```

> When changing Python dependencies, regenerate `requirements.txt` so the Vercel Python runtime stays in sync with `uv.lock`:
> ```bash
> uv export --no-hashes --format requirements-txt -o requirements.txt
> ```

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m "Add my feature"`
4. Push the branch: `git push origin feature/my-feature`
5. Open a Pull Request

---

## License

This project is proprietary. All rights reserved. Contact the repository owner for usage permissions.

---

**Author:** [Subrahmanya Shivaram Hegde](https://github.com/subrahmanyahegde24)
