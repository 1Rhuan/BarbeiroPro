# Barbearia Booking Prototype — Design

## Intent

Build a semi-functional barbershop website based on the supplied reference. Customers can book an appointment from the landing page; the barber can see those bookings on a separate agenda page. The project is a local prototype, not a finished booking service for public use.

## Goals

- Create a polished, responsive landing page in the reference's dark and gold premium style.
- Let a customer choose a service, barber, date, and available time, then provide a name and phone number to book.
- Save bookings in a local SQLite database through a small TypeScript API.
- Let the barber see bookings by date and update their status.

## Non-goals

- Service or staff management screens, customer accounts, payment, notifications, calendar sync, or public hosting.
- Production authentication, multi-location support, or production data migration.
- Claiming the supplied demonstration brand details belong to the user's actual business.

## Anti-goals

- A static mockup where a submitted booking disappears or is visible only in browser storage.
- A broad salon-management dashboard beyond the requested appointment agenda.
- Presenting synthetic business details, staff, or bookings as verified real information.

## Constraints and confirmed choices

- The project directory was empty when work began.
- The user requested React, SQLite, and allowed a simple TypeScript backend.
- The user wants only the landing page and a page where the barber sees appointments.
- The user delegated the example values to the builder. Use clearly demonstrative defaults based on the reference: “Barbearia Exclusiva”; services Corte de Cabelo (R$ 50), Barba (R$ 40), Corte + Barba (R$ 80), Sobrancelha (R$ 20), and Corte + Barba + Sobrancelha (R$ 100); barbers Lucas Ferreira, Gabriel Santos, Rafael Lima, and Matheus Alves; Monday–Saturday, 09:00–20:00, closed Sunday. Keep these values in seed/config data so they are easy to replace.
- The barber's page must ask for a demonstration password. Use a clear local demo credential and label it as demonstration access; it is not production authentication.
- No actual phone, address, brand assets, administrator credentials, or hosting target were supplied.
- The two reference images set the visual direction: near-black surfaces, warm gold accents, editorial typography, strong barber imagery, a prominent booking callout, and a visible multi-step booking flow. Keep the page copy in Portuguese and the app to the two requested pages. Do not copy unverified ratings, testimonials, membership claims, or invented business facts.

## Approaches

1. **Static frontend with browser storage.** Rejected because bookings would not be written to the requested SQLite backend.
2. **Local React app with a TypeScript API and SQLite.** Selected. It keeps the prototype simple, stores bookings across restarts, and connects the customer flow to the barber agenda without setting up hosted infrastructure.
3. **Hosted app with remote database and production access controls.** Deferred because the user asked for a semi-functional prototype and did not choose a deployment target.

## Surface and visitor flow

### Landing and booking (`/`)

- Start with a strong image-led hero, concise booking call to action, and the premium dark/gold visual language in both references.
- Keep the page focused: service examples, barber choices, a short service-benefit strip, booking flow, practical business hours, and a compact footer.
- Booking steps: choose service; choose barber; choose a date and available time; enter name and phone; review and submit; show an explicit confirmation with the appointment details.
- Use the reference's available start times: 09:00, 10:00, 11:00, then 14:00 through 19:00. Services use demonstration durations (30 minutes for haircut and beard; 60 for haircut plus beard; 15 for eyebrow; 75 for the full combo). Hide a start time when the service would finish outside business hours or conflict with an existing booking.
- On narrow screens, stack the booking steps into a readable sequence with large touch targets.
- Validate required fields and reject a slot that has become unavailable since it was displayed.

### Barber appointments (`/barbeiro/agendamentos`)

- Require a simple demonstration password before showing the agenda. For local evaluation, display the demo credential on the login screen and document it in the run guide; this gate is not production authentication.
- Show the day's appointments in time order, with a date filter and clear status labels.
- Each entry includes customer name and phone, service, barber, time, and price.
- Allow the barber to mark an appointment confirmed, completed, or cancelled. Cancelling releases the time slot.
- Include a useful empty state when the selected day has no bookings.
- Do not seed fake customer appointments. The agenda starts empty and shows real bookings created through the prototype flow.
- The prototype is intended to run locally; production access control is outside this scope and must be designed before public deployment.

## Data and behavior

- `services`: seeded demonstration service name, duration, and price.
- `barbers`: seeded demonstration names and optional portrait references.
- `appointments`: service, barber, customer name and phone, start time, duration, price snapshot, status, and created timestamp.
- The API supplies service and barber choices, available slots, appointment creation, the barber's filtered agenda, and status updates.
- Reject booking creation when required values are invalid or when the chosen barber has an overlapping non-cancelled appointment. Perform the conflict check and insert as one database transaction.
- Store the database in a local ignored data directory; include a seed path for the demonstration catalog and a short run guide.

## Technical direction

- React and TypeScript frontend, using Vite as the lightweight development/build tool.
- TypeScript backend using Express and SQLite via `better-sqlite3`.
- Keep database queries and booking rules server-side; use a small, documented REST API.
- Keep brand and catalog examples in seed/config data rather than scattering them through components.
- Use local development only until the user selects a hosting target. Do not represent the prototype as production-ready.

## Left to the builder

- Component boundaries, route organization, SQLite schema details, exact seed barber names and durations, responsive breakpoints, and the choice of suitable image assets.
- Add a basic README with setup and run commands. Do not add unrelated dashboard features.
