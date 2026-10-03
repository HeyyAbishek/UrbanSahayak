# PadosiPro: Local Lifestyle & Home Services Platform

> A complete, production-ready React Native application and Node.js backend designed to connect residents with dedicated Lifestyle Managers (LMs) for errands, household management, and concierge services.

---

## 🚀 Tech Stack Highlights

- **Mobile Client:** React Native, Expo (SDK 57), TypeScript
- **Styling:** NativeWind (Tailwind CSS) for native responsive design tokens (no WebViews).
- **API & Backend:** Node.js, Express
- **Database Layer:** PostgreSQL 16 managed via Prisma ORM
- **Local Dev Tools:** Docker (Postgres + Mailpit for SMTP interception)

---

## ✨ Platform Features

- **Advanced Catalog Discovery:** A highly optimized search index supporting word-level weighting. Users can search across 50+ services and find direct partner integrations (e.g., PolicyBazaar, Booking.com).
- **Unified Batch Requests:** Users can select multiple tasks across different categories, assign global urgency levels (Standard, Same Day, Express, Scheduled), and submit everything in a single transaction.
- **Resilient Authentication:** A secure dual-layer system. Features bcrypt password hashing and a 6-digit OTP email flow equipped with rate-limiting, 30-second cooldowns, and brute-force protection.
- **Household Management:** Users can maintain profiles for family members, staff, or pets to provide context to their assigned Lifestyle Manager.
- **Seamless Local Testing:** Zero-config local development utilizing Mailpit to capture outgoing OTP emails and automated database seeding.

---

## 🏗️ Architecture & Navigation Flow

**The Application Flow:**

`Registration` ➔ `OTP Verification (Memory-safe)` ➔ `JWT Auto Sign-In` ➔ `Address Onboarding` ➔ `Dashboard` ➔ `Task Multi-select` ➔ `Urgency Configuration` ➔ `Submission`

**Project Structure:**

- `/mobile`: The frontend client. Uses Expo Router for file-based navigation (split into `/auth`, `/onboarding`, and `/main`). Reusable design primitives are located in `/components`.
- `/server`: The Express backend. Contains modular route handlers, JWT middleware, Zod validation, and the Prisma schema/seed scripts.
- `docker-compose.yml`: Spins up local Postgres and Mailpit for a sandbox environment.

---

## 🛠️ Quick Start Guide

### 1. Prerequisites

Ensure you have **Node.js (v18+)**, **Docker Desktop**, and the **Expo Go** app installed on your phone (or an Android Emulator running).

### 2. Bootstrapping the Backend

Install dependencies from the root, start the Docker containers, and initialize the database:

```bash
npm install
docker compose up -d

cd server
cp .env.example .env
npx prisma migrate dev
npx prisma db seed
npm run dev
```

The API will boot up on:

`http://localhost:3001`

### 3. Running the Mobile App

In a new terminal window, configure the environment and start Metro:

```bash
cd mobile
cp .env.example .env
npx expo start
```

*Scan the QR code with Expo Go, or press `w` to launch the application directly in your local web browser for rapid testing.*

---

## 🌐 Network Configuration for Devices

Depending on how you test the app, you must update `EXPO_PUBLIC_API_URL` in `mobile/.env`:

- **Physical Device (Same Wi-Fi):** Use your computer's local IP (e.g., `http://192.168.1.5:3001`).
- **Android Emulator:** Set to `http://10.0.2.2:3001`.
- **USB Debugging:** Set to `http://localhost:3001` and run the following ADB reverse ports:

```bash
adb reverse tcp:8081 tcp:8081
adb reverse tcp:3001 tcp:3001
```

> **Note:** Always restart Expo with `npx expo start -c` after changing environment variables.

---

## ⚙️ Environment Variables

### Server (`server/.env`)

- `DATABASE_URL`: Postgres connection string (Required)
- `JWT_SECRET`: Secret key for token signing (Required)
- `PORT`: API Port (Default: 3001)
- `OTP_EXPIRY_MINUTES`: Code lifespan (Default: 10)
- `OTP_RESEND_SECONDS`: Cooldown timer (Default: 30)
- `OTP_MAX_ATTEMPTS`: Lockout threshold (Default: 5)
- `SMTP_HOST` / `SMTP_PORT`: Email server (Defaults to localhost:1025 for Mailpit)

### Client (`mobile/.env`)

- `EXPO_PUBLIC_API_URL`: Backend connection URL (Required)

---

## 🧪 Testing & Deployment

### Automated Tests

```bash
npm run typecheck         # Workspace type validation
npm run test:server       # Unit tests
```

### Integration Testing

To run the auth flow tests against an isolated DB instance:

```bash
docker exec padosipro-postgres createdb -U padosi padosipro_test

cd server

TEST_DATABASE_URL="postgresql://padosi:padosi@localhost:5432/padosipro_test?schema=public" npx prisma migrate deploy

TEST_DATABASE_URL="postgresql://padosi:padosi@localhost:5432/padosipro_test?schema=public" npm run test:integration
```

### Building the APK

The project is pre-configured for EAS build. To generate a standalone Android `.apk`:

```bash
cd mobile
npx eas-cli login
npx eas-cli build -p android --profile preview
```

---

## 🛡️ Security Highlights

- **Safe Credential Handling:** Development OTPs are handled securely in-memory and logged to the terminal/Mailpit, avoiding URL parameter leaks.
- **Data Sanitization:** E.164-compatible mobile number normalization, email lowercasing, and strict Zod payload validation on every endpoint.
- **Orphan Data Prevention:** PostgreSQL cascading deletes ensure relational integrity across households, profiles, and tasks.
