# HomeHealth Scheduler — Naming Conventions

Conventions for all code written in this project. Consistency across the full stack reduces cognitive load, prevents naming collisions, and makes the codebase navigable at scale.

---

## General Principles

- Names must be descriptive and intention-revealing. Avoid abbreviations unless universally understood (e.g., `id`, `url`, `pto`).
- Prefer clarity over brevity. `weekly_hour_allocation` (Python) and `weeklyHourAllocation` (JS/TS) are better than `wha`.
- Avoid generic names: `data`, `info`, `temp`, `obj`, `result`, `response` on their own are not acceptable — qualify them (`shift_data`, `agency_info`, `conflictResponse`).
- Never use Hungarian notation (e.g., `strName`, `bIsActive`, `iCount`).
- Boolean names must read as a yes/no question: `is_active`, `is_draft`, `has_conflict` (Python) / `isActive`, `isDraft`, `hasConflict` (TypeScript).
- Each language section below defines the casing rules for that layer. Follow each layer's rules strictly — do not mix casing styles across layers.

---

## Python (Flask Backend)

Follows [PEP 8](https://peps.python.org/pep-0008/) throughout.

### Variables and Parameters
- Use `snake_case`.

```python
weekly_hour_allocation = 40
current_user_id = session.get("user_id")
conflicting_shift = find_overlap(employee_id, date)
```

### Constants (module-level, truly immutable)
- Use `SCREAMING_SNAKE_CASE`.
- Define in the module that owns the concept, or in a shared `constants.py` if used across multiple modules.

```python
MAX_PTO_HOURS = 8
MIN_SHIFT_DURATION_HOURS = 2
SHIFT_INCREMENT_MINUTES = 15
DEFAULT_ADMIN_USERNAME = "admin"

AGENCY_COLORS = [
    "#3B82F6",
    "#8B5CF6",
    "#EC4899",
    "#F97316",
    "#14B8A6",
    "#EAB308",
    "#6366F1",
    "#84CC16",
]

USER_ROLES = ("admin", "employee")
REQUEST_STATUSES = ("pending", "approved", "denied")
VISIBILITY_TYPES = ("public", "private", "admin_only")
```

### Functions
- Use `snake_case`.
- Names should be verb or verb phrases describing what the function does.
- Boolean-returning functions should be prefixed with `is_`, `has_`, or `can_`.
- Service-layer functions that perform database writes should be prefixed with the action: `create_`, `update_`, `delete_`.
- Query functions that read data should be prefixed with `get_` or `find_`.

```python
def calculate_billable_hours(start_time: str, end_time: str) -> float: ...
def check_shift_overlap(employee_id: int, date: str, start: str, end: str) -> bool: ...
def get_employee_shifts(employee_id: int, week_start: str) -> list[dict]: ...
def create_shift(employee_id: int, agency_id: int, date: str, start_time: str, end_time: str) -> dict: ...
def is_within_pto_block(shift_start: str, shift_end: str, pto_blocks: list) -> bool: ...
def find_conflicting_shifts(employee_id: int, date: str, start: str, end: str) -> list: ...
```

### Classes
- Use `PascalCase`.
- Class names should be nouns representing the entity or concept.
- SQLAlchemy model classes match the entity name exactly (singular).

```python
class User(db.Model): ...
class Employee(db.Model): ...
class Agency(db.Model): ...
class Shift(db.Model): ...
class PtoRequest(db.Model): ...
class ScheduleRequest(db.Model): ...
class SubCalendar(db.Model): ...
class SubCalendarEvent(db.Model): ...
class EmployeeAgencyAssignment(db.Model): ...
class WeeklyHourAllocation(db.Model): ...
class EmployeeAccessCode(db.Model): ...
class CalendarVisibilityGrant(db.Model): ...
```

### Flask Blueprints
- Blueprint variable names: `snake_case`, named for the domain they own.
- Blueprint `name` argument: `snake_case`.
- Blueprint `url_prefix`: `kebab-case`.

```python
auth_bp = Blueprint("auth", __name__, url_prefix="/api/auth")
admin_agencies_bp = Blueprint("admin_agencies", __name__, url_prefix="/api/admin/agencies")
admin_shifts_bp = Blueprint("admin_shifts", __name__, url_prefix="/api/admin/shifts")
employee_bp = Blueprint("employee", __name__, url_prefix="/api/employee")
calendar_bp = Blueprint("calendar", __name__, url_prefix="/api/calendar")
sub_calendars_bp = Blueprint("sub_calendars", __name__, url_prefix="/api/sub-calendars")
```

### Route Handler Functions
- Use `snake_case`.
- Name them for the HTTP action + resource they handle.
- Must be unique within the application (Flask requires unique endpoint names).

```python
@auth_bp.get("/login")        # initiates Replit OIDC redirect
def login(): ...

@auth_bp.get("/callback")     # OIDC callback — upserts user, sets session
def callback(): ...

@admin_agencies_bp.get("/")
def list_agencies(): ...

@admin_agencies_bp.post("/")
def create_agency(): ...

@admin_agencies_bp.get("/<int:agency_id>")
def get_agency(agency_id): ...

@admin_agencies_bp.put("/<int:agency_id>")
def update_agency(agency_id): ...

@admin_agencies_bp.delete("/<int:agency_id>")
def delete_agency(agency_id): ...

@admin_shifts_bp.post("/")
def create_shift(): ...

@admin_pto_bp.put("/<int:request_id>/review")
def review_pto_request(request_id): ...
```

### Decorators
- Use `snake_case`.
- Name them as adjectives or past participles that describe what they enforce.

```python
@require_auth
@require_admin
@require_employee
```

### Files and Modules
- Use `snake_case`, always lowercase.
- Module names should be short nouns identifying the domain.

```
models/user.py
models/shift.py
models/pto_request.py
models/sub_calendar.py
models/sub_calendar_event.py
models/employee_agency_assignment.py
models/weekly_hour_allocation.py
models/employee_access_code.py
models/calendar_visibility_grant.py

blueprints/auth.py
blueprints/admin/agencies.py
blueprints/admin/shifts.py
blueprints/admin/pto_requests.py
blueprints/admin/schedule_requests.py
blueprints/admin/calendar_permissions.py
blueprints/employee/shifts.py
blueprints/employee/pto_requests.py

services/conflict_service.py
services/shift_service.py
services/pto_service.py
services/calendar_service.py
services/access_code_service.py

utils/time_utils.py
utils/decorators.py
```

### `to_dict()` Methods on Models
- Every SQLAlchemy model must implement a `to_dict()` method that returns a plain Python dict for JSON serialization.
- Keys in the returned dict use `snake_case` to match the database column names.
- Timestamps and dates are serialized to ISO 8601 strings.

```python
def to_dict(self) -> dict:
    return {
        "id": self.id,
        "employee_id": self.employee_id,
        "agency_id": self.agency_id,
        "date": self.date.isoformat(),
        "start_time": self.start_time,
        "end_time": self.end_time,
        "billable_hours": float(self.billable_hours),
        "is_draft": self.is_draft,
        "notes": self.notes,
    }
```

---

## Database (SQLAlchemy + PostgreSQL)

### Table Names
- `snake_case`, plural nouns, defined in the `__tablename__` attribute.

```python
class User(db.Model):
    __tablename__ = "users"

class Shift(db.Model):
    __tablename__ = "shifts"

class PtoRequest(db.Model):
    __tablename__ = "pto_requests"

class EmployeeAgencyAssignment(db.Model):
    __tablename__ = "employee_agency_assignments"

class SubCalendar(db.Model):
    __tablename__ = "sub_calendars"

class SubCalendarEvent(db.Model):
    __tablename__ = "sub_calendar_events"

class CalendarVisibilityGrant(db.Model):
    __tablename__ = "calendar_visibility_grants"

class WeeklyHourAllocation(db.Model):
    __tablename__ = "weekly_hour_allocations"

class EmployeeAccessCode(db.Model):
    __tablename__ = "employee_access_codes"
```

### Column Names
- Database column names: `snake_case`.
- SQLAlchemy attribute names on the model class: `snake_case` (same as column name — no divergence).

```python
class Shift(db.Model):
    __tablename__ = "shifts"

    id = db.Column(db.Integer, primary_key=True)
    employee_id = db.Column(db.Integer, db.ForeignKey("employees.id"), nullable=False)
    agency_id = db.Column(db.Integer, db.ForeignKey("agencies.id"), nullable=False)
    date = db.Column(db.Date, nullable=False)
    start_time = db.Column(db.String(5), nullable=False)
    end_time = db.Column(db.String(5), nullable=False)
    billable_hours = db.Column(db.Numeric(5, 2), nullable=False)
    is_draft = db.Column(db.Boolean, default=False)
    notes = db.Column(db.Text, nullable=True)
```

### Relationships
- Named as the related entity in `snake_case`, plural for one-to-many, singular for many-to-one.

```python
class Employee(db.Model):
    shifts = db.relationship("Shift", backref="employee", lazy="dynamic")
    assignments = db.relationship("EmployeeAgencyAssignment", backref="employee", lazy="select")
    pto_requests = db.relationship("PtoRequest", backref="employee", lazy="dynamic")
```

### Flask-Migrate
- Migration files are auto-generated by Alembic. Do not rename them.
- Run `flask db migrate -m "short description"` with a lowercase, hyphen-separated description of the change.

```
flask db migrate -m "add-shifts-table"
flask db migrate -m "add-is-draft-to-shifts"
flask db migrate -m "add-calendar-visibility-grants"
```

---

## TypeScript / JavaScript (React Frontend)

### Variables and Parameters
- Use `camelCase`.

```ts
const weeklyHourAllocation = 40;
const currentUserId = authContext.user?.id;
const conflictingShift = await fetchConflict(employeeId, date);
```

### Constants (module-level, truly immutable)
- Use `SCREAMING_SNAKE_CASE`.

```ts
const MAX_PTO_HOURS = 8;
const MIN_SHIFT_DURATION_HOURS = 2;
const SHIFT_INCREMENT_MINUTES = 15;

const AGENCY_COLORS = [
  "#3B82F6",
  "#8B5CF6",
  "#EC4899",
  "#F97316",
  "#14B8A6",
  "#EAB308",
  "#6366F1",
  "#84CC16",
] as const;

// Breakpoint constants — must match the values in variables.css exactly
const BREAKPOINT_SM = 480;
const BREAKPOINT_MD = 768;
const BREAKPOINT_LG = 1024;
const BREAKPOINT_XL = 1280;

// Minimum touch target size per WCAG 2.5.5
const MIN_TOUCH_TARGET_PX = 44;
```

### Functions
- Use `camelCase`.
- Names should be verb phrases describing what the function does.
- Prefix React event handlers with `handle`.
- Prefix data-fetching functions with `fetch` or `get`.
- Prefix boolean-returning functions with `is`, `has`, or `can`.

```ts
function calculateBillableHours(startTime: string, endTime: string): number {}
function handleApproveRequest(requestId: number): Promise<void> {}
async function fetchEmployeeShifts(employeeId: number, weekStart: string): Promise<Shift[]> {}
function isWithinPtoBlock(shiftStart: string, shiftEnd: string, ptoBlocks: PtoBlock[]): boolean {}
function formatTimeDisplay(time: string): string {}
function snapToFifteenMinutes(time: string): string {}
```

### Types and Interfaces
- Use `PascalCase`.
- Do not prefix interfaces with `I`.
- Suffix API request body shapes with `Request`.
- Suffix API response shapes (when they wrap the entity) with `Response`.
- Suffix React form state types with `FormValues`.

```ts
type User = { id: number; oauth_subject: string; display_name: string; profile_image_url: string | null; role: "admin" | "employee"; is_active: boolean };
type Shift = { id: number; employee_id: number; agency_id: number; date: string; start_time: string; end_time: string; billable_hours: number; is_draft: boolean };
type Agency = { id: number; name: string; total_weekly_hours: number; color_hex: string };
type CreateShiftRequest = { employee_id: number; agency_id: number; date: string; start_time: string; end_time: string };
type ShiftConflictResponse = { error: "conflict"; message: string; details: ConflictDetails };
type PtoRequestFormValues = { date: string; hours_requested: number; reason: string };
```

> Note: API response fields use `snake_case` because they are deserialized directly from the Flask JSON responses. Frontend-only computed or display values use `camelCase`.

### Union Type Literals
- Prefer string union literals over enums for all status and role fields. These values are shared with the backend and must match exactly.

```ts
type UserRole = "admin" | "employee";
type RequestStatus = "pending" | "approved" | "denied";
type SubCalendarVisibility = "public" | "private" | "admin_only";
type GrantType = "allowed" | "revoked";
```

### API Client Functions (`frontend/src/api/`)
- One file per backend resource domain.
- Functions named as `verb + Resource`, using `camelCase`.
- All functions are `async` and return typed promises.

```ts
// api/shifts.ts
export async function getAdminShifts(params: ShiftFilterParams): Promise<Shift[]> {}
export async function createShift(data: CreateShiftRequest): Promise<Shift> {}
export async function updateShift(id: number, data: UpdateShiftRequest): Promise<Shift> {}
export async function deleteShift(id: number): Promise<void> {}

// api/ptoRequests.ts
export async function submitPtoRequest(data: CreatePtoRequest): Promise<PtoRequest> {}
export async function getMyPtoRequests(): Promise<PtoRequest[]> {}
export async function reviewPtoRequest(id: number, data: ReviewRequest): Promise<PtoRequest> {}
```

---

## React Components

### Component Files and Names
- One component per file.
- File name matches the component name, in `PascalCase`.
- Use `.tsx` extension for all components.

```
ShiftCard.tsx           → export function ShiftCard() {}
AgencyBadge.tsx         → export function AgencyBadge() {}
PtoRequestForm.tsx      → export function PtoRequestForm() {}
ConflictBanner.tsx      → export function ConflictBanner() {}
AdminSidebar.tsx        → export function AdminSidebar() {}
```

### Component Props Types
- Suffix with `Props`, defined immediately above the component.

```tsx
type ShiftCardProps = {
  shift: Shift;
  agency: Agency;
  onEdit: (shiftId: number) => void;
  isDraft?: boolean;
};

export function ShiftCard({ shift, agency, onEdit, isDraft = false }: ShiftCardProps) {
  return (
    <div className="shift-card">...</div>
  );
}
```

### Hooks
- Always prefix with `use`, in `camelCase`.
- File name matches the hook name exactly.

```ts
useAuth.ts            → export function useAuth() {}
useShifts.ts          → export function useShifts(employeeId?: number) {}
useCalendarEvents.ts  → export function useCalendarEvents(start: Date, end: Date) {}
useConflictCheck.ts   → export function useConflictCheck() {}
```

**Responsive hooks** — named for what they detect, not how they detect it:

```ts
useBreakpoint.ts  → export function useBreakpoint(): "mobile" | "tablet" | "desktop" {}
useIsMobile.ts    → export function useIsMobile(): boolean {}
```

`useBreakpoint` wraps `window.matchMedia` and returns the current named breakpoint tier. `useIsMobile` is a convenience wrapper that returns `true` when the viewport is below `BREAKPOINT_MD` (768px). Use these hooks to conditionally render layout variants in React — do not use them for styling (that belongs in CSS).

```tsx
function SchedulingPage() {
  const isMobile = useIsMobile();

  return isMobile
    ? <MobileShiftList />
    : <FullCalendarView />;
}
```

### Context
- Context value type: suffix with `ContextValue`.
- Context object: suffix with `Context`.
- Provider component: suffix with `Provider`.
- Consumer hook: named for the domain (drop `Context`).

```tsx
type AuthContextValue = { user: User | null; logout: () => void; isLoading: boolean };
// No login() callback — login is a browser redirect to GET /api/auth/login (Replit OIDC)
const AuthContext = createContext<AuthContextValue | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {}
export function useAuth(): AuthContextValue {}
```

### Responsive Component Variants
When a component has a fundamentally different structure on mobile vs. desktop (not just a CSS layout shift), create separate named components and a parent that switches between them using `useIsMobile`.

- Prefix the mobile variant with `Mobile`.
- Prefix the desktop variant with `Desktop` only if it would otherwise be ambiguous — in most cases the base name is the desktop version.
- The parent (switcher) component keeps the original name with no prefix.

```
ShiftList.tsx           → renders MobileShiftList or DesktopShiftList based on useIsMobile()
MobileShiftList.tsx     → chronological list view for small screens
DesktopShiftList.tsx    → table view for large screens (only needed if the base name is ambiguous)

NavSidebar.tsx          → always-visible sidebar — rendered on lg+
NavDrawer.tsx           → slide-in drawer — rendered on mobile
```

- Do not create `SmallShiftList`, `TinyNav`, or other size-adjective names — use `Mobile` / `Desktop` exclusively to keep variant names predictable.
- If the difference between breakpoints is only spacing, font size, or column count, handle it in CSS only — no separate component needed.

### Pages
- `PascalCase`, suffix with `Page`.
- Stored in `pages/` grouped by role.

```
pages/auth/LoginPage.tsx          ← redirects to /api/auth/login (Replit OIDC)
pages/auth/LinkAccountPage.tsx    ← shown to new users; collects access code
pages/admin/DashboardPage.tsx
pages/admin/AgenciesPage.tsx
pages/admin/EmployeesPage.tsx
pages/admin/EmployeeDetailPage.tsx
pages/admin/SchedulingPage.tsx
pages/admin/PtoReviewPage.tsx
pages/admin/ScheduleRequestsPage.tsx
pages/admin/CalendarPermissionsPage.tsx
pages/employee/DashboardPage.tsx
pages/employee/CalendarPage.tsx
pages/employee/SubCalendarPage.tsx
pages/employee/RequestsPage.tsx
```

---

## CSS

### File Names
- `kebab-case`, always lowercase.

```
styles/global.css
styles/variables.css
styles/components.css
styles/calendar.css
styles/forms.css
styles/nav.css
styles/responsive.css
```

### CSS Custom Properties
- All custom properties defined in `variables.css` under `:root`.
- Use `--kebab-case`, prefixed by category: `--color-`, `--font-`, `--space-`, `--radius-`, `--shadow-`, `--bp-`.
- Properties that change across breakpoints (like `--side-padding`) are re-declared inside the appropriate media query.

```css
:root {
  --color-primary: #2563EB;
  --color-primary-hover: #1D4ED8;
  --color-danger: #EF4444;
  --color-nav-bg: #1F2937;
  --color-agency-1: #3B82F6;
  --color-agency-2: #8B5CF6;
  --font-sans: 'Inter', sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
  --space-2xl: 48px;
  --radius-sm: 6px;
  --radius-md: 8px;
  --radius-pill: 12px;
  --shadow-card: 0 1px 3px rgba(0, 0, 0, 0.1);
  --shadow-modal: 0 20px 60px rgba(0, 0, 0, 0.3);

  /* Side padding adapts per breakpoint */
  --side-padding: 16px;
}

@media (min-width: 480px) { :root { --side-padding: 20px; } }
@media (min-width: 768px) { :root { --side-padding: 24px; } }
@media (min-width: 1024px) { :root { --side-padding: 32px; } }
```

### Breakpoint Names and Values

The four named breakpoints used throughout the project. Apply them consistently using `min-width` media queries (mobile-first).

| Token Name | Min Width | Targets |
|---|---|---|
| `sm` | 480px | Large phones, landscape phones |
| `md` | 768px | Tablets, landscape phones |
| `lg` | 1024px | Small desktops, landscape tablets |
| `xl` | 1280px | Standard desktops and above |

The breakpoint names `sm`, `md`, `lg`, `xl` are also used in:
- TypeScript constant names: `BREAKPOINT_SM`, `BREAKPOINT_MD`, `BREAKPOINT_LG`, `BREAKPOINT_XL`
- CSS comment labels on every media query block
- React hook names: `useBreakpoint`

```css
/* Always comment every media query with its breakpoint name */

/* md — tablets and up */
@media (min-width: 768px) {
  .nav-sidebar { display: flex; }
  .nav-drawer  { display: none; }
}

/* lg — desktops and up */
@media (min-width: 1024px) {
  .nav-sidebar { width: 240px; }
}
```

### Media Query Ordering
- Write base (mobile) styles first, then layer up with `min-width` queries in ascending order.
- Never mix `min-width` and `max-width` queries for the same property — pick one direction and stay consistent.
- Group all responsive overrides for a component together at the bottom of its rule block, not in a separate file.

```css
/* Correct — mobile-first, ascending breakpoints */
.dashboard-grid {
  display: flex;
  flex-direction: column;
  gap: var(--space-md);
}

@media (min-width: 768px) {
  .dashboard-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .dashboard-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### CSS Class Names
- Use `kebab-case` for all class names.
- Use BEM-style naming (`block__element--modifier`) for component-specific classes to avoid collisions.
- Use `--mobile`, `--tablet`, `--desktop` modifiers only when a component has a completely different structural variant per breakpoint (rare — prefer media queries on the base class).
- Use utility-style single-purpose classes only for shared layout primitives.

```css
/* Block */
.shift-card { ... }

/* Element */
.shift-card__header { ... }
.shift-card__time { ... }
.shift-card__agency-label { ... }

/* Modifier — state */
.shift-card--draft { opacity: 0.5; border-style: dashed; }
.shift-card--conflict { border: 2px dashed #EF4444; }

/* Status badge */
.status-badge { ... }
.status-badge--approved { background: #D1FAE5; color: #065F46; }
.status-badge--denied { background: #FEE2E2; color: #991B1B; }
.status-badge--pending { background: #FEF3C7; color: #92400E; }

/* Navigation variants */
.nav-sidebar { ... }          /* always-visible sidebar — shown on lg+ */
.nav-drawer { ... }           /* slide-in drawer — shown on mobile */
.nav-drawer--open { ... }     /* drawer open state */

/* Utility */
.visually-hidden { ... }
.text-mono { font-family: var(--font-mono); }
.touch-target { min-height: 44px; min-width: 44px; } /* enforces min tap size */
```

### Inline Styles (React)
- Only use inline styles for dynamic values that cannot be expressed as static CSS classes (e.g., agency color from the database, dynamic widths from JavaScript).
- Never use inline styles to work around responsive layout — that belongs in CSS.

```tsx
<div
  className="shift-card"
  style={{ borderLeftColor: agency.color_hex }}
>
```

---

## API Endpoints

### URL Paths
- `kebab-case`, plural resources, RESTful conventions.
- Path parameters: `snake_case` with angle brackets in Flask route notation.

```
GET    /api/auth/login              ← initiates Replit OIDC redirect
GET    /api/auth/callback           ← OIDC callback, upserts user, sets session
POST   /api/auth/logout
GET    /api/auth/me
POST   /api/auth/link-access-code  ← links Replit identity to employee record on first login

GET    /api/admin/agencies
POST   /api/admin/agencies
GET    /api/admin/agencies/<int:agency_id>
PUT    /api/admin/agencies/<int:agency_id>
DELETE /api/admin/agencies/<int:agency_id>

GET    /api/admin/employees
POST   /api/admin/employees
GET    /api/admin/employees/<int:employee_id>
PUT    /api/admin/employees/<int:employee_id>
DELETE /api/admin/employees/<int:employee_id>

POST   /api/admin/access-codes
DELETE /api/admin/access-codes/<int:code_id>

GET    /api/admin/shifts
POST   /api/admin/shifts
PUT    /api/admin/shifts/<int:shift_id>
DELETE /api/admin/shifts/<int:shift_id>

GET    /api/admin/pto-requests
PUT    /api/admin/pto-requests/<int:request_id>/review

GET    /api/calendar/events

GET    /api/sub-calendars
POST   /api/sub-calendars
GET    /api/sub-calendars/<int:calendar_id>/events
POST   /api/sub-calendars/<int:calendar_id>/events
PUT    /api/sub-calendars/<int:calendar_id>/events/<int:event_id>
DELETE /api/sub-calendars/<int:calendar_id>/events/<int:event_id>
```

### JSON Request & Response Keys
- Always `snake_case` — this is the contract between Flask and the React frontend.
- The frontend reads these keys directly from the API response without transformation.

```json
{
  "employee_id": 1,
  "agency_id": 2,
  "start_time": "08:00",
  "end_time": "16:00",
  "billable_hours": 8.0,
  "is_draft": false
}
```

---

## File and Directory Naming

### General Rules
- All directory names: `snake_case` (Python/backend) or `kebab-case` (frontend).
- Never use spaces or uppercase in directory names.
- Group files by domain/feature, not by type.

### Backend (`backend/`)
```
backend/
  app/
    __init__.py
    extensions.py
    config.py
    models/
      user.py
      employee.py
      agency.py
      shift.py
      pto_request.py
      schedule_request.py
      sub_calendar.py
      sub_calendar_event.py
      employee_agency_assignment.py
      weekly_hour_allocation.py
      employee_access_code.py
      calendar_visibility_grant.py
    blueprints/
      auth.py
      admin/
        agencies.py
        employees.py
        assignments.py
        shifts.py
        pto_requests.py
        schedule_requests.py
        access_codes.py
        calendar_permissions.py
      employee/
        shifts.py
        pto_requests.py
        schedule_requests.py
        shared_calendars.py
      calendar.py
      sub_calendars.py
    services/
      conflict_service.py
      shift_service.py
      pto_service.py
      calendar_service.py
      access_code_service.py
    utils/
      time_utils.py
      decorators.py
  migrations/
  seed.py
  run.py
  requirements.txt
```

### Frontend (`frontend/`)
```
frontend/
  src/
    api/
      client.ts
      auth.ts
      agencies.ts
      employees.ts
      shifts.ts
      pto-requests.ts
      schedule-requests.ts
      sub-calendars.ts
      calendar.ts
    components/
      calendar/
        ShiftEvent.tsx
        PtoMarker.tsx
        AgencyLegend.tsx
        SubCalendarSidebar.tsx
      shifts/
        ShiftCard.tsx
        ShiftFormModal.tsx
        ConflictBanner.tsx
      agencies/
        AgencyBadge.tsx
        AgencyForm.tsx
      employees/
        EmployeeRow.tsx
        EmployeeForm.tsx
        AccessCodeDisplay.tsx
      pto/
        PtoRequestForm.tsx
        PtoStatusBadge.tsx
      sub-calendars/
        SubCalendarList.tsx
        SubCalendarEventForm.tsx
      ui/
        Button.tsx
        Modal.tsx
        Badge.tsx
        Card.tsx
        StatusBadge.tsx
    pages/
      auth/
        LoginPage.tsx
        LinkAccountPage.tsx
      admin/
        DashboardPage.tsx
        AgenciesPage.tsx
        EmployeesPage.tsx
        EmployeeDetailPage.tsx
        SchedulingPage.tsx
        PtoReviewPage.tsx
        ScheduleRequestsPage.tsx
        CalendarPermissionsPage.tsx
      employee/
        DashboardPage.tsx
        CalendarPage.tsx
        SubCalendarPage.tsx
        RequestsPage.tsx
    hooks/
      useAuth.ts
      useShifts.ts
      useCalendarEvents.ts
      useConflictCheck.ts
      useBreakpoint.ts       # returns "mobile" | "tablet" | "desktop"
      useIsMobile.ts         # returns boolean, true when < 768px
    context/
      AuthContext.tsx
    constants/
      agencyColors.ts
      breakpoints.ts         # BREAKPOINT_SM, MD, LG, XL
      roles.ts
      statusLabels.ts
    types/
      index.ts
    styles/
      global.css
      variables.css          # CSS custom properties + breakpoint overrides
      components.css
      calendar.css
      forms.css
      nav.css
      responsive.css         # Layout-level responsive overrides (grid, sidebar, etc.)
```

---

## Error Handling

### Flask Error Responses
- All error responses return JSON with consistent keys: `error` (short machine-readable string) and `message` (human-readable explanation).
- Use `snake_case` for all JSON keys.
- Include a `details` object when additional context is needed.

```python
return jsonify({
    "error": "conflict",
    "message": "This employee is already scheduled for Agency 1 from 8:00 AM to 4:00 PM on this date.",
    "details": {
        "conflicting_shift_id": 42,
        "agency_name": "Agency 1",
        "conflict_start": "08:00",
        "conflict_end": "16:00"
    }
}), 409

return jsonify({"error": "unauthorized", "message": "Login required."}), 401
return jsonify({"error": "forbidden", "message": "Admin access required."}), 403
return jsonify({"error": "not_found", "message": "Shift not found."}), 404
return jsonify({"error": "validation_error", "message": "hours_requested cannot exceed 8."}), 422
```

### Python Exception Classes
- `PascalCase`, suffix with `Error`.
- Raised in service functions and caught in route handlers.

```python
class ConflictError(Exception): ...
class ValidationError(Exception): ...
class NotFoundError(Exception): ...
class AccessDeniedError(Exception): ...
```

### Frontend Error Handling
- API client functions catch non-OK responses and throw typed errors.
- Error types follow `PascalCase` with `Error` suffix.

```ts
class ApiError extends Error {
  constructor(public status: number, public code: string, message: string) {
    super(message);
  }
}

class ConflictError extends ApiError {}
class NotFoundError extends ApiError {}
class UnauthorizedError extends ApiError {}
```

---

## Environment Variables

### Naming Rules
- `SCREAMING_SNAKE_CASE`.
- Prefixed by category so all related variables group together alphabetically and are easy to find.
- Never embed provider brand names (e.g., `REPLIT_`, `AUTH0_`) in variable names — use the generic category prefix so the variable survives a provider change without a rename.

### Required Variables

| Variable | Category Prefix | Description |
|---|---|---|
| `DATABASE_URL` | DB | Full PostgreSQL connection string. Provided automatically by Replit; replace with any PostgreSQL URL to migrate. |
| `SECRET_KEY` | — | Long random string for signing server-side sessions. |
| `OIDC_DISCOVERY_URL` | `OIDC_` | URL to the OIDC provider's discovery document. Default: `https://replit.com/.well-known/openid-configuration`. Change this one variable to switch providers. |
| `OIDC_CLIENT_ID` | `OIDC_` | OAuth client ID issued by the OIDC provider. |
| `OIDC_CLIENT_SECRET` | `OIDC_` | OAuth client secret issued by the OIDC provider. |
| `ADMIN_OAUTH_SUBJECT` | `ADMIN_` | The `sub` claim from the admin's OIDC token. Used to grant admin role on first login. |

### Rules for Adding New Variables
- Group by prefix: all `OIDC_` vars together, all `ADMIN_` vars together.
- If a variable is truly platform-specific (e.g., a Replit object storage bucket name), use a descriptive prefix that describes the resource (`STORAGE_BUCKET_NAME`), not the platform (`REPLIT_BUCKET_NAME`).
- Document every new variable in `config.py` with a comment explaining its purpose and default value.

```python
# config.py — reads all platform config from env vars
class Config:
    SECRET_KEY          = os.environ.get("SECRET_KEY", "dev-secret-change-me")
    SQLALCHEMY_DATABASE_URI = os.environ.get("DATABASE_URL")
    OIDC_DISCOVERY_URL  = os.environ.get("OIDC_DISCOVERY_URL", "https://replit.com/.well-known/openid-configuration")
    OIDC_CLIENT_ID      = os.environ.get("OIDC_CLIENT_ID")
    OIDC_CLIENT_SECRET  = os.environ.get("OIDC_CLIENT_SECRET")
    ADMIN_OAUTH_SUBJECT = os.environ.get("ADMIN_OAUTH_SUBJECT")
```

---

## Git

### Branch Names
- `kebab-case`, prefixed with the type of work.

```
feat/admin-shift-scheduling
feat/employee-pto-requests
feat/conflict-detection-service
feat/sub-calendar-visibility-grants
feat/fullcalendar-integration
fix/weekly-hour-overflow-validation
fix/pto-block-not-excluding-billable
fix/multi-agency-overlap-detection
chore/seed-script
chore/flask-migrate-init
chore/sqlalchemy-models
```

### Commit Messages
- Imperative mood, present tense, lowercase, 72 characters max.

```
add shift conflict detection service
implement pto blocking in conflict service
add flask blueprints for admin agency routes
seed demo data with multi-agency conflict scenario
fix weekly allocation validation on agency update
add sub-calendar visibility grant endpoints
add require_admin decorator for all admin routes
fix pto hours not excluded from billable totals
add fullcalendar react integration to scheduling page
```
