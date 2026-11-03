# Product

<!-- impeccable:product-schema 1 -->

## Platform

web

## Stack

React, a TypeScript backend, and SQLite, as requested. For this prototype, Vite, Express, and better-sqlite3 are the builder's implementation choices.

## Users

- Customers choosing a barbershop service and appointment time.
- The barber reviewing and updating the appointment agenda.

## Product Purpose

Provide a barbershop landing page that lets a customer book a service and a focused page where the barber can review those appointments.

## Operating Context

This is a semi-functional prototype in a new, empty project. No deployment target has been specified. A booking must be stored in SQLite so it is available to the barber's appointments page.

## Capabilities and Constraints

- Keep the scope to the landing and booking flow, plus the barber's appointments page.
- Use demonstration business details and appointment data that can be changed later.
- The barber page has a demonstration password gate for local evaluation.
- A production deployment, production authentication, payments, and external messaging have not been requested.

## Brand Commitments

Use the two supplied dark, gold-accented barbershop pages as visual references for this surface. The names, ratings, claims, and business details shown in those images are examples, not confirmed real business facts.

## Evidence on Hand

The supplied images show the desired landing-page and booking-flow style, and contain example services, prices, staff names, and hours. They do not establish the user's actual business details.

## Product Principles

- Make booking clear and quick for a customer.
- Show only appointment times that remain available.
- Give the barber a concise, usable view of the agenda.
