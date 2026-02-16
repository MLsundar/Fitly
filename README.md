# 🧵 Fitly — Custom Tailoring Marketplace

> **Powered by NOVAPLM** | Connecting consumers with expert tailors through smart bidding

## Overview

Fitly is an Uber-like marketplace for custom tailoring. Consumers take body measurements using their phone camera, post tailoring requests, and receive competitive bids from verified tailors.

### Payment Flow
```
Consumer → Pays NOVAPLM (100%) → NOVAPLM keeps 20% → Tailor receives 80%
```

## Features

### Consumer
- 📸 **Camera Measurements** — Use phone/computer camera to capture body measurements
- 👔 **Garment Selection** — Pants, Half-sleeve shirts, Full-sleeve shirts, Shorts (Comfort fit)
- 🏷️ **Receive Bids** — Tailors compete with prices and timelines
- 💳 **Secure Payment** — Pay through NOVAPLM via Stripe
- 📋 **Order Tracking** — Track from bid to completion

### Tailor
- 🔍 **Browse Orders** — See open consumer requests with measurements
- 💰 **Place Bids** — Compete on price, timeline, and quality
- ✂️ **Manage Jobs** — Track won jobs and earnings
- 📊 **Dashboard** — Revenue tracking (80% of payments)

## Tech Stack

| Layer      | Technology                          |
|------------|-------------------------------------|
| Frontend   | Vanilla JS SPA (responsive web app) |
| Backend    | Node.js + Express                   |
| Database   | PostgreSQL via Prisma ORM           |
| Auth       | JWT (separate consumer/tailor)      |
| Payments   | Stripe (NOVAPLM as platform)        |
| Hosting    | Microsoft Azure                     |

## Project Structure

```
fitly/
├── backend/
│   ├── prisma/
│   │   └── schema.prisma        # Database schema
│   ├── src/
│   │   ├── server.js            # Express entry point
│   │   ├── config/
│   │   │   └── database.js      # Prisma client
│   │   ├── middleware/
│   │   │   └── auth.js          # JWT authentication
│   │   └── routes/
│   │       ├── auth.js          # Registration & login
│   │       ├── consumer.js      # Consumer profile & dashboard
│   │       ├── tailor.js        # Tailor profile, orders, bids
│   │       ├── order.js         # Order CRUD, bid selection
│   │       ├── bid.js           # Bid CRUD
│   │       ├── payment.js       # Stripe payments & payouts
│   │       └── measurement.js   # Body measurements
│   ├── Dockerfile
│   ├── .env.example
│   └── package.json
├── frontend/
│   └── index.html               # Complete SPA
├── AZURE_DEPLOYMENT.md          # Azure setup guide
└── README.md
```

## Quick Start (Local Development)

### 1. Database
```bash
# Start PostgreSQL (Docker)
docker run -d --name fitly-db \
  -e POSTGRES_DB=fitly \
  -e POSTGRES_USER=fitlyadmin \
  -e POSTGRES_PASSWORD=password123 \
  -p 5432:5432 \
  postgres:15

# Or use a local PostgreSQL instance
```

### 2. Backend
```bash
cd backend
cp .env.example .env
# Edit .env with your database URL

npm install
npx prisma generate
npx prisma migrate dev --name init
npm run dev
# API running at http://localhost:4000
```

### 3. Frontend
```bash
cd frontend
# Open index.html in browser, or serve it:
npx serve .
# Frontend running at http://localhost:3000
```

## API Endpoints

### Auth
| Method | Endpoint                     | Description          |
|--------|------------------------------|----------------------|
| POST   | /api/auth/consumer/register  | Register consumer    |
| POST   | /api/auth/consumer/login     | Consumer login       |
| POST   | /api/auth/tailor/register    | Register tailor      |
| POST   | /api/auth/tailor/login       | Tailor login         |
| GET    | /api/auth/verify             | Verify JWT token     |

### Consumer
| Method | Endpoint                | Description           |
|--------|-------------------------|-----------------------|
| GET    | /api/consumer/profile   | Get profile           |
| PUT    | /api/consumer/profile   | Update profile        |
| GET    | /api/consumer/orders    | Get my orders         |
| GET    | /api/consumer/dashboard | Dashboard stats       |

### Tailor
| Method | Endpoint                      | Description              |
|--------|-------------------------------|--------------------------|
| GET    | /api/tailor/profile           | Get profile              |
| PUT    | /api/tailor/profile           | Update profile           |
| GET    | /api/tailor/available-orders  | Browse open orders       |
| GET    | /api/tailor/my-bids           | My submitted bids        |
| GET    | /api/tailor/my-jobs           | My won jobs              |
| GET    | /api/tailor/dashboard         | Dashboard stats          |

### Orders
| Method | Endpoint                       | Description          |
|--------|--------------------------------|----------------------|
| POST   | /api/orders                    | Create order         |
| GET    | /api/orders/:id                | Get order details    |
| POST   | /api/orders/:id/select-bid     | Accept a bid         |
| POST   | /api/orders/:id/complete       | Mark completed       |
| POST   | /api/orders/:id/cancel         | Cancel order         |

### Bids
| Method | Endpoint                 | Description          |
|--------|--------------------------|----------------------|
| POST   | /api/bids                | Place a bid          |
| PUT    | /api/bids/:id            | Update bid           |
| POST   | /api/bids/:id/withdraw   | Withdraw bid         |

### Measurements
| Method | Endpoint                  | Description           |
|--------|---------------------------|-----------------------|
| POST   | /api/measurements         | Save measurements     |
| GET    | /api/measurements         | Get all measurements  |
| GET    | /api/measurements/:id     | Get one measurement   |
| PUT    | /api/measurements/:id     | Update measurement    |
| DELETE | /api/measurements/:id     | Delete measurement    |

### Payments
| Method | Endpoint                         | Description            |
|--------|----------------------------------|------------------------|
| POST   | /api/payments/create             | Create payment         |
| POST   | /api/payments/confirm/:id        | Confirm payment        |
| GET    | /api/payments/history            | Payment history        |

## Future Enhancements
- [ ] Fabric & color selection
- [ ] Additional fit styles (Slim, Tailored, Loose)
- [ ] Women's garments
- [ ] Ratings & reviews
- [ ] Real-time notifications (WebSocket)
- [ ] AI-powered measurement extraction (MediaPipe/TensorFlow)
- [ ] Tailor portfolio gallery
- [ ] In-app chat between consumer and tailor

---

© 2026 NOVAPLM. All rights reserved.
