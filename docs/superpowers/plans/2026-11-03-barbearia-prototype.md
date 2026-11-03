# Protótipo de Agendamento de Barbearia — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a local React website where a customer books a barber appointment and the barber can review it in a password-gated agenda backed by SQLite.

**Architecture:** A Vite React client talks to a small Express API written in TypeScript. The API owns catalog data, availability rules, appointment persistence, and a simple in-memory demo session; SQLite stores the catalog and appointments in a local ignored data directory.

**Tech Stack:** React, TypeScript, Vite, Express, SQLite (`better-sqlite3`).

**Spec:** `docs/superpowers/specs/2026-11-03-barbearia-design.md`

## Global Constraints

- Scope is the landing/booking page and the barber's appointments page.
- Use editable demonstration services, barbers, prices, and hours based on the supplied images.
- Booking records are stored in SQLite; do not use browser storage as the source of truth.
- The barber agenda requires the demonstration password `barbeiro123`; this is a local prototype gate, not production authentication.
- Do not seed fictitious customer appointments; the agenda starts empty.
- Keep the project local. Public deployment and production access control are out of scope.
- Do not add or run automated tests in this task.

## Review Focus

- **Concurrent bookings for one barber and overlapping times:** the API must reject the later booking and return an actionable conflict response.
- **Cancelled appointments:** their time becomes available again; active appointments still block overlapping slots.
- **Business boundaries:** Sunday is closed; starts use 09:00, 10:00, 11:00, and 14:00–19:00; services cannot cross the 12:00–14:00 break or the 20:00 closing time.
- **Agenda access:** a wrong password or missing session must not reveal appointment records through the agenda API.
- **Booking and agenda states:** empty agenda, failed submission, unavailable slot, and narrow mobile layout remain clear and usable.

---

### Task 1: Project scaffold and shared catalog

**Files:**
- Create: `package.json`, `index.html`, `vite.config.ts`, `tsconfig.json`, `tsconfig.app.json`, `tsconfig.server.json`, `.gitignore`, `.env.example`
- Create: `src/shared/types.ts`, `src/shared/catalog.ts`
- Create: `server/index.ts`, `server/database.ts`, `server/schema.sql`, `server/seed.ts`
- Create: `README.md`

**Interfaces:**
- `src/shared/types.ts` exports `Service`, `Barber`, `AppointmentStatus`, `Appointment`, `AvailabilitySlot`, and API request/response types.
- `src/shared/catalog.ts` exports the demonstration services, barbers, business hours, and allowed start times.
- `server/database.ts` exports `openDatabase(path?: string): Database` and `initializeDatabase(db: Database): void`.

- [ ] Scaffold the React/Vite client and TypeScript/Express server with `dev`, `build`, and `start` scripts; proxy `/api` through Vite to the local server.
- [ ] Define SQLite tables for services, barbers, and appointments, including foreign keys, status constraints, duration/price snapshots, and indexes for date/barber lookups.
- [ ] Seed the five reference services and four reference barbers, but no customer appointments.
- [ ] Ignore `.env`, `node_modules`, build output, and SQLite database/WAL files; document setup, local run commands, and the demonstration password in `README.md`.
- [ ] Review that the schema, seed values, and shared TypeScript types agree.

### Task 2: Booking rules and customer API

**Files:**
- Create: `server/booking.ts`
- Create: `server/routes/catalog.ts`, `server/routes/booking.ts`
- Modify: `server/index.ts`

**Interfaces:**
- `getAvailability(db: Database, input: { date: string; barberId: string; serviceId: string }): AvailabilitySlot[]`
- `createAppointment(db: Database, input: CreateAppointmentInput): AppointmentConfirmation`
- `GET /api/services` returns `Service[]`.
- `GET /api/barbers` returns `Barber[]`.
- `GET /api/availability?date=YYYY-MM-DD&barberId=...&serviceId=...` returns valid available start times.
- `POST /api/appointments` accepts `{ serviceId, barberId, date, time, customerName, customerPhone }` and returns the persisted confirmation; invalid or newly occupied slots return a structured error, using HTTP 409 for a conflict.

- [ ] Implement availability using the design's operating windows and service duration. Reject past dates, Sundays, invalid times, times crossing the midday break, and times ending after 20:00.
- [ ] Implement appointment validation, price/duration snapshots, and overlap checks inside one SQLite transaction so two bookings cannot claim the same barber time.
- [ ] Register catalog, availability, and booking routes; return validation and conflict errors in a consistent JSON shape.
- [ ] Review that a cancelled booking no longer blocks availability and active overlapping bookings do.

### Task 3: Barber demo session and agenda API

**Files:**
- Create: `server/auth.ts`
- Create: `server/routes/barber.ts`
- Modify: `server/index.ts`

**Interfaces:**
- `POST /api/barber/session` accepts `{ password }`, sets an HttpOnly, SameSite cookie on success, and returns `{ authenticated: true }`.
- `GET /api/barber/session` returns `{ authenticated: boolean }`.
- `DELETE /api/barber/session` ends the in-memory demonstration session.
- `GET /api/appointments?date=YYYY-MM-DD&status=...` returns `Appointment[]` only for an authenticated demo session.
- `PATCH /api/appointments/:id/status` accepts `{ status: AppointmentStatus }` only for an authenticated demo session.

- [ ] Implement the demonstration password from `DEMO_ADMIN_PASSWORD`, with `barbeiro123` as the local fallback, plus expiring in-memory sessions and logout.
- [ ] Protect agenda list and status-update routes with the session middleware; do not expose appointment records to unauthenticated requests.
- [ ] Implement date/status filters and status changes (`pending`, `confirmed`, `completed`, `cancelled`) in the API.
- [ ] Review session expiry, wrong-password behavior, and cancelled-slot availability handling.

### Task 4: Landing page and booking flow

**Files:**
- Create: `src/main.tsx`, `src/App.tsx`
- Create: `src/landing/LandingPage.tsx`, `src/landing/BookingWizard.tsx`, `src/landing/BookingConfirmation.tsx`
- Create: `src/components/SiteHeader.tsx`, `src/components/SiteFooter.tsx`
- Create: `src/styles/tokens.css`, `src/styles/global.css`, `src/styles/landing.css`, `src/styles/booking.css`
- Create: `public/images/barber-hero.png` (original/generated or otherwise suitable barber image)

**Interfaces:**
- `BookingWizard` loads `Service[]` and `Barber[]`, requests availability when service/barber/date changes, and submits `CreateAppointmentInput` to `POST /api/appointments`.
- `BookingConfirmation` displays the returned appointment ID, service, barber, date, time, and price.

- [ ] Build the landing page's hero, service-benefit strip, service overview, barber selection, hours, booking stepper, and footer in the references' near-black, warm-gold, premium style; keep all copy in Portuguese and omit unverified ratings, testimonials, and membership claims.
- [ ] Implement service, barber, date/time, and customer contact steps with required-field feedback and a final review before submitting.
- [ ] Refresh availability after selections and after a 409 response; show a clear unavailable-slot message without losing the other booking details.
- [ ] Show the persisted confirmation after success and provide a path to start another booking.
- [ ] Review keyboard focus, readable contrast, and the booking sequence at desktop and narrow widths.

### Task 5: Barber login and appointments page

**Files:**
- Create: `src/barber/BarberPage.tsx`, `src/barber/BarberLogin.tsx`, `src/barber/AppointmentAgenda.tsx`, `src/barber/AppointmentRow.tsx`
- Create: `src/styles/barber.css`
- Modify: `src/App.tsx`

**Interfaces:**
- `BarberPage` handles path `/barbeiro/agendamentos`, checks `GET /api/barber/session`, and renders either `BarberLogin` or `AppointmentAgenda`.
- `AppointmentAgenda` loads the selected day's appointments and status filters; updates a row through `PATCH /api/appointments/:id/status`.

- [ ] Add a password form with a visible local demo credential hint and session logout.
- [ ] Show appointments in time order with date/status filters and customer, service, barber, time, price, and status details.
- [ ] Add confirm, complete, and cancel actions; refresh the agenda after each successful update.
- [ ] Design an intentional empty state for days with no appointments, without adding fictitious customer records.
- [ ] Review that direct navigation and refresh preserve the login gate and the agenda remains usable on mobile.

### Task 6: Visual consistency and handoff

**Files:**
- Modify: `README.md`
- Modify as needed: `src/styles/*.css`, `public/images/barber-hero.png`

- [ ] Align header, booking, confirmation, and barber agenda typography, spacing, borders, and gold/black palette with the supplied reference.
- [ ] Review responsive behavior in one desktop and one narrow layout pass; fix material overflow, clipped controls, and unreadable text found in that pass.
- [ ] Ensure the README gives the exact install, environment, database, and local run instructions, and labels the password gate and seed data as demonstration-only.
- [ ] Do not publish the prototype or introduce production deployment configuration.
