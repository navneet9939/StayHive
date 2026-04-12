# StayHive

A full-stack student housing marketplace connecting university students with verified PGs, apartments, hostels, and studios near their campuses.

---

## Tech Stack

**Frontend**
- Next.js 15 (App Router) · TypeScript · Tailwind CSS
- Framer Motion · Zustand · TanStack Query v5
- React Hook Form + Zod · Lucide React · Radix UI

**Backend**
- Node.js + Express · TypeScript
- MongoDB + Mongoose · Redis (ioredis)
- Cloudinary · Nodemailer · Winston
- JWT (access + refresh tokens) · Helmet · Swagger

**DevOps**
- Docker + docker-compose · GitHub Actions CI/CD
- Render (backend) · Vercel (frontend) · MongoDB Atlas

---

## Project Structure

```
StayHive/
├── client/                  # Next.js App Router frontend
│   ├── src/
│   │   ├── app/             # Pages (App Router)
│   │   │   ├── (auth)/      # Login, register, forgot/reset password
│   │   │   ├── (dashboard)/ # Student, Owner, Admin dashboards
│   │   │   └── (main)/      # Homepage, property listing, detail
│   │   ├── components/
│   │   │   ├── atoms/       # Button, Input, Badge, Avatar, Skeleton
│   │   │   ├── molecules/   # PropertyCard, BookingPanel, ReviewCard, SearchBar
│   │   │   └── organisms/   # Navbar, HeroSection, PropertyGrid, FilterSidebar, Footer
│   │   ├── hooks/           # useDebounce, useMediaQuery, useLocalStorage, useIntersectionObserver
│   │   ├── lib/
│   │   │   ├── api/         # Axios client + auth/property/booking API modules
│   │   │   └── queryClient.ts
│   │   ├── store/           # Zustand: auth, ui, property stores
│   │   ├── types/           # TypeScript interfaces
│   │   └── utils/           # cn, formatters, motion variants
│   ├── Dockerfile
│   └── package.json
│
├── server/                  # Express backend
│   ├── src/
│   │   ├── config/          # db, redis, cloudinary, logger, swagger, env
│   │   ├── controllers/     # auth, property, booking, review, user, admin
│   │   ├── middlewares/     # auth, rbac, validate, errorHandler, upload, rateLimiter
│   │   ├── models/          # User, Property, Booking, Review, Payment, Commission, Audit
│   │   ├── repositories/    # Base + entity-specific DB abstractions
│   │   ├── routes/          # auth, property, booking, review, user, admin
│   │   ├── services/        # auth, property, booking, review, user, cache, notification, commission
│   │   ├── utils/           # ApiError, ApiResponse, jwt, hash, paginator, slugify
│   │   ├── validations/     # Zod schemas for all request types
│   │   ├── app.ts
│   │   └── server.ts
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml
├── .github/workflows/ci.yml
└── README.md
```

---

## Prerequisites

- Node.js >= 20
- Docker + Docker Compose
- MongoDB Atlas account (or local MongoDB)
- Redis instance (or use docker-compose)
- Cloudinary account
- SMTP credentials (Gmail, Resend, etc.)

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-username/StayHive.git
cd stayHive
```

### 2. Environment configuration

**Backend** (`server/.env`):
```bash
cp server/.env.example server/.env
```

Edit `server/.env`:
```env
NODE_ENV=development
PORT=5000
MONGODB_URI=mongodb+srv://<user>:<pass>@cluster.mongodb.net/stayhive
REDIS_URL=redis://localhost:6379

JWT_ACCESS_SECRET=<32+ char random string>
JWT_REFRESH_SECRET=<32+ char random string>
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d

CLOUDINARY_CLOUD_NAME=<your cloud name>
CLOUDINARY_API_KEY=<your api key>
CLOUDINARY_API_SECRET=<your api secret>

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=you@gmail.com
SMTP_PASS=<app password>
SMTP_FROM_NAME=StayHive
SMTP_FROM_EMAIL=noreply@stayhive.com

CLIENT_URL=http://localhost:3000
```

**Frontend** (`client/.env.local`):
```bash
cp client/.env.example client/.env.local
```

Edit `client/.env.local`:
```env
NEXT_PUBLIC_API_URL=http://localhost:5000/api/v1
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

---

## Running with Docker Compose (recommended)

```bash
docker-compose up --build
```

Services started:
- Frontend: http://localhost:3000
- Backend API: http://localhost:5000
- MongoDB: localhost:27017
- Redis: localhost:6379

---

## Running without Docker

### Backend

```bash
cd server
npm install
npm run dev
```

The server starts at `http://localhost:5000`. API docs available at `http://localhost:5000/api-docs`.

### Frontend

```bash
cd client
npm install
npm run dev
```

The app starts at `http://localhost:3000`.

---

## API Overview

Base URL: `http://localhost:5000/api/v1`

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/auth/register` | Register new user | Public |
| POST | `/auth/login` | Login | Public |
| POST | `/auth/refresh-token` | Rotate access token | Cookie |
| POST | `/auth/logout` | Logout & clear cookie | JWT |
| POST | `/auth/forgot-password` | Send reset email | Public |
| POST | `/auth/reset-password` | Reset password | Public |
| GET | `/properties` | List properties (filters) | Optional |
| GET | `/properties/featured` | Featured properties | Public |
| GET | `/properties/:slug` | Property detail | Optional |
| POST | `/properties` | Create property | Owner |
| PATCH | `/properties/:id` | Update property | Owner |
| DELETE | `/properties/:id` | Delete property | Owner |
| POST | `/properties/:id/verify` | Verify property | Admin |
| POST | `/bookings` | Create booking | Student |
| GET | `/bookings` | My bookings | JWT |
| PATCH | `/bookings/:id/confirm` | Confirm booking | Owner |
| PATCH | `/bookings/:id/cancel` | Cancel booking | JWT |
| POST | `/reviews` | Post review | Student |
| GET | `/reviews/property/:id` | Property reviews | Public |
| GET | `/users/me` | My profile | JWT |
| PATCH | `/users/me` | Update profile | JWT |
| GET | `/admin/stats` | Platform stats | Admin |
| GET | `/admin/users` | All users | Admin |
| PATCH | `/admin/users/:id/status` | Update user status | Admin |

Full interactive docs: `http://localhost:5000/api-docs` (development only)

---

## Database Schema Overview

```
User          → role: student | owner | admin
Property      → owner: User, location: GeoJSON, images[], stats:{avgRating, reviewCount, viewCount}
Booking       → student: User, property: Property, status: pending→confirmed→completed|cancelled
Review        → student: User, booking: Booking (1:1), property: Property
Payment       → booking: Booking, gateway fields
Commission    → booking: Booking, tiered rate (10% / 8% / 6%)
AuditLog      → TTL: 90 days, polymorphic entity reference
University    → name, slug, location: GeoJSON
```

---

## Features

- **Role-based access control** — Student, Owner, Admin with composable middleware guards
- **JWT refresh token rotation** — Stateful tokens with reuse detection (revokes all sessions on reuse)
- **Redis caching** — TTL-based per entity, SCAN-based pattern invalidation (non-blocking)
- **Geospatial search** — `$near` queries for properties close to universities (2dsphere index)
- **Full-text search** — Weighted MongoDB text index on property title, description, address
- **Infinite scroll** — TanStack Query `useInfiniteQuery` with IntersectionObserver
- **Image uploads** — Cloudinary via Multer storage adapter, cleanup on property deletion
- **Commission tiering** — Automatic 10%/8%/6% based on owner lifetime revenue
- **Email notifications** — Booking confirmations, password reset, verifications
- **Rate limiting** — Redis-backed per-route limits (auth: 10/15min, search: 60/min)
- **Brute-force protection** — Account lock after 5 failed login attempts (30-min cooldown)
- **SSR/SSG hybrid** — Homepage (SSG+ISR), listings and detail (SSR), dashboards (CSR)
- **Dark mode** — CSS custom property theming with Zustand persistence
- **Audit logging** — Immutable audit trail with 90-day TTL

---

## CI/CD Pipeline

GitHub Actions workflow (`.github/workflows/ci.yml`):

1. **Lint & type-check** — backend + frontend in parallel
2. **Backend tests** — with ephemeral Mongo + Redis services
3. **Frontend build** — Next.js production build validation
4. **Docker build & push** — to GitHub Container Registry (main branch only)
5. **Deploy** — Render (backend) + Vercel (frontend), production environment

Required GitHub secrets:
```
RENDER_API_KEY, RENDER_SERVICE_ID
VERCEL_TOKEN, VERCEL_ORG_ID, VERCEL_PROJECT_ID
NEXT_PUBLIC_API_URL, NEXT_PUBLIC_APP_URL
CODECOV_TOKEN (optional)
```

---

## Security

- HTTP-only, Secure, SameSite=Strict refresh token cookie
- Helmet CSP/HSTS/X-Frame-Options headers
- `express-mongo-sanitize` against NoSQL injection
- Zod validation on all inputs
- Rate limiting on all sensitive endpoints
- Passwords hashed with bcrypt (12 rounds)
- JWT access token stored in memory only (never in localStorage)

---

## License

MIT
