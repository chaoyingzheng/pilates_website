# Still Point Pilates: Design & Build Plan

> Status: **design review**. The clickable prototype (`prototype/index.html`) is ready for feedback. The production build (Next.js) has not started yet.
> "Still Point Pilates" is a placeholder studio name.

## 1. What we're building

A simple studio website centered on **group class scheduling and booking**.

- **Members** browse the weekly schedule, create an account, book a spot, join a waitlist when a class is full, and cancel.
- **Studio admins** manage the schedule (including recurring classes), class types, instructors, members, rosters and attendance, booking rules, and see simple reports.

Decisions so far:

| Topic | Decision |
|---|---|
| Session types | Group classes only (capacity limit + waitlist). No 1:1 private sessions for now |
| Payments | None for now. Credits/packs or pay-per-class can be added later (Stripe) |
| Stack | Next.js (TypeScript) + PostgreSQL |
| Hosting | Google Cloud Run + Neon Postgres (free tiers) |
| Theme | **Sand & Clay** (see §3) |

## 2. Design direction

Researched references: Luminance Pilates, JETSET Pilates (schedule page), LG Movement (group classes page).

**Overall feel:** minimal, boutique and editorial. Lots of white space, real studio photography, few navigation items, and one clear "Book a class" button in the header.

**Typography**
- An elegant serif for headings, with one word set in italic and in the accent color as a signature, e.g. *"Book your* ***Pilates*** *class"*, *"Find your* ***center***".
- A clean sans for body text and UI.
- Small all-caps "eyebrow" labels with wide letter-spacing above each heading, e.g. `— SCHEDULE`, `— THE STUDIO`.
- Text links in caps ending in an arrow: `FULL SCHEDULE →`.

**Schedule UX** (modeled on JETSET's list, the most informative of the three)
- A week strip of 7 day tabs, plus *Prev week / Next week*. Today is marked with a dot.
- Filters for class type, instructor and level.
- Each class is **one row**: start time and duration · class name + level tag · instructor and room · spots left (shown only when 3 or fewer remain, or "Full · N on waitlist") · action button.
- Action states: **Reserve** · **Waitlist** · **Booked ✓** · **Waitlist #2** · **Opens Dec 3** (outside the booking window) · Ended · Cancelled.
- Clicking a row opens a detail panel with the description, times, open spots and cancel/calendar actions.
- No calendar grid on the public site. The grid view is for admins only.

**Class levels & policies** (inspired by LG Movement)
- The Classes page explains the levels (Foundations, Level 1, Level 2, All levels) and has a "Know before you book" section (arrival, grip socks, booking window, cancellation cutoff, how the waitlist works).
- Later: some levels can be marked "approval required", and admins approve members for them.

## 3. Theme: Sand & Clay (chosen)

All visual decisions live in a small set of **design tokens** (CSS variables). In the production app they become the shadcn/ui theme in `src/app/globals.css` plus fonts loaded with `next/font`, so the whole look changes in one place.

| Token | Value | Used for |
|---|---|---|
| `--bg` | `#FAF7F2` | Page background (warm cream) |
| `--surface` | `#FFFFFF` | Cards, inputs, panels |
| `--soft` | `#F1E9DF` | Subtle fills, selected admin nav item, image placeholders |
| `--text` | `#2B2724` | Main text (warm charcoal) |
| `--muted` | `#7A716A` | Secondary text, labels |
| `--line` | `#E7DFD5` | Borders, dividers |
| `--accent` | `#B4694A` | Clay. Primary buttons, italic accent words, highlights |
| `--accent-ink` | `#FFFFFF` | Text on accent buttons |
| `--ok` / `--warn` / `--danger` | `#4E7A57` / `#B07A2A` / `#A8463B` | Booked / few spots or waitlist / full, cancelled or destructive |

| Type | Font | Style |
|---|---|---|
| Headings | **Cormorant Garamond** | Weight 500; italic accent word in clay |
| Body & UI | **Inter** | 15px base; labels in all caps with 0.12–0.22em letter-spacing |

| Shape | Value |
|---|---|
| Cards, inputs, tags | 4px radius, 1px `--line` border, no heavy shadows |
| Buttons | Pill (999px radius), all caps, solid clay (primary), outline (secondary), red outline (destructive) |

Class types each get their own color (used on admin calendar blocks and class cards). The prototype uses sand `#B9A48A`, clay `#B4694A`, sage `#6F8466` and slate `#7C8BA1`.

## 4. Pages

**Public**
- `/`: hero, "Up next" classes, class types, instructors
- `/schedule`: **the main page** (see the schedule UX in §2)
- `/classes`: class types, levels, studio policies
- `/instructors`, `/contact`
- `/login`, `/register`, `/forgot-password`

**Member** (`/account`)
- Upcoming bookings (with cancel and waitlist position), add to calendar (.ics)
- Class history (attended / no-show / cancelled)
- Profile and password

**Admin** (`/admin`, admin role only)
- **Dashboard**: classes today with fill bars, average fill over the next 7 days, bookings in the last 7 days, waitlist count, recent activity, sent-email log
- **Schedule**: week calendar; add one class or a **weekly series** (pick weekdays + end date); edit, cancel or duplicate a class
- **Class roster**: booked members, attendance check-in (Attended / No-show, opens 30 min before class), remove a member, waitlist, add a member (optionally over capacity)
- **Classes & instructors**: add/edit class types (name, level, length, default capacity, color, description); add/edit/deactivate instructors
- **Members**: search, booking history, make admin, deactivate (cancels their upcoming bookings)
- **Reports**: classes held, average fill, attendance rate and no-shows per class type and per instructor; export bookings as CSV
- **Settings**: studio details, booking window, cancellation cutoff, max upcoming bookings per member

## 5. Booking rules

| Rule | Default | Behavior |
|---|---|---|
| Booking window | 14 days | Classes further out show "Opens <date>" |
| Cancellation cutoff | 12 hours | Members can't cancel a confirmed spot online inside the cutoff (admins can). Leaving a waitlist is always allowed |
| Max upcoming bookings | 8 per member | Includes waitlist spots |
| Full class | n/a | New bookings go on the waitlist in order |
| Waitlist promotion | Automatic | When a spot opens (cancellation, admin removal, capacity increase), the first person on the waitlist is booked and emailed |
| Admin cancels a class | n/a | All bookings are cancelled and everyone is emailed |
| Class time/instructor change | n/a | Booked members are emailed |

In production, booking runs inside a database transaction so two people can't take the last spot at the same time.

## 6. Technical plan (summary)

- **Next.js 15** (App Router, TypeScript, Server Actions), **Tailwind CSS + shadcn/ui**, **zod**
- **Prisma** with SQLite in local dev and **PostgreSQL** in production
- **Auth.js**: email and password, roles `MEMBER` / `ADMIN`
- **Email**: a pluggable sender (console in dev, Resend in production) for confirmations, waitlist promotions and password resets
- **Data model**: User, Instructor, ClassType, ClassSession, RecurringSeries, Booking, StudioSettings, PasswordResetToken
- **Tests**: Vitest for booking rules; Playwright for end-to-end flows
- **Hosting**: Docker image → **Google Cloud Run** (scales to zero, so it fits the free tier) + **Neon** Postgres; GitHub Actions deploy on push to `main`; secrets in Secret Manager; a budget alert set as a safety net. All config comes from environment variables, so it can move to AWS or Azure later.

## 7. Milestones

0. ✅ Clickable prototype → **design review (now)** → update this plan
1. Project setup: Next.js, Tailwind/shadcn with Sand & Clay tokens, Prisma schema, seed data
2. Accounts: register, login, password reset, admin-only routes
3. Public schedule + booking / waitlist / cancellation rules + tests
4. Member account area
5. Admin: schedule, weekly series, roster and attendance
6. Admin: class types, instructors, members, settings
7. Admin dashboard, reports, emails, calendar files
8. Polish: public pages, mobile layout, real photos and copy
9. Deploy to Cloud Run + Neon

## 8. Open questions for reviewers

- Studio name, logo, real photos and copy
- Are the default booking rules right (14-day window, 12-hour cutoff, 8 bookings max)?
- Should late cancellations or no-shows have a penalty (e.g. a fee or losing booking rights for a while)?
- Do we need "approval required" levels at launch?
- Is payment (class packs / memberships) needed for launch, or later?
