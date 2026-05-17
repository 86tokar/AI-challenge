# Task 2 — Gatherly-Go Test Report

**App URL:** https://gatherly-go.lovable.app/  
**Test Date:** 2026-05-10  
**Method:** Unauthenticated HTTP page inspection of all discoverable public routes  
**Current date used as baseline:** May 10, 2026  

Legend: ✅ Pass · ⚠️ Partial / Minor issue · ❌ Fail / Missing · ❓ Cannot verify without auth

---

## Seeded Data

| Item | Value | Status |
|------|-------|--------|
| Host | Brooklin Book Club (`/hosts/brooklin-book-club-vdtk`) | ✅ |
| Upcoming event 1 | "123" — May 11, 2026 (`/events/c8123cae7ae4`) | ⚠️ Placeholder title, no description, no cover image |
| Upcoming event 2 | "Sunset Poetry Reading at Riverside Park" — May 24, 2026 (`/events/sunset-poetry-reading`) | ✅ Well-formed event |
| Past event | None — both seeded events are future-dated | ❌ Hard requirement miss |

> **Note on past event:** The homepage "Include past" toggle reveals `sunset-poetry-reading` as an additional result, but that event is dated May 24, 2026 (future). Neither seeded event is a past event. Both event pages still show "RSVP — it is free" with no "Ended" label.

---

## 1. Core Publishing and Hosting

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 1.1 | Self-serve Host registration | ✅ | `/become-host` redirects unauthenticated users to login. Signup at `/signup` collects Name, Email, Password. |
| 1.2 | Host profile: name, logo, short bio, contact email, public Host page | ⚠️ | `/hosts/brooklin-book-club-vdtk` shows name ✅, email ✅. **Logo absent** ❌. Bio field only repeats the host name ❌. Public Host page exists ✅. |
| 1.3 | Event: title, description, start/end date+time+timezone, venue or link, capacity, cover image | ⚠️ | `sunset-poetry-reading`: title ✅, description ✅, dates ✅, timezone (Europe/Warsaw) ✅, physical venue address ✅, capacity (60 seats) ✅, cover image (Unsplash) ✅. `c8123cae7ae4`: title is "123" ❌, no description ❌, no cover image ❌, online link ✅, capacity (50 seats) ✅. |
| 1.4 | Public / Unlisted visibility modes | ❓ | Both events appear on homepage. Unlisted (link-only) cannot be verified without auth. |
| 1.5 | Draft / Published states + Publish / Unpublish / Duplicate actions | ❓ | Requires auth. Both events appear published. |
| 1.6 | Free/Paid toggle — Paid disabled with "Coming soon" tooltip | ✅ | Both event pages show "Free event · Paid tickets coming soon". |

---

## 2. Discovery and Sharing

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 2.1 | Explore page: text search, date range filter, location filter, "Include past" toggle | ⚠️ | Homepage (`/`) serves as Explore. Text search ✅, location field ✅, "Include past" toggle ✅. **No date range filter visible** ❌. `/explore` → HTTP 404 ❌. |
| 2.2 | Past events show "Ended"; RSVP hidden on past events | ❌ | No past event is seeded. "Ended" state and hidden RSVP cannot be demonstrated at all. |
| 2.3 | Event and Host pages include social preview metadata (OG / Twitter Card) | ❓ | Cannot inspect `<head>` meta tags via text fetch. Requires browser DevTools. |

---

## 3. RSVP and Tickets

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 3.1 | RSVP requires sign-in; unauthenticated users redirected and returned to event afterward | ✅ | "RSVP — it's free" button present on event pages. Auth-gated routes redirect to `/login`. Redirect param preserved in signup link (`?redirect=%2Flogin`). |
| 3.2 | Capacity enforced; over-capacity RSVPs go to waitlist | ❓ | Cannot test without auth and a nearly full event. |
| 3.3 | Confirmed attendees get QR ticket + "Add to Calendar" | ❓ | Cannot test without completing RSVP flow. |
| 3.4 | Cancel RSVP; "My Tickets" page shows upcoming tickets | ❓ | `/tickets` route exists (auth-gated) ✅. `/my-tickets` → 404 ❌. Full flow requires auth. |

---

## 4. Waitlist

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 4.1 | FIFO waitlist; auto-promote on cancellation / capacity increase | ❓ | Cannot test without auth and a full event. |
| 4.2 | Promotion visible in-app to affected attendee | ❓ | Cannot test without auth. |

---

## 5. Roles and Permissions

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 5.1 | Host and Checker roles per Host account | ❓ | Cannot verify without auth. |
| 5.2 | Invite members by role via copyable link | ❓ | Cannot verify without auth. |
| 5.3 | Host: full management; Checker: check-in page only | ❓ | Cannot verify without auth. |

---

## 6. Host Dashboard and Operations

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 6.1 | Dashboard: Upcoming/Past events with per-event stats (Going, Waitlist, Checked-in) | ❓ | `/host/dashboard` → redirects to login. Route exists ✅. Content unverifiable without auth. |
| 6.2 | CSV export: name, email, RSVP status, check-in time | ❓ | Cannot test without auth. |
| 6.3 | "My Events" page with filters (host, date range, text) + role-appropriate quick actions | ❓ | `/my-events` → redirects to login. Route exists ✅. Content unverifiable without auth. |

---

## 7. Check-in Page

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 7.1 | Check-in page: manual code entry, QR generated per ticket | ❌ | All tried routes → 404: `/events/{id}/checkin`, `/events/{id}/check-in`, `/host/events/{id}/checkin`. No check-in route is publicly discoverable. |
| 7.2 | Live counters, duplicate prevention, undo last scan | ❓ | Route not found; cannot test. |

---

## 8. Community Content and Feedback

| # | Requirement | Status | Evidence |
|---|-------------|--------|----------|
| 8.1 | Post-event feedback: 1–5 star rating + optional comment (after event ends) | ❌ | No past event seeded. Feedback UI cannot be reached or demonstrated. |
| 8.2 | Photo gallery uploads requiring Host approval before display | ⚠️ | Gallery section exists on both event pages ("No photos yet") ✅. Upload + approval flow requires auth. |
| 8.3 | Report event / report photo; review queue for reported items | ⚠️ | "Report this event" link visible on event pages ✅. `/report` → 404 ❌. Review queue requires auth to verify. |

---

## 9. Submission Artifacts

| # | Requirement | Status | Notes |
|---|-------------|--------|-------|
| 9.1 | At least one Host seeded | ✅ | Brooklin Book Club |
| 9.2 | At least one upcoming event seeded | ✅ | Two upcoming events present |
| 9.3 | At least one past event seeded | ❌ | Both seeded events are future-dated (May 11 and May 24, 2026) |
| 9.4 | Example CSV export file | ❓ | Not verifiable without auth |
| 9.5 | report.md in repository | ✅ | This file |
| 9.6 | Step-by-step README (Publish → RSVP → Ticket → Check-in) | ❓ | Not verified in this test pass |

---

## Route Inventory (Verified)

| Route | HTTP | Notes |
|-------|------|-------|
| `/` | 200 | Homepage / Explore |
| `/login` | 200 | Login form |
| `/signup` | 200 | Sign-up form |
| `/become-host` | 200 → login | Auth-gated, redirect works |
| `/hosts/{slug}` | 200 | Public host profile |
| `/events/{id}` | 200 | Public event page |
| `/tickets` | 200 → login | Auth-gated, route exists |
| `/my-events` | 200 → login | Auth-gated, route exists |
| `/host/dashboard` | 200 → login | Auth-gated, route exists |
| `/host/events` | 200 → login | Auth-gated, route exists |
| `/explore` | 404 | Missing — expected Explore URL |
| `/events` | 404 | Missing |
| `/events/{id}/checkin` | 404 | Check-in route missing |
| `/events/{id}/check-in` | 404 | Check-in route missing |
| `/host/events/{id}` | 404 | Missing |
| `/host/events/{id}/checkin` | 404 | Missing |
| `/my-tickets` | 404 | Missing (spec calls for "My Tickets" page) |
| `/admin` | 404 | Missing |
| `/report` | 404 | Missing |

---

## Summary of Failures

### Critical (Hard Requirements)

| # | Issue |
|---|-------|
| C1 | **No past event seeded.** Both events are future-dated. "Ended" state, hidden RSVP, and post-event feedback cannot be demonstrated. |
| C2 | **Check-in route does not exist.** All variants of the check-in URL return 404. This is a primary operational feature of the platform. |

### Significant Issues

| # | Issue |
|---|-------|
| S1 | **No date range filter on Explore.** Spec requires it; only text search, location, and past toggle are present. |
| S2 | **`/explore` returns 404.** The Explore page is only accessible via the root `/`. |
| S3 | **Seeded event "123" is incomplete.** Placeholder title, no description, no cover image — looks like broken test data. |
| S4 | **Host profile missing logo and bio.** Two of four required profile fields are missing or defaulted. |
| S5 | **"Sunset Poetry Reading" not listed on host's public profile.** Only the "123" event appears under Upcoming on the host page. |

### Minor Issues

| # | Issue |
|---|-------|
| M1 | `/my-tickets` → 404. The spec calls for a "My Tickets" page; the existing `/tickets` route may serve this purpose but the expected URL is broken. |
| M2 | Bio field shows the host name instead of actual bio text. Looks like a default/placeholder value. |

---

## What Works Well

- Clean homepage with text search, location filter, and "Include past" toggle
- `sunset-poetry-reading` is a production-quality seeded event: real title, description, physical venue address, cover image, capacity, timezone
- "Free event · Paid tickets coming soon" messaging on both event pages
- "Report this event" link visible on event pages
- Gallery section present on event pages
- Auth protection correctly applied to all management routes with redirect param preserved
- Login and signup pages with correct minimal field sets
- Two distinct event routing patterns (UUID-based and slug-based)

---

## Recommended Fixes Before Final Submission

1. **Seed a past event** (any date before May 10, 2026) and verify it renders "Ended" with RSVP hidden.
2. **Implement and fix the check-in route** — determine the correct URL pattern and ensure it is accessible to Checker-role users.
3. **Fix event "123"** — give it a real title, description, and cover image, or replace it with a well-formed seeded event.
4. **Add "Sunset Poetry Reading" to the host's public profile** event listing.
5. **Add a date range filter** to the Explore page.
6. **Add `/explore` as a route** pointing to the homepage or its own page.
7. **Complete the host profile** — add logo upload and bio text.
8. **Verify `/my-tickets`** is a working route (or confirm `/tickets` is the canonical path).
9. **Perform a full authenticated walkthrough** covering: RSVP → QR ticket → Add to Calendar → Cancel → Waitlist promotion → Check-in → CSV export → Post-event feedback → Gallery upload + approval → Report → Review queue.
