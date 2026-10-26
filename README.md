# Tutor Booking System

Booking, matching and scheduling of tutor sessions for a tutoring center, built with
**HTML/JSX, JavaScript, CSS, React and Next.js** (App Router). No other libraries are needed.

## Run it

```bash
npm install
cp .env.example .env.local     # then edit SESSION_SECRET
npm run seed                   # creates the admin account in the database
npm run dev                    # open http://localhost:3000
npm test                       # runs the automated tests
```

The admin cannot register on the website. `npm run seed` creates it directly in the database
(login comes from `ADMIN_EMAIL`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`).

## How it works in one minute

1. A **tutor** applies, fills the profile (subjects, free times, how many tutees, photo, resume). The **admin** approves and sets the pay per hour.
2. A **tutee** registers, fills the profile, then uses **Find Match** (category, grade, one or more tutors, each with a subject and weekly schedule). The system calculates the hourly rate live. Or they book an **initial assessment** from the **Available Slots** page.
3. The **admin** opens **Pending Requests**, clicks *Find top tutors*, and confirms. The match is added to everyone's **Schedule** automatically and both people are notified.
4. The admin marks **attendance**, uploads **materials**, rates tutors (**Feedback**), and runs **Payroll**. Contracts are ended with *Terminate*.

### Matching rules (`lib/matching.js`)

A tutor is offered only if ALL of these are true: approved, teaches the subject, not on leave today, has room for one more tutee, is free at every requested time, and has no other class at those times.
Tutors who pass are ranked 0-100: ratings (60 pts), spare capacity (25 pts), lighter workload (15 pts).
The admin's confirm click re-checks everything, so two admins (or two picks) can never double-book a tutor.

## Requirements checklist

| Requirement | Where |
|---|---|
| Homepage (hiring, FAQs, contacts, info), register as tutee / apply as tutor | `app/page.js`, `app/register/*` |
| Login with email or username | `app/login/page.js`, `lib/services/users.js` |
| Tutee and tutor profiles (tutor chooses how many tutees) | `app/profile/page.js` |
| Find Match with multiple tutors, live rate | `app/find-match/page.js`, `lib/rates.js` |
| Recommend top tutors, pending requests | `app/requests/page.js`, `lib/matching.js` |
| Chat with admin + automated birthday greeting | `app/chat/page.js`, `lib/services/birthday.js` |
| Schedule list, admin terminates contract | `app/schedule/page.js` |
| Feedback (admin) | `app/admin/feedback/page.js` |
| Activity history per role | `app/history/page.js`, `lib/services/history.js` |
| Notifications (web, email, SMS) | `lib/services/notifications.js`, `lib/delivery.js` |
| Admin dashboard (progress, attendance, scheduling, payroll) | `app/dashboard/page.js` |
| Leave form (tutors and admin) | `app/leave/page.js` |
| Payroll (admin) | `app/admin/payroll/page.js` |
| Available slots for assessment / booking | `app/available-slots/page.js` |
| Learning materials upload, protected | `app/materials/page.js`, `lib/services/files.js` |
| Three categories | `lib/catalog.js` |
| Purchasing page | `app/shop/page.js` |
| Only admin sees all records | `app/admin/people/page.js` |
| Admin account created in the database | `scripts/seed-admin.js` |

## Folder map

```
app/            pages (what users see) and app/api (the server routes)
components/     small reusable pieces: NavBar, SlotPicker, Field, Notice...
lib/            the logic: catalog, time, rates, matching (pure, tested)
lib/services/   one file per feature: requests, bookings, payroll... (talk to the database)
scripts/        seed-admin.js and make-function-list.js
tests/          automated tests
docs/FUNCTIONS.md   every function with a one-line explanation (run `npm run docs` to refresh)
```

Reading order for a new developer: `lib/catalog.js` -> `lib/time.js` -> `lib/matching.js` -> `lib/services/bookings.js` -> one API route -> one page.

**Code style rule:** every function has a one-line `/** comment */` above it. `npm run docs` reads those to rebuild `docs/FUNCTIONS.md`.

## Things to set before real use

- **Prices** in `lib/catalog.js` (`baseRate`, `HIGH_SCHOOL_EXTRA`, `ASSESSMENT_FEE`, shop prices) are placeholders.
- **Contact details** in `lib/siteInfo.js` were typed from the brochure photo. Please verify them.
- **Email and SMS** only print to the server console (`lib/delivery.js`). Plug in nodemailer / an SMS provider there.
- **Database** is one JSON file (`data/db.json`). Fine for learning and a small center; for production replace the 5 functions in `lib/db.js` with a real database.
- **Learning materials** are stored outside `public/`, served only through `/api/files/<id>` after a permission check, with no-cache headers. No website can stop a person from photographing or re-sharing a file they are allowed to open.
- **Not included yet:** forgot-password, login rate limiting, online payment.
