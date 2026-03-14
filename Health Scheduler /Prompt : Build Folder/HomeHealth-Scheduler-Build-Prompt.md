# HomeHealth Scheduler — Build Prompt

## Part 1 — Application Overview

HomeHealth Scheduler is a web application with two distinct user experiences on a shared codebase:

- **Admin Portal** — A single administrator manages agencies, employees, billable scheduling, conflict detection, and all user permissions.
- **Employee Portal** — Caregivers view their own schedules, manage personal sub-calendars, submit PTO and schedule requests, and view shared team calendars.

Both portals share the same URL. After login, the app routes each user to the correct dashboard based on their role. There is no separate admin subdomain or separate employee app.

The application serves two parallel purposes:

- **Billable scheduling** — The core operational system where the admin manages agencies, assigns employees, tracks billable hours, and enforces conflict rules.
- **Personal calendaring** — A supplementary layer where both admin and employees maintain personal sub-calendars for non-billable events, with visibility controls the admin governs.

These two systems share the same calendar UI but operate on separate data. Sub-calendar events are never billable and never trigger conflict detection. Billable shifts are never editable by employees.

---

## Part 2 — Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.11+ with Flask |
| Database | PostgreSQL with SQLAlchemy ORM + Flask-SQLAlchemy |
| Database Migrations | Flask-Migrate (Alembic) |
| Frontend | React + TypeScript + JavaScript + HTML5 + CSS |
| Frontend Build | Vite |
| Calendar Library | FullCalendar React — monthly, weekly, and daily views with event overlays and drag-and-drop |
| Authentication | Replit Auth — OpenID Connect (OIDC) via `authlib`. No custom login forms or passwords. All identity is provided by Replit. |
| Session Storage | Server-side sessions stored in Replit PostgreSQL via `flask-session` + `SQLAlchemySessionInterface` |
| Database | Replit native PostgreSQL — connection string read from the `DATABASE_URL` environment variable automatically provided by Replit |
| API Format | JSON REST API served by Flask Blueprints, consumed by the React frontend |
| Icons | Lucide React (`lucide-react`) |
| Fonts | Inter + JetBrains Mono via Google Fonts CDN |
| Responsive Design | Mobile-first CSS with custom breakpoints, fluid layouts, and touch-friendly interactions |

### Python Dependencies (`requirements.txt`)
```
flask
flask-sqlalchemy
flask-migrate
flask-session
flask-cors
authlib
requests
psycopg2-binary
python-dotenv
```

### Frontend Dependencies
```
react
react-dom
typescript
vite
@vitejs/plugin-react
@fullcalendar/react
@fullcalendar/daygrid
@fullcalendar/timegrid
@fullcalendar/interaction
lucide-react
```

---

## Part 3 — Project File Structure

```
homehealth-scheduler/
├── backend/
│   ├── app/
│   │   ├── __init__.py              # Flask app factory
│   │   ├── extensions.py            # db, oauth, session, migrate instances
│   │   ├── config.py                # Config classes (Development, Production)
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── user.py
│   │   │   ├── employee.py
│   │   │   ├── agency.py
│   │   │   ├── assignment.py
│   │   │   ├── shift.py
│   │   │   ├── weekly_hour_allocation.py
│   │   │   ├── pto_request.py
│   │   │   ├── schedule_request.py
│   │   │   ├── access_code.py
│   │   │   ├── sub_calendar.py
│   │   │   ├── sub_calendar_event.py
│   │   │   └── calendar_visibility_grant.py
│   │   ├── blueprints/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py              # /api/auth/*
│   │   │   ├── admin/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── agencies.py
│   │   │   │   ├── employees.py
│   │   │   │   ├── assignments.py
│   │   │   │   ├── shifts.py
│   │   │   │   ├── pto_requests.py
│   │   │   │   ├── schedule_requests.py
│   │   │   │   ├── access_codes.py
│   │   │   │   └── calendar_permissions.py
│   │   │   ├── employee/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── shifts.py
│   │   │   │   ├── pto_requests.py
│   │   │   │   └── schedule_requests.py
│   │   │   ├── calendar.py          # /api/calendar/events
│   │   │   └── sub_calendars.py     # /api/sub-calendars/*
│   │   ├── services/
│   │   │   ├── conflict_service.py  # All conflict detection logic
│   │   │   ├── shift_service.py     # Shift CRUD + hour calculations
│   │   │   ├── pto_service.py       # PTO blocking logic
│   │   │   ├── calendar_service.py  # Event aggregation + role filtering
│   │   │   └── access_code_service.py
│   │   └── utils/
│   │       ├── time_utils.py        # Time parsing, 15-min snapping, overlap checks
│   │       ├── decorators.py        # @require_auth, @require_admin, @require_employee
│   │       └── auth_utils.py        # OIDC user upsert, session helpers, admin check
│   ├── migrations/                  # Flask-Migrate / Alembic migration files
│   ├── seed.py                      # Database seed script
│   ├── run.py                       # Entry point: flask run
│   └── requirements.txt
│
├── frontend/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── package.json
│   └── src/
│       ├── main.tsx
│       ├── App.tsx
│       ├── api/
│       │   ├── client.ts            # Base fetch wrapper, handles 401/409 etc.
│       │   ├── auth.ts
│       │   ├── agencies.ts
│       │   ├── employees.ts
│       │   ├── shifts.ts
│       │   ├── ptoRequests.ts
│       │   ├── scheduleRequests.ts
│       │   ├── subCalendars.ts
│       │   └── calendar.ts
│       ├── components/
│       │   ├── calendar/
│       │   │   ├── ShiftEvent.tsx
│       │   │   ├── PtoMarker.tsx
│       │   │   ├── AgencyLegend.tsx
│       │   │   └── SubCalendarSidebar.tsx
│       │   ├── shifts/
│       │   │   ├── ShiftCard.tsx
│       │   │   ├── ShiftFormModal.tsx
│       │   │   └── ConflictBanner.tsx
│       │   ├── agencies/
│       │   │   ├── AgencyBadge.tsx
│       │   │   └── AgencyForm.tsx
│       │   ├── employees/
│       │   │   ├── EmployeeRow.tsx
│       │   │   ├── EmployeeForm.tsx
│       │   │   └── AccessCodeDisplay.tsx
│       │   ├── pto/
│       │   │   ├── PtoRequestForm.tsx
│       │   │   └── PtoStatusBadge.tsx
│       │   ├── sub-calendars/
│       │   │   ├── SubCalendarList.tsx
│       │   │   └── SubCalendarEventForm.tsx
│       │   └── ui/
│       │       ├── Button.tsx
│       │       ├── Modal.tsx
│       │       ├── Badge.tsx
│       │       ├── Card.tsx
│       │       └── StatusBadge.tsx
│       ├── pages/
│       │   ├── auth/
│       │   │   ├── LoginPage.tsx
│       │   │   └── RegisterPage.tsx
│       │   ├── admin/
│       │   │   ├── DashboardPage.tsx
│       │   │   ├── AgenciesPage.tsx
│       │   │   ├── EmployeesPage.tsx
│       │   │   ├── EmployeeDetailPage.tsx
│       │   │   ├── SchedulingPage.tsx
│       │   │   ├── PtoReviewPage.tsx
│       │   │   ├── ScheduleRequestsPage.tsx
│       │   │   └── CalendarPermissionsPage.tsx
│       │   └── employee/
│       │       ├── DashboardPage.tsx
│       │       ├── CalendarPage.tsx
│       │       ├── SubCalendarPage.tsx
│       │       └── RequestsPage.tsx
│       ├── hooks/
│       │   ├── useAuth.ts
│       │   ├── useShifts.ts
│       │   ├── useCalendarEvents.ts
│       │   └── useConflictCheck.ts
│       ├── context/
│       │   └── AuthContext.tsx
│       ├── constants/
│       │   ├── agencyColors.ts
│       │   └── roles.ts
│       ├── types/
│       │   └── index.ts             # All shared TypeScript types
│       └── styles/
│           ├── global.css
│           ├── variables.css        # CSS custom properties
│           ├── components.css
│           └── calendar.css
```

---

## Part 4 — Database Schema

Define all models in `backend/app/models/` using Flask-SQLAlchemy. Each model file defines the SQLAlchemy class, its columns, relationships, and a `to_dict()` method for JSON serialization.

### Models

**User** (`users` table)
- `id` — Integer, primary key
- `oauth_subject` — String(128), unique, not null — the `sub` claim from the OIDC token; permanently and uniquely identifies the authenticated account regardless of provider
- `display_name` — String(128), not null — human-readable name from the OIDC profile, updated on each login
- `profile_image_url` — String(512), nullable — avatar URL from the OIDC profile, updated on each login
- `role` — String(20), not null — `'admin'` or `'employee'`
- `is_active` — Boolean, default `True`
- `created_at` — DateTime, default `datetime.utcnow`

**EmployeeAccessCode** (`employee_access_codes` table)
- `id` — Integer, primary key
- `code` — String(64), unique, not null
- `employee_id` — Integer, FK → `employees.id`, not null
- `used_at` — DateTime, nullable
- `created_at` — DateTime, default `datetime.utcnow`

**Agency** (`agencies` table)
- `id` — Integer, primary key
- `name` — String(120), not null
- `total_weekly_hours` — Numeric(6,2), not null
- `color_hex` — String(7), not null
- `notes` — Text, nullable

**Employee** (`employees` table)
- `id` — Integer, primary key
- `user_id` — Integer, FK → `users.id`, nullable (null until self-registration)
- `full_name` — String(120), not null
- `contact_info` — Text, nullable
- `is_active` — Boolean, default `True`

**EmployeeAgencyAssignment** (`employee_agency_assignments` table)
- `id` — Integer, primary key
- `employee_id` — Integer, FK → `employees.id`, not null
- `agency_id` — Integer, FK → `agencies.id`, not null
- `weekly_hour_allocation` — Numeric(6,2), not null
- `available_days` — JSON, not null (list of strings: `['monday', 'tuesday', ...]`)
- `default_start_time` — String(5), not null (HH:MM)
- `default_end_time` — String(5), not null (HH:MM)

**WeeklyHourAllocation** (`weekly_hour_allocations` table)
- `id` — Integer, primary key
- `employee_id` — Integer, FK → `employees.id`, not null
- `agency_id` — Integer, FK → `agencies.id`, not null
- `week_start_date` — Date, not null
- `target_hours` — Numeric(6,2), not null
- `scheduled_hours` — Numeric(6,2), not null, default `0`

**Shift** (`shifts` table)
- `id` — Integer, primary key
- `employee_id` — Integer, FK → `employees.id`, not null
- `agency_id` — Integer, FK → `agencies.id`, not null
- `date` — Date, not null
- `start_time` — String(5), not null (HH:MM)
- `end_time` — String(5), not null (HH:MM)
- `billable_hours` — Numeric(5,2), not null (auto-calculated on save)
- `is_draft` — Boolean, default `False`
- `notes` — Text, nullable

**PtoRequest** (`pto_requests` table)
- `id` — Integer, primary key
- `employee_id` — Integer, FK → `employees.id`, not null
- `date` — Date, not null
- `hours_requested` — Numeric(4,2), not null (max 8)
- `reason` — Text, nullable
- `status` — String(20), not null, default `'pending'` — `'pending'` | `'approved'` | `'denied'`
- `admin_explanation` — Text, nullable
- `created_at` — DateTime, default `datetime.utcnow`

**ScheduleRequest** (`schedule_requests` table)
- `id` — Integer, primary key
- `employee_id` — Integer, FK → `employees.id`, not null
- `request_type` — String(80), not null
- `details` — Text, not null
- `status` — String(20), not null, default `'pending'` — `'pending'` | `'approved'` | `'denied'`
- `admin_explanation` — Text, nullable
- `created_at` — DateTime, default `datetime.utcnow`

**SubCalendar** (`sub_calendars` table)
- `id` — Integer, primary key
- `user_id` — Integer, FK → `users.id`, not null
- `name` — String(120), not null
- `color_hex` — String(7), not null
- `visibility` — String(20), not null, default `'public'` — `'public'` | `'private'` | `'admin_only'`
- `created_at` — DateTime, default `datetime.utcnow`

**SubCalendarEvent** (`sub_calendar_events` table)
- `id` — Integer, primary key
- `sub_calendar_id` — Integer, FK → `sub_calendars.id`, not null
- `title` — String(200), not null
- `date` — Date, not null
- `start_time` — String(5), nullable (HH:MM)
- `end_time` — String(5), nullable (HH:MM)
- `notes` — Text, nullable
- `created_at` — DateTime, default `datetime.utcnow`

**CalendarVisibilityGrant** (`calendar_visibility_grants` table)
- `id` — Integer, primary key
- `granted_to_user_id` — Integer, FK → `users.id`, not null
- `sub_calendar_id` — Integer, FK → `sub_calendars.id`, not null
- `grant_type` — String(20), not null — `'allowed'` | `'revoked'`

Enforce all foreign key constraints at the SQLAlchemy model level using `ForeignKey` and at the PostgreSQL level.

---

## Part 5 — API Design

All routes are Flask Blueprint routes registered under the `/api` prefix. Each Blueprint maps to a domain area. All endpoints accept and return JSON. Auth state is tracked via server-side Flask sessions.

### Auth (`/api/auth`)
- `GET /api/auth/login` — initiates Replit OIDC flow; redirects browser to Replit's authorization endpoint. Accepts an optional `?returnTo=` query param for post-login redirect.
- `GET /api/auth/callback` — OIDC callback handler; receives the authorization code from Replit, exchanges it for tokens via `authlib`, upserts the `User` record (creating it on first login), sets the server-side session, and redirects to the appropriate dashboard based on role.
- `POST /api/auth/logout` — clears the server-side session and redirects to the app root.
- `GET /api/auth/me` — returns the current session user object or `401` if not authenticated.
- `POST /api/auth/link-access-code` — body: `{ access_code }` — called after first OIDC login by a new user with no linked employee record. Validates the code, links the `oauth_subject` to the `employees` record, sets role to `employee`, and consumes the code.

### Admin — Access Codes (`/api/admin/access-codes`)
- `POST /api/admin/access-codes` — body: `{ employee_id }` — generates a one-time code
- `DELETE /api/admin/access-codes/<id>` — deletes an unused code

### Admin — Agencies (`/api/admin/agencies`)
- `GET /api/admin/agencies` — list all agencies
- `POST /api/admin/agencies` — create agency
- `GET /api/admin/agencies/<id>` — get agency with assignments
- `PUT /api/admin/agencies/<id>` — update agency
- `DELETE /api/admin/agencies/<id>` — delete agency

### Admin — Employees (`/api/admin/employees`)
- `GET /api/admin/employees` — list all employees
- `POST /api/admin/employees` — create employee record
- `GET /api/admin/employees/<id>` — get employee with assignments and access codes
- `PUT /api/admin/employees/<id>` — update employee
- `DELETE /api/admin/employees/<id>` — soft delete (`is_active = False`)

### Admin — Assignments (`/api/admin/assignments`)
- `POST /api/admin/assignments` — create employee-agency assignment
- `PUT /api/admin/assignments/<id>` — update assignment
- `DELETE /api/admin/assignments/<id>` — remove assignment

### Admin — Shifts (`/api/admin/shifts`)
- `GET /api/admin/shifts` — list shifts, filterable by `?start=&end=&employee_id=&agency_id=`
- `POST /api/admin/shifts` — create shift (runs conflict validation before save)
- `PUT /api/admin/shifts/<id>` — update shift (runs conflict validation before save)
- `DELETE /api/admin/shifts/<id>` — delete shift

### Admin — PTO Review (`/api/admin/pto-requests`)
- `GET /api/admin/pto-requests` — list all PTO requests, filterable by `?status=`
- `PUT /api/admin/pto-requests/<id>/review` — approve or deny, body: `{ status, admin_explanation }`

### Admin — Schedule Request Review (`/api/admin/schedule-requests`)
- `GET /api/admin/schedule-requests` — list all schedule requests, filterable by `?status=`
- `PUT /api/admin/schedule-requests/<id>/review` — approve or deny, body: `{ status, admin_explanation }`

### Admin — Calendar Permissions (`/api/admin/calendar-permissions`)
- `GET /api/admin/calendar-permissions` — list all employees and their sub-calendars with visibility states
- `PUT /api/admin/calendar-permissions/sub-calendars/<id>` — override sub-calendar visibility
- `POST /api/admin/calendar-permissions/grants` — create a visibility grant
- `DELETE /api/admin/calendar-permissions/grants/<id>` — remove a grant

### Calendar Data (`/api/calendar`)
- `GET /api/calendar/events?start=&end=` — returns billable shifts and permitted sub-calendar events for the current session user, filtered by role

### Sub-Calendars (`/api/sub-calendars`)
- `GET /api/sub-calendars` — list current user's sub-calendars
- `POST /api/sub-calendars` — create sub-calendar
- `GET /api/sub-calendars/<id>` — get sub-calendar
- `PUT /api/sub-calendars/<id>` — update sub-calendar
- `DELETE /api/sub-calendars/<id>` — delete sub-calendar
- `GET /api/sub-calendars/<id>/events` — list events
- `POST /api/sub-calendars/<id>/events` — create event
- `PUT /api/sub-calendars/<id>/events/<event_id>` — update event
- `DELETE /api/sub-calendars/<id>/events/<event_id>` — delete event

### Employee Portal (`/api/employee`)
- `GET /api/employee/shifts` — own confirmed billable shifts (read-only)
- `GET /api/employee/pto-requests` — own PTO requests
- `POST /api/employee/pto-requests` — submit PTO request
- `GET /api/employee/schedule-requests` — own schedule requests
- `POST /api/employee/schedule-requests` — submit schedule request
- `GET /api/employee/shared-calendars` — other employees' public sub-calendars the user can view

---

## Part 6 — User Roles & Access Control

Two roles exist. Every Flask route, every database query, and every React UI element must be gated by role.

### Admin Role
- The admin is identified by their OIDC subject. On first login, the system checks the incoming `oauth_subject` against the `ADMIN_OAUTH_SUBJECT` environment variable. If it matches, the user record is created with `role = 'admin'`. This value must be set in Replit's environment secrets before first run.
- There is exactly one admin. The admin account cannot be created or changed via the UI.
- Unrestricted access to the entire application.

### Employee Role
- Employees log in via Replit Auth (OIDC). On first login, if no `User` record exists for their `oauth_subject`, they are redirected to an access-code entry page.
- The employee enters the one-time access code the admin generated. The system validates it, links the `oauth_subject` to the matching `employees` record, creates a `User` with `role = 'employee'`, consumes the code (cannot be reused), and redirects to the employee dashboard.
- On subsequent logins, the OIDC callback finds the existing `User` record by `oauth_subject` and updates `display_name` and `profile_image_url` in place.
- Admin can revoke access by setting `is_active = False`, blocking login without deleting data.
- Employees see and interact with only their own data unless granted access by admin.

### Flask Decorators
Create reusable decorators in `backend/app/utils/decorators.py`:

```python
@require_auth       # checks session['user_id'] exists, returns 401 if not
@require_admin      # checks session['role'] == 'admin', returns 403 if not
@require_employee   # checks session['role'] == 'employee', returns 403 if not
```

Apply `@require_admin` to all admin blueprint routes. Apply `@require_employee` to all employee blueprint routes. Apply `@require_auth` to calendar and sub-calendar routes (accessible by both roles).

### Session & Middleware
- Flask sessions are server-side, stored in Replit's PostgreSQL via `flask-session` using `SQLAlchemySessionInterface`.
- Replit Auth (OIDC) is configured via `authlib`'s `OAuth` client, registered in `app/__init__.py`. The OIDC discovery URL is read from the `OIDC_DISCOVERY_URL` environment variable (default: `https://replit.com/.well-known/openid-configuration`). The client reads `OIDC_CLIENT_ID` and `OIDC_CLIENT_SECRET` from environment secrets.
- After a successful OIDC callback, the server stores `user_id` and `role` in the session. On every protected request, `user_id` is read from the session, the `User` is fetched from the database, and attached to `flask.g.current_user`.
- If the incoming `oauth_subject` matches `ADMIN_OAUTH_SUBJECT` and no user record exists yet, a new admin `User` is created automatically at callback time.
- If the incoming `oauth_subject` is unknown and does not match the admin env var, the user is redirected to `/link-account` to enter their access code.
- Unauthenticated requests to protected routes return `401`. Role mismatch returns `403`.

### Role Routing (React Frontend)
After login, the frontend reads the user's role from the `/api/auth/me` response and navigates accordingly:

| Role | Post-Login Route |
|---|---|
| `admin` | `/admin/dashboard` |
| `employee` | `/employee/dashboard` |

---

## Part 7 — Admin Portal Features

### 1. Agency Management
- Create, edit, and delete agencies.
- Each agency has: name, total weekly billable hours, calendar color (from the agency color palette), optional notes.
- Validation: the sum of all employee hour allocations within an agency must never exceed `total_weekly_hours`. Enforce on save in the service layer, return a `422` with a descriptive error message.

### 2. Employee Management
- Create employee records with name, contact info, and active status.
- Assign employees to one or more agencies. Each assignment specifies: weekly hour allocation, available days, default shift start/end times.
- An employee assigned to multiple agencies must never be double-booked across them.
- Generate one-time access codes from the employee detail page. Codes display once — admin copies and shares them manually.
- Revoke or reinstate employee portal access without deleting any data.

### 3. Billable Shift Scheduling
- Create, edit, and delete shifts. Each shift ties an employee to an agency for a date, start time, end time, and billable hours (auto-calculated from time range).
- Default to 15-minute increments. Minimum shift length: 2 hours. Admin can override both.
- Before saving any shift, run full conflict validation in `conflict_service.py`.
- Shifts can be marked as draft. Draft shifts appear on the calendar with reduced opacity and dashed border and do not count toward billable totals until confirmed.

### 4. Weekly Hour Tracking
- For each employee-agency pair per week: track target hours, scheduled hours, remaining hours.
- Color-coded visual indicators when over or under weekly target.
- Summary dashboard: total hours per agency this week, employees with unmet targets, active conflicts.

### 5. PTO Review
- Admin sees all pending PTO requests.
- Approve or deny; denial requires a written explanation.
- Approved PTO blocks scheduling for affected hours.
- Full-day PTO (8 hours): entire day released; other employees can cover.
- Partial PTO (<8 hours): only those hours blocked; employee schedulable outside them.
- PTO hours excluded from billable totals; do not trigger overlap warnings.

### 6. Schedule Request Review
- Admin sees all pending schedule change requests.
- Approve or deny with written explanation.
- Approved requests populate the calendar as confirmed shifts.

### 7. Calendar Permissions Management
- Dedicated page listing all employees and their sub-calendars.
- Admin can override any sub-calendar's visibility to Public, Private, or Admin Only.
- Admin can revoke a specific employee's access to a specific other employee's public calendar.
- Admin Only calendars: hidden from all employees including the calendar owner.

| Visibility State | Who Can See It |
|---|---|
| Public (default) | All logged-in employees and admin |
| Private (owner-set) | Owner and admin only |
| Admin Only | Admin only — hidden even from owner |
| Access Revoked | Admin has blocked a specific employee from a specific public calendar |

---

## Part 8 — Employee Portal Features

### Employee Dashboard
- This week's scheduled confirmed billable shifts (read-only) in a condensed list view.
- Status of all PTO and schedule requests with admin explanations.
- Quick-access list of own sub-calendars.

### Billable Shift View
- Read-only view of own confirmed billable shifts on the calendar.
- Color-coded by agency using the agency color palette.
- Draft shifts are not visible to employees.

### PTO Requests
- Submit: date, hours requested (max 8), optional reason.
- View status and admin explanation from dashboard.
- Approved PTO blocks appear on calendar as distinct markers.

### Schedule Change Requests
- Submit preferred schedule, availability change, or shift swap request with details.
- View status and admin explanation from dashboard.
- Approved requests appear as confirmed shifts on calendar.

### Sub-Calendar Management
- Create personal sub-calendars with name and color.
- Set visibility: Public (default) or Private.
- Add, edit, delete events on own sub-calendars (title, date, optional time, notes).
- View other employees' Public sub-calendars unless admin has revoked access.

| Capability | Details |
|---|---|
| View own billable shifts | Read-only. Cannot edit or delete. |
| Submit PTO requests | Submit form. Admin approves/denies with explanation. |
| Submit schedule requests | Submit form. Admin approves/denies with explanation. |
| View request status | See Pending/Approved/Denied + admin explanation. |
| Create sub-calendars | Create, rename, recolor, delete own sub-calendars. |
| Add sub-calendar events | Create, edit, delete events on own sub-calendars. |
| Set sub-calendar visibility | Toggle between Public and Private. |
| View others' calendars | View Public sub-calendars where admin has not revoked access. |

---

## Part 9 — Conflict Detection & Prevention

Conflict detection is the most critical backend logic. It lives entirely in `backend/app/services/conflict_service.py` and must run on every shift save (create and update). The frontend never performs conflict logic — it only displays what the backend returns.

### Conflict Rules
1. The same employee (`employees.id`) may not be double-booked across any agencies during overlapping hours.
2. An employee may not be scheduled during approved PTO hours (full-day or partial blocks).
3. An employee's scheduled hours for a given week may not exceed their `weekly_hour_allocation` for that agency.
4. Sub-calendar events are completely excluded from conflict detection. Only `Shift` records are evaluated.

### Conflict Response
- Return `409 Conflict` with a JSON body describing the specific rule violated, which agency, and which times.
- The React frontend blocks the save and displays the message inline near the form.

```json
{
  "error": "conflict",
  "message": "This employee is already scheduled for Agency 1 from 8:00 AM to 4:00 PM on this date.",
  "details": {
    "conflicting_shift_id": 42,
    "agency_name": "Agency 1",
    "conflict_start": "08:00",
    "conflict_end": "16:00"
  }
}
```

### Conflict Display on Calendar
- Conflicting shifts: red dashed border + warning icon (`AlertTriangle` from lucide-react).
- Draft shifts: agency color at 50% opacity, dashed border, "Draft" label.

### Multi-Agency Employee Conflict Logic
- An employee assigned to two agencies shares the same `employees.id`.
- When scheduling, query all `Shift` records for that employee across all agencies and check for any time overlap with the proposed shift's date and time range.
- The conflict message must name the specific agency owning the conflicting shift.
- Example: "This employee is already scheduled for Agency 1 from 8:00 AM to 4:00 PM on this date."

---

## Part 10 — Calendar System

The calendar is the core interface. It must support both billable shift display and personal sub-calendar overlays in a unified view.

### Main Scheduling Calendar
- FullCalendar React with Monthly, Weekly, Daily view toggles.
- Confirmed billable shifts as solid color-coded blocks by agency.
- Approved PTO as distinct grey overlay markers.
- Conflicts highlighted with red dashed border and warning icon.
- Admin: drag-and-drop shift adjustments and click-to-edit overlay modal.
- Agency legend with visibility toggle checkboxes.
- "Personal Calendars" sidebar listing user's sub-calendars with show/hide toggles. When visible, sub-calendar events appear as semi-transparent overlays beneath shifts.
- Today's column highlighted with a distinct background color.

### Dedicated Sub-Calendar View
- Separate page showing only sub-calendar events (no billable shifts).
- Sidebar lists own sub-calendars plus shared calendars from other employees grouped under "Shared Calendars."
- Monthly, Weekly, Daily toggles.
- Click any date or time slot to create a new event.

### Calendar Data API Behavior (`GET /api/calendar/events`)
- **Admin:** all billable shifts, all PTO markers, all sub-calendar events for all users (no visibility restrictions).
- **Employee:** own confirmed billable shifts (read-only), own PTO markers, own sub-calendar events, Public sub-calendar events from other employees where access has not been revoked.
- All role-based filtering happens in `calendar_service.py`. The frontend makes one request per view change.

### Event Rendering Rules

| Event Type | Visual Treatment |
|---|---|
| Confirmed billable shift | Solid agency color block, left accent border, agency name label |
| Draft shift | Agency color at 50% opacity, dashed border, "Draft" label |
| Conflict shift | Red dashed border overlay, AlertTriangle icon |
| Approved PTO | Grey block (`#D1D5DB` at 40% opacity), "PTO" label |
| Own sub-calendar event | Sub-calendar color, semi-transparent, no left border accent |
| Shared sub-calendar event | Sub-calendar color, semi-transparent, employee name prefix on label |
| Today column | `#FEF3C7` background on weekly/daily view |

---

## Part 11 — UI/UX Design Specifications

### Typography

Load via Google Fonts CDN in `index.html`:
```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400&display=swap" rel="stylesheet">
```

| Element | Font | Size / Weight | Line Height |
|---|---|---|---|
| Page Title (h1) | Inter | 24px / 700 | 32px |
| Section Heading (h2) | Inter | 20px / 600 | 28px |
| Card Heading (h3) | Inter | 16px / 600 | 24px |
| Body Text | Inter | 14px / 400 | 20px |
| Small / Helper Text | Inter | 12px / 400 | 16px |
| Button Text | Inter | 14px / 500 | 20px |
| Nav Links | Inter | 14px / 500 | 20px |
| Calendar Event Text | Inter | 12px / 400 | 16px |
| Monospace (IDs, times) | JetBrains Mono | 13px / 400 | 18px |

### CSS Custom Properties

Define in `frontend/src/styles/variables.css` and import globally:

```css
:root {
  /* Primary palette */
  --color-primary: #2563EB;
  --color-primary-hover: #1D4ED8;
  --color-primary-light: #DBEAFE;
  --color-secondary: #10B981;
  --color-danger: #EF4444;
  --color-warning: #F59E0B;
  --color-info: #3B82F6;

  /* Surface */
  --color-bg: #F9FAFB;
  --color-surface: #FFFFFF;
  --color-border: #E5E7EB;

  /* Text */
  --color-text-primary: #111827;
  --color-text-secondary: #6B7280;
  --color-text-muted: #9CA3AF;

  /* Navigation */
  --color-nav-bg: #1F2937;
  --color-nav-text: #F9FAFB;

  /* Agency colors */
  --color-agency-1: #3B82F6;
  --color-agency-2: #8B5CF6;
  --color-agency-3: #EC4899;
  --color-agency-4: #F97316;
  --color-agency-5: #14B8A6;
  --color-agency-6: #EAB308;
  --color-agency-7: #6366F1;
  --color-agency-8: #84CC16;

  /* Typography */
  --font-sans: 'Inter', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;

  /* Spacing */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
  --space-2xl: 48px;

  /* Layout */
  --max-content-width: 1280px;
  --nav-height: 56px;
  --side-padding: 24px;

  /* Borders */
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-pill: 12px;

  /* Responsive breakpoints (used as reference — apply via media queries) */
  /* --bp-sm: 480px   — large phones, landscape */
  /* --bp-md: 768px   — tablets */
  /* --bp-lg: 1024px  — small desktops, landscape tablets */
  /* --bp-xl: 1280px  — standard desktop */
}
```

### Primary Color Palette

| Role | Hex | Usage |
|---|---|---|
| Primary | `#2563EB` | Buttons, active nav, primary actions |
| Primary Hover | `#1D4ED8` | Button hover states |
| Primary Light | `#DBEAFE` | Selected rows, active tab backgrounds |
| Secondary | `#10B981` | Success messages, approved status |
| Danger | `#EF4444` | Delete buttons, conflicts, denied status |
| Warning | `#F59E0B` | Pending badges, warning alerts |
| Info | `#3B82F6` | Informational banners, tooltips |
| Background | `#F9FAFB` | Page background |
| Surface | `#FFFFFF` | Cards, modals, form containers |
| Border | `#E5E7EB` | Table borders, card borders, dividers |
| Text Primary | `#111827` | Headings, body text |
| Text Secondary | `#6B7280` | Labels, helper text, timestamps |
| Text Muted | `#9CA3AF` | Placeholders, disabled text |
| Nav Background | `#1F2937` | Top navigation bar |
| Nav Text | `#F9FAFB` | Navigation link text |

### Agency Calendar Colors

Assign in order of agency creation. Apply consistently to all shifts, legend swatches, and badges.

| Slot | Hex | Name |
|---|---|---|
| 1 | `#3B82F6` | Blue |
| 2 | `#8B5CF6` | Purple |
| 3 | `#EC4899` | Pink |
| 4 | `#F97316` | Orange |
| 5 | `#14B8A6` | Teal |
| 6 | `#EAB308` | Yellow |
| 7 | `#6366F1` | Indigo |
| 8 | `#84CC16` | Lime |

### UI Component Specs

**Buttons**

| Type | Background | Text | Border | Radius | Padding |
|---|---|---|---|---|---|
| Primary | `#2563EB` | `#FFFFFF` | none | 6px | 8px 16px |
| Secondary | `#FFFFFF` | `#374151` | 1px solid `#D1D5DB` | 6px | 8px 16px |
| Danger | `#EF4444` | `#FFFFFF` | none | 6px | 8px 16px |
| Ghost | transparent | `#2563EB` | none | 6px | 8px 16px |
| Disabled | `#E5E7EB` | `#9CA3AF` | none | 6px | 8px 16px |

**Status Badges**

| Status | Background | Text Color | Radius |
|---|---|---|---|
| Approved | `#D1FAE5` | `#065F46` | 12px pill |
| Denied | `#FEE2E2` | `#991B1B` | 12px pill |
| Pending | `#FEF3C7` | `#92400E` | 12px pill |

**Cards, Inputs & Modals**

| Property | Value |
|---|---|
| Card border | 1px solid `#E5E7EB` |
| Card radius | 8px |
| Card shadow | `0 1px 3px rgba(0,0,0,0.1)` |
| Card padding | 16px 20px |
| Input border | 1px solid `#D1D5DB` |
| Input radius | 6px |
| Input padding | 8px 12px |
| Input focus border | 2px solid `#2563EB` |
| Input focus shadow | `0 0 0 3px rgba(37,99,235,0.1)` |
| Input error border | 2px solid `#EF4444` |
| Modal overlay | `rgba(0,0,0,0.5)` |
| Modal radius | 12px |
| Modal shadow | `0 20px 60px rgba(0,0,0,0.3)` |
| Modal max-width | 480px |
| Modal padding | 24px |

### Icon Reference (Lucide React)

| Where Used | Lucide Icon Component |
|---|---|
| Dashboard nav | `LayoutDashboard` |
| Calendar nav | `Calendar` |
| Sub-Calendars nav | `CalendarDays` |
| Agencies nav | `Building2` |
| Employees nav | `Users` |
| PTO Requests nav | `Clock` |
| Schedule Requests nav | `ClipboardList` |
| Calendar Permissions nav | `ShieldCheck` |
| Settings nav | `Settings` |
| Logout | `LogOut` |
| Add new | `Plus` |
| Edit | `Pencil` |
| Delete | `Trash2` |
| Approve | `Check` |
| Deny / close | `X` |
| Search | `Search` |
| Filter | `SlidersHorizontal` |
| Conflict warning | `AlertTriangle` |
| Approved status | `CheckCircle2` |
| Denied status | `XCircle` |
| Pending status | `Clock4` |
| Info tooltip | `Info` |
| Calendar prev/next | `ChevronLeft` / `ChevronRight` |
| Agency visible/hidden | `Eye` / `EyeOff` |
| Drag handle | `GripVertical` |
| Generate schedule (stub) | `Wand2` |
| Access code | `KeyRound` |
| Revoke access | `UserX` |

---

## Part 12 — Mobile Responsiveness

The application must be fully usable on mobile phones (360px+), tablets (768px+), and desktops (1024px+). All layouts are built mobile-first: the base CSS targets small screens, and media queries progressively enhance for larger viewports. No feature is hidden or disabled on mobile — the layout adapts instead.

### Viewport Meta Tag

Required in `frontend/index.html`:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0" />
```

### Breakpoints

| Name | Min Width | Target Devices |
|---|---|---|
| `sm` | 480px | Large phones, landscape phones |
| `md` | 768px | Tablets, landscape phones |
| `lg` | 1024px | Small desktops, landscape tablets |
| `xl` | 1280px | Standard desktops and above |

Apply breakpoints using standard CSS `min-width` media queries:

```css
/* Base styles — mobile (< 480px) */
.page-layout { ... }

/* sm — large phones and up */
@media (min-width: 480px) { ... }

/* md — tablets and up */
@media (min-width: 768px) { ... }

/* lg — desktops and up */
@media (min-width: 1024px) { ... }

/* xl — wide desktops and up */
@media (min-width: 1280px) { ... }
```

### Navigation

- **Mobile (< 768px):** The top navigation collapses into a hamburger menu. Tapping it opens a full-height slide-in drawer from the left containing all nav links. The drawer closes when the user taps a link or taps outside the drawer. The `Menu` and `X` Lucide icons are used for the toggle.
- **Tablet (768px – 1023px):** Navigation shows a compact icon-only sidebar that expands on hover or tap. Labels are hidden to save space.
- **Desktop (1024px+):** Full sidebar navigation with icons and labels, always visible.

### Layout Behavior by Breakpoint

| Layout Area | Mobile | Tablet | Desktop |
|---|---|---|---|
| Sidebar nav | Hidden, drawer on hamburger tap | Icon-only, fixed left | Full labels, fixed left |
| Page content | Full width, single column | Full width, some two-column sections | Max-width 1280px, multi-column |
| Dashboard cards | Stacked, full width | 2-column grid | 3–4 column grid |
| Data tables | Horizontally scrollable, key columns only visible | All columns shown | All columns shown with actions |
| Forms | Full width, single column | Full width, single column | Max-width 480px or two-column side-by-side |
| Modals | Full screen (100vw, 100vh) | Centered, max-width 480px | Centered, max-width 480px |
| Shift cards | Stacked list view | Compact cards, 2-per-row | Calendar block view |

### Calendar on Mobile

FullCalendar must adapt its view based on screen size. Use the `initialView` prop set conditionally based on window width:

- **Mobile (< 768px):** Default to `listWeek` view (agenda/list format). Monthly and daily views remain accessible via the view toggle but daily is the recommended fallback if grid is needed. Hide the sidebar agency legend — replace with a compact color-dot + agency name inline on each event.
- **Tablet (768px+):** Default to `timeGridWeek` (weekly grid). Sidebar is accessible via a toggle button.
- **Desktop (1024px+):** Default to `dayGridMonth`. Full sidebar visible alongside the calendar.

FullCalendar touch behavior:
- Enable `longPressDelay: 250` to distinguish tap-to-view from tap-to-drag on touch devices.
- Drag-and-drop for shift editing is enabled on touch screens. Show a brief tooltip on first touch to inform the user.
- Event popups triggered by tap must close when tapping outside.

### Touch-Friendly Interaction Rules

- Minimum tap target size: **44px × 44px** for all buttons, links, and interactive calendar events.
- Avoid hover-only states. Any behavior triggered by hover on desktop must also be accessible by tap on mobile.
- Form inputs must not be smaller than 44px tall to prevent browser zoom on focus (especially on iOS Safari).
- Inline delete/edit actions that appear on row hover must be replaced with a swipe-to-reveal or a tap-to-expand action menu on mobile.

### Responsive Component Behavior

**Data Tables (admin employee list, shift list, request queue)**
- On mobile, collapse to a card-per-row layout. Each card shows the 2–3 most important fields with an expand toggle to see all fields.
- On tablet and above, show the standard table with horizontal scroll if needed.

**Admin Scheduling Page**
- On mobile, show a day-picker at the top and a vertical list of that day's shifts below. Tap a shift to open the edit modal.
- On tablet+, show the full FullCalendar weekly grid.

**Employee Dashboard**
- On mobile, show this week's shifts as a chronological list with date separators.
- On tablet+, show the compact calendar beside the request status sidebar.

**Modals and Forms**
- On mobile, all modals render as a bottom sheet (slides up from the bottom edge, covers 90% of screen height, with a drag handle at the top).
- On tablet+, modals render as centered overlays with the standard max-width 480px.

**Sub-Calendar Sidebar**
- On mobile, the sub-calendar toggle list is hidden by default behind a "Calendars" button that opens a bottom drawer.
- On tablet+, the sidebar is visible alongside the calendar.

### Side Padding by Breakpoint

| Breakpoint | `--side-padding` value |
|---|---|
| Mobile | 16px |
| sm (480px+) | 20px |
| md (768px+) | 24px |
| lg (1024px+) | 32px |

Override `--side-padding` at each breakpoint using media queries inside `variables.css`:

```css
:root { --side-padding: 16px; }
@media (min-width: 480px) { :root { --side-padding: 20px; } }
@media (min-width: 768px) { :root { --side-padding: 24px; } }
@media (min-width: 1024px) { :root { --side-padding: 32px; } }
```

---

## Part 13 — Seed Data

Seed the database by running `python seed.py` on first run. The app must be demonstrable immediately without any manual data entry.

### Agencies & Employees

| Agency | Color | Weekly Hours | Employees & Allocations |
|---|---|---|---|
| Agency 1 | `#3B82F6` Blue | 60 hrs/week | Employee A (40 hrs, Mon–Fri 8am–4pm), Employee B (20 hrs, Mon–Wed 9am–5pm) |
| Agency 2 | `#8B5CF6` Purple | 29 hrs/week | Employee C (10 hrs), Employee D (10 hrs), Employee E (9 hrs) |

**Key detail:** Employee A and Employee C are the same physical person. They share the same `employees` record. The seed must create at least one shift for Employee A (Agency 1) and one overlapping shift for Employee C (Agency 2) that the system flags as a conflict.

### Conflict Scenario
- Employee A: seeded shift on Monday 8:00 AM – 4:00 PM for Agency 1.
- Employee C (same `employees.id`): attempted shift on Monday 10:00 AM – 2:00 PM for Agency 2 — the system detects and records the conflict.

### PTO Scenario
- Employee B: approved 4-hour PTO on Wednesday 9:00 AM – 1:00 PM.
- Employee B: seeded shift on Wednesday 1:00 PM – 5:00 PM (outside PTO) — succeeds.
- An attempt to schedule Employee B on Wednesday 11:00 AM – 3:00 PM is blocked by the PTO rule.

### Sub-Calendar Scenario
- Employee A: sub-calendar "Personal" (Public), one event: "Doctor Appointment" Friday 11:00 AM.
- Employee B: sub-calendar "Training Notes" (Private), one event — must not appear in Employee A's view.
- Admin: sub-calendar "Admin Notes" (Admin Only) — hidden from all employees.

### Admin Account Setup
- There is no admin record in the seed script. The admin is identified at runtime by their OIDC subject value.
- Before running the app for the first time, set the following in Replit's environment secrets:

| Variable | Description |
|---|---|
| `OIDC_CLIENT_ID` | Provided by Replit's Auth configuration panel |
| `OIDC_CLIENT_SECRET` | Provided by Replit's Auth configuration panel |
| `OIDC_DISCOVERY_URL` | `https://replit.com/.well-known/openid-configuration` (change this one variable to migrate to another OIDC provider) |
| `ADMIN_OAUTH_SUBJECT` | The `sub` claim from the admin's OIDC token — identifies who gets the admin role on first login |
| `SECRET_KEY` | A long random string used to sign server-side sessions |
| `DATABASE_URL` | Automatically provided by Replit's native PostgreSQL — no manual setup needed |

---

## Part 14 — Migration Portability

This application is designed to run on Replit but can be migrated to any other platform without modifying application logic. All platform-specific details are isolated to environment variables and two files.

### What is portable by design

| Layer | How it stays portable |
|---|---|
| Database | SQLAlchemy ORM throughout — no raw SQL, no PostgreSQL-specific syntax. Connection string is read from `DATABASE_URL` only. Change that one variable to point at any PostgreSQL instance. |
| Auth | All OIDC logic lives in `auth_utils.py` and `blueprints/auth.py` only. The provider is configured entirely by `OIDC_DISCOVERY_URL`, `OIDC_CLIENT_ID`, and `OIDC_CLIENT_SECRET`. |
| User identity | Stored as `oauth_subject` (the standard OIDC `sub` claim). Every OIDC provider uses this field. |
| Sessions | Stored in the PostgreSQL database via `flask-session`. Follows the `DATABASE_URL`. |
| Migrations | Tracked by Flask-Migrate (Alembic). Run `flask db upgrade` on any new host. |

### Steps to migrate off Replit

1. Provision a PostgreSQL database on the new host. Set `DATABASE_URL` to the new connection string.
2. Choose an OIDC provider (Auth0, Okta, Google, etc.). Set `OIDC_DISCOVERY_URL`, `OIDC_CLIENT_ID`, `OIDC_CLIENT_SECRET` to the new provider's values.
3. Set `ADMIN_OAUTH_SUBJECT` to the admin's `sub` claim from the new provider.
4. Run `flask db upgrade` on the new host.
5. No application code changes required.

---

## Part 15 — Deliverables Checklist

### Admin Can:
1. Log in and access the full admin dashboard.
2. Create and manage agencies with billable hour caps and calendar colors.
3. Create employees, assign to agencies, define weekly allocations and availability.
4. Generate one-time access codes for employee self-registration.
5. Revoke or reinstate employee portal access.
6. Build weekly schedules with real-time conflict detection and prevention.
7. Review, approve, and deny PTO requests with written explanations.
8. Review, approve, and deny schedule requests with written explanations.
9. View the scheduling calendar with monthly, weekly, and daily views.
10. Drag-and-drop or manual shift adjustments with live conflict enforcement.
11. Manage sub-calendar visibility from the Calendar Permissions screen.
12. Create and manage own personal sub-calendars overlaid on the main calendar.
13. See a placeholder "Generate Suggested Schedule" button (stub — modal says "coming soon").

### Employees Can:
1. Self-register using admin-issued access code and set own credentials.
2. Log in and access personal employee dashboard.
3. View own confirmed billable shifts (read-only).
4. Submit PTO requests and track approval status with admin explanations.
5. Submit schedule change requests and track approval status.
6. Create, manage, and delete personal sub-calendars.
7. Add, edit, and delete events on own sub-calendars.
8. Toggle sub-calendar visibility between Public and Private.
9. View other employees' Public sub-calendars (unless admin revoked access).
10. Overlay personal sub-calendar events on the main calendar view.

### Technical Requirements:
- Database seeded on first run (`python seed.py`) with the full scenario from Part 12.
- All Flask routes protected by `@require_admin` or `@require_employee` decorators — no admin routes accessible to employee sessions.
- FullCalendar fetches events from `/api/calendar/events` — no hardcoded event data in the frontend.
- All conflict detection logic lives in `conflict_service.py` — never client-side only.
- The React frontend communicates with the Flask backend exclusively via JSON API calls (no server-rendered templates).
- CORS configured on the Flask backend to allow requests from the Vite dev server origin during development.
