# Engineering Design Document - PadosiPro

## System Architecture & Tech Stack

This project is structured as a monorepo utilizing npm workspaces, split into `mobile/`, `server/`, and a `docker-compose.yml` infrastructure configuration.

- **Mobile Client (Expo SDK 57 & React Native):**
  Built using Expo Router for file-based navigation (segregated into `auth/`, `onboarding/`, and `main/` groups). The UI is strictly native, styled via Tailwind CSS through NativeWind. Design tokens are synchronized between `tailwind.config.js` and `constants/theme.ts`, with reusable primitives stored in `components/ui.tsx`. Business logic is abstracted away from screens: `services/` handles typed API calls, while `store/AuthContext.tsx` manages the JWT lifecycle via SecureStore and dictates navigation state (auth vs. onboarding vs. dashboard).
- **Search & Discovery Engine:**
  A highly optimized, client-side indexing system located in `mobile/services/searchIndex.ts`. It performs fuzzy, word-level matching across service names, categories, and curated aliases. It dynamically calculates the best matches, suggests alternative categories, and supports direct partner routing (e.g., forwarding users to Booking.com or PolicyBazaar for DIY tasks).
- **Backend Service (Node.js, Express, TypeScript):**
  A lightweight API interfacing with PostgreSQL 16 via Prisma ORM. It enforces strict data validation on every endpoint using Zod and standardizes all responses into a predictable `{ ok, error }` shape. Authentication relies on bcrypt for hashing passwords and OTPs, issuing 30-day JWTs for session management. Email delivery is governed by an `EmailService` abstraction: it falls back to console logging locally, uses Mailpit when `SMTP_HOST` is defined, and easily swaps to a real provider for production.
- **Data Modeling:**
  Core entities include `User`, `OtpCode`, `UserProfile`, `HouseholdMember`, and `ServiceRequest`. The service catalog is injected via database seeding, containing 14 distinct categories that map to 24 help-kinds and 27 specific services (yielding 51 unique, selectable tasks, easily satisfying the assignment's >20 task requirement).

## Key Architectural Decisions (Trade-offs)

- **Dual-Layer Authentication (Password + OTP):** Following the brief's requirements, verified users log in with an email and password rather than an OTP-only flow. Passwords are securely hashed with bcrypt. If an unverified user successfully submits a password, they receive a 403 response and are automatically routed to the OTP verification screen.
- **Account Collision Safeguards:** Registration strictly requires a unique 10-digit mobile number. If a user attempts to register with an email that is already verified, the system intercepts the request and cleanly redirects them to the login screen with their email pre-filled.
- **Request Granularity:** Rather than creating a complex "multi-item order" table, the system generates one distinct `ServiceRequest` record per selected service. This aligns perfectly with real-world Lifestyle Manager workflows (where different tasks have different lifecycles) while reusing the existing database schema.
- **Unified Confirmation Step:** Instead of forcing users to configure urgency on a per-task basis, the UI groups multi-select picks into a single confirmation and urgency-selection screen, minimizing friction.
- **Optional Business Profiles:** Forcing users to enter a business name creates unnecessary drop-off during onboarding. It is left optional, allowing LMs to collect it later if a specific task requires it.
- **Mailpit vs. Ethereal:** Mailpit was chosen for local SMTP testing because it runs entirely offline via Docker Compose, provides a clean UI at `:8025`, and utilizes the exact same codebase paths as a production SMTP setup.
- **Expo Go Optimization:** The app relies strictly on Expo-supported native modules, intentionally avoiding custom native code. This ensures reviewers can instantly run and evaluate the app using the standard Expo Go client without needing to compile a dev build.

## Security Posture & UI Resilience

- **Secure Credential Passing:** Development OTPs generated during the auth flow are transmitted via in-memory state management. This prevents sensitive codes from leaking into URL query parameters or device navigation history.
- **Environment & Origin Strictness:** In production mode, the Express server will refuse to boot if `JWT_SECRET` is missing. Additionally, CORS policies are tightly controlled via the `ALLOWED_ORIGINS` variable.
- **Input & Keyboard Resiliency:** The onboarding UI utilizes dynamic `onLayout` tracking to handle soft keyboards. This ensures lower-screen inputs (like the Business Name field) remain visible and are not obscured when typing on smaller devices.

## Out of Scope for MVP

To adhere to the core assignment brief, several peripheral features were intentionally omitted. There is no payment gateway integration, wallet top-up system, push notification service, or admin dashboard. The "Chat" button on the Lifestyle Manager card is purely visual. Features marked as "Soon" in the UI catalog render correctly but are non-interactive. Lastly, JWT refresh-token rotation was excluded, as a 30-day token stored in SecureStore is sufficient for this evaluation.

## Roadmap & Future Enhancements

Given an additional week of development time, the following improvements would be implemented:

1. Integrate a live SMTP provider (e.g., SendGrid/SES) and remove the local terminal OTP echoes.
2. Implement robust refresh-token rotation alongside a server-side logout denylist.
3. Build a real-time request tracking timeline UI driven by updates to `ServiceRequest.status`.
4. Generate an EAS preview APK and record a comprehensive end-to-end demo video.
5. Migrate the service catalog from hardcoded seed scripts to an admin-editable database table.
