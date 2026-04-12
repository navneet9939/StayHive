# StayHive — Production System Architecture
## Series-A Student Housing Marketplace

**Version:** 1.0.0
**Date:** 2026-02-25
**Authors:** Principal Architecture Team

---

## Table of Contents

1. [Enterprise Folder Structure](#1-enterprise-folder-structure)
2. [Database Design](#2-database-design)
3. [RBAC Model](#3-rbac-model)
4. [API Structure](#4-api-structure)
5. [Frontend UI System](#5-frontend-ui-system)
6. [Performance Strategy](#6-performance-strategy)
7. [Security Hardening](#7-security-hardening)
8. [Scalability Roadmap](#8-scalability-roadmap)
9. [Advanced Enterprise Features](#9-advanced-enterprise-features)
10. [UI/UX Design System](#10-uiux-design-system)

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                        StayHive Platform                          │
├─────────────────┬───────────────────────────┬───────────────────────┤
│   CDN (Vercel)  │   API Gateway (Nginx)      │   WebSocket (Socket.io)│
│   Next.js SSR   │   Express Cluster          │   Real-time Chat       │
├─────────────────┴───────────────────────────┴───────────────────────┤
│                     Service Mesh (Internal)                         │
│  Auth Svc │ Listing Svc │ Booking Svc │ AI Svc │ Payment Svc        │
├─────────────────────────────────────────────────────────────────────┤
│   MongoDB Atlas (Primary)  │  Redis Cluster  │  Elasticsearch        │
│   + Read Replicas          │  (Cache/Queue)  │  (Search Engine)      │
├─────────────────────────────────────────────────────────────────────┤
│   Cloudinary (Media) │ SendGrid (Email) │ Stripe (Payments)          │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 1. Enterprise Folder Structure

```
StayHive/
├── apps/
│   ├── frontend/                    # Next.js 14 App Router
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── (auth)/
│   │   │   │   │   ├── login/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   ├── register/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   └── layout.tsx
│   │   │   │   ├── (main)/
│   │   │   │   │   ├── listings/
│   │   │   │   │   │   ├── [id]/
│   │   │   │   │   │   │   ├── page.tsx       # ISR listing detail
│   │   │   │   │   │   │   └── loading.tsx
│   │   │   │   │   │   └── page.tsx           # Search/browse
│   │   │   │   │   ├── search/
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   └── layout.tsx
│   │   │   │   ├── dashboard/
│   │   │   │   │   ├── (student)/
│   │   │   │   │   │   ├── bookings/page.tsx
│   │   │   │   │   │   ├── saved/page.tsx
│   │   │   │   │   │   └── messages/page.tsx
│   │   │   │   │   ├── (landlord)/
│   │   │   │   │   │   ├── properties/page.tsx
│   │   │   │   │   │   ├── analytics/page.tsx
│   │   │   │   │   │   └── earnings/page.tsx
│   │   │   │   │   ├── (admin)/
│   │   │   │   │   │   ├── users/page.tsx
│   │   │   │   │   │   ├── fraud/page.tsx
│   │   │   │   │   │   └── reports/page.tsx
│   │   │   │   │   └── layout.tsx
│   │   │   │   ├── api/               # Next.js API routes (BFF pattern)
│   │   │   │   │   └── revalidate/
│   │   │   │   │       └── route.ts
│   │   │   │   ├── layout.tsx         # Root layout + providers
│   │   │   │   ├── page.tsx           # Homepage
│   │   │   │   ├── not-found.tsx
│   │   │   │   └── error.tsx
│   │   │   ├── components/
│   │   │   │   ├── ui/                # Base design system components
│   │   │   │   │   ├── Button.tsx
│   │   │   │   │   ├── Input.tsx
│   │   │   │   │   ├── Modal.tsx
│   │   │   │   │   ├── Badge.tsx
│   │   │   │   │   ├── Skeleton.tsx
│   │   │   │   │   ├── Toast.tsx
│   │   │   │   │   └── index.ts
│   │   │   │   ├── listings/
│   │   │   │   │   ├── ListingCard.tsx
│   │   │   │   │   ├── ListingGrid.tsx
│   │   │   │   │   ├── ListingDetail.tsx
│   │   │   │   │   ├── ListingGallery.tsx
│   │   │   │   │   ├── PriceCalculator.tsx
│   │   │   │   │   └── FilterPanel.tsx
│   │   │   │   ├── chat/
│   │   │   │   │   ├── ChatWindow.tsx
│   │   │   │   │   ├── MessageBubble.tsx
│   │   │   │   │   └── ConversationList.tsx
│   │   │   │   ├── maps/
│   │   │   │   │   ├── MapView.tsx
│   │   │   │   │   └── MapMarker.tsx
│   │   │   │   └── forms/
│   │   │   │       ├── BookingForm.tsx
│   │   │   │       ├── ListingForm.tsx
│   │   │   │       └── ProfileForm.tsx
│   │   │   ├── lib/
│   │   │   │   ├── api/
│   │   │   │   │   ├── client.ts      # Axios instance + interceptors
│   │   │   │   │   ├── listings.ts
│   │   │   │   │   ├── auth.ts
│   │   │   │   │   └── bookings.ts
│   │   │   │   ├── hooks/
│   │   │   │   │   ├── useListings.ts
│   │   │   │   │   ├── useAuth.ts
│   │   │   │   │   └── useSocket.ts
│   │   │   │   └── utils/
│   │   │   │       ├── formatters.ts
│   │   │   │       └── validators.ts
│   │   │   ├── store/
│   │   │   │   ├── index.ts           # Zustand store root
│   │   │   │   └── slices/
│   │   │   │       ├── authSlice.ts
│   │   │   │       ├── listingSlice.ts
│   │   │   │       └── chatSlice.ts
│   │   │   ├── styles/
│   │   │   │   ├── globals.css
│   │   │   │   └── animations.css
│   │   │   └── types/
│   │   │       └── index.ts
│   │   ├── public/
│   │   ├── next.config.ts
│   │   ├── tailwind.config.ts
│   │   └── tsconfig.json
│   │
│   └── backend/                      # Express API Server
│       ├── src/
│       │   ├── server.ts              # Entry point
│       │   ├── app.ts                 # Express app factory
│       │   ├── config/
│       │   │   ├── index.ts           # Config aggregator
│       │   │   ├── database.ts        # MongoDB connection
│       │   │   ├── redis.ts           # Redis connection
│       │   │   └── swagger.ts         # OpenAPI config
│       │   ├── modules/               # Feature modules (DDD approach)
│       │   │   ├── auth/
│       │   │   │   ├── auth.controller.ts
│       │   │   │   ├── auth.service.ts
│       │   │   │   ├── auth.router.ts
│       │   │   │   ├── auth.validator.ts
│       │   │   │   └── auth.types.ts
│       │   │   ├── listings/
│       │   │   │   ├── listing.controller.ts
│       │   │   │   ├── listing.service.ts
│       │   │   │   ├── listing.router.ts
│       │   │   │   ├── listing.model.ts
│       │   │   │   ├── listing.validator.ts
│       │   │   │   └── listing.types.ts
│       │   │   ├── users/
│       │   │   ├── bookings/
│       │   │   ├── reviews/
│       │   │   ├── chat/
│       │   │   ├── payments/
│       │   │   ├── notifications/
│       │   │   └── analytics/
│       │   ├── middleware/
│       │   │   ├── authenticate.ts    # JWT verification
│       │   │   ├── authorize.ts       # RBAC enforcement
│       │   │   ├── rateLimiter.ts
│       │   │   ├── errorHandler.ts
│       │   │   ├── requestLogger.ts
│       │   │   ├── validate.ts        # Zod validation
│       │   │   └── cacheMiddleware.ts
│       │   ├── services/
│       │   │   ├── ai/
│       │   │   │   ├── pricingEngine.ts
│       │   │   │   └── recommendationEngine.ts
│       │   │   ├── fraud/
│       │   │   │   └── fraudDetector.ts
│       │   │   ├── cache/
│       │   │   │   └── cacheService.ts
│       │   │   ├── storage/
│       │   │   │   └── cloudinaryService.ts
│       │   │   └── email/
│       │   │       └── emailService.ts
│       │   ├── jobs/                  # Background jobs (Bull queues)
│       │   │   ├── emailQueue.ts
│       │   │   ├── imageProcessing.ts
│       │   │   └── analyticsAggregator.ts
│       │   ├── utils/
│       │   │   ├── logger.ts          # Winston configuration
│       │   │   ├── apiResponse.ts
│       │   │   ├── pagination.ts
│       │   │   └── encryption.ts
│       │   └── types/
│       │       └── express.d.ts       # Express type augmentation
│       ├── tests/
│       ├── Dockerfile
│       └── tsconfig.json
│
├── packages/
│   ├── shared-types/                  # Shared TypeScript interfaces
│   │   └── src/
│   │       ├── user.types.ts
│   │       ├── listing.types.ts
│   │       └── api.types.ts
│   └── ui-kit/                        # Shared Storybook component library
│
├── infra/
│   ├── docker/
│   │   ├── docker-compose.yml         # Local dev stack
│   │   └── docker-compose.prod.yml
│   ├── nginx/
│   │   └── nginx.conf
│   └── terraform/                     # IaC for AWS
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
│
├── docs/
│   └── adr/                           # Architecture Decision Records
│
└── scripts/
    ├── seed.ts
    └── migrate.ts
```

---

See individual section files for implementation details:
- `/docs/02-database-design.md`
- `/docs/03-rbac-model.md`
- `/docs/04-api-structure.md`
- `/docs/05-frontend-ui.md`
- `/docs/06-performance.md`
- `/docs/07-security.md`
- `/docs/08-scalability.md`
- `/docs/09-enterprise-features.md`
- `/docs/10-design-system.md`
