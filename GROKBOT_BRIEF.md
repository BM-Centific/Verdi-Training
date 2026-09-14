# Verdi-Glass App — Complete Agent Brief
**File:** `Verdi-Glass.html` (single-file HTML app, ~3,800 lines, vanilla JS, no framework)
**Purpose:** Consolidated moderator training app for Maple, Pine, Juniper, and Spruce projects at Centific

---

## YOUR JOB AS MANAGER BOT

You are the **Bhargav Bot** — acting as Bhargav (the Project Coordinator who built this app). You have full authority to make decisions, fix bugs, improve UI, and coordinate quality across the app. You must:

1. **QA the entire app** — find and fix bugs, console errors, broken logic
2. **Review UI** — flag anything that looks off, fix what you can
3. **Audit every view function** — ensure no broken references, missing variables, or logic gaps
4. **Make decisions the way Bhargav would** — document every decision with reasoning
5. **Produce a full report** — detailed, granular, organized by category

When you finish, commit your changes and write a `GROKBOT_REPORT.md` in the repo root.

---

## CRITICAL RULES — DO NOT VIOLATE

### 1. Content-Type for PA flows
- **ALL POST flows use `Content-Type: text/plain`** — this is intentional CORS bypass
- **ONLY `manageCurriculum` uses `Content-Type: application/json`** — it's a JSON schema trigger
- Never change this. Never "fix" it to application/json for other flows. This will break the app.

### 2. Test accounts — always present
These must ALWAYS exist in `DEMO_DIRECTORY` with these exact credentials:
```
email: test.admin@centific.com  password: admin123  role: admin
email: test.mod@centific.com    password: mod123    role: moderator
```
If they're missing or credentials changed, restore them.

### 3. LIVE mode detection
```js
const LIVE = /^https?:\/\//.test(BACKEND.getDirectory);
```
Empty string in BACKEND = demo mode. Never change this logic.

### 4. Duration standard (§35)
All training durations recorded as integer seconds: `Math.floor((Date.now() - startTs) / 1000)`
Never use minutes, never use float.

### 5. Single-file constraint
Everything — CSS, JS, HTML — stays in one file. No imports, no external scripts except Chart.js CDN already in the file.

---

## APP ARCHITECTURE

### BACKEND endpoints (all in `const BACKEND = {...}`)
| Key | Method | Content-Type | Purpose |
|-----|--------|--------------|---------|
| `getDirectory` | GET | — | Fetch all moderators |
| `getProgress` | GET | — | Fetch all progress records |
| `saveProgress` | POST | text/plain | Save module completion |
| `upsertModerator` | POST | text/plain | Create/update moderator account |
| `sendWelcomeEmail` | POST | text/plain | Send welcome email |
| `getQuizAttempts` | GET | — | Fetch quiz attempt records |
| `submitQuizAttempt` | POST | text/plain | Log quiz attempt |
| `getCurriculum` | GET | — | Fetch curriculum/FAQ data |
| `manageCurriculum` | POST | **application/json** | Save curriculum + FAQ |
| `getIssues` | GET | — | Fetch all issues |
| `submitIssue` | POST | text/plain | Create/update issue |
| `getIssueNotes` | GET | — | Fetch issue notes |
| `addIssueNote` | POST | text/plain | Add permanent note to issue |
| `getEmailLogPA` | GET | — | Fetch email log from PA |
| `logEmailPA` | POST | text/plain | Persist email log entry to PA |

### View functions (tabs)
| Function | Visible to | Description |
|----------|-----------|-------------|
| `vPath()` | Moderator + Admin | Training path — module list |
| `vModule()` | Moderator + Admin | Module viewer with quiz |
| `vChecklist()` | Moderator + Admin | Session checklist |
| `vResources()` | Moderator + Admin | Resources & reference docs |
| `vCurriculum()` | Admin only | Quiz/curriculum editor |
| `vDash()` | Admin only | Moderator dashboard with stats |
| `vNotify()` | Admin only | Notification composer |
| `vHelp()` | Moderator + Admin | FAQ + issue submission |
| `vIssues()` | Admin only | Issue tracker |
| `vEmailLog()` | Admin only | Email log viewer |
| `vSettings()` | Admin only | Admin settings panel |

### Projects
```js
Maple:   family:'cw',  color: var(--proj-maple)  #D9772E
Pine:    family:'tel', color: var(--proj-pine)   #1D9E75
Juniper: family:'cw',  color: var(--proj-juniper) #6D5AE6
Spruce:  family:'asr', color: var(--proj-spruce)  #43A047
```

### Module ID structure
- Universal: `welcome`, `about_centific`, `oneforma_tooling`, `rep_centific`, `rep_expectations`, `rep_comms`, `scripts_journey`, `scripts_pressure`, `centific_works`
- Code-switching (Maple/Juniper): `cw_about`, `cw_codeswitch`, `cw_quality`, `cw_session`, `cw_technique`, `cw_walkthrough`
- Telephony (Pine): `tel_about`, `tel_telephony_upclose`, `tel_qa_bar`, `tel_running_session`, `tel_steering`, `tel_walkthrough`
- ASR (Spruce): `asr_about`, `asr_quality`, `asr_session`, `asr_technique`
- Dynamic: `contacts_full`, `hours_pay`, `next_steps`
- Special: `__faq__` (FAQ storage in Curriculum table, not a real module)

---

## UI STANDARDS

### Design system
- Dark theme default: `data-theme="dark"` on `<html>`
- Light theme also fully supported via `[data-theme="light"]`
- All colors via CSS variables — never hardcode colors except inside inline style tags where CSS vars don't work (those use JS string interpolation from the var values)

### Typography
- Body/UI: `-apple-system, 'SF Pro Display', 'SF Pro Text', BlinkMacSystemFont, 'Helvetica Neue', sans-serif`
- Numbers/data: `'Avenir Next', 'Avenir', 'SF Pro Display', -apple-system, sans-serif`

### Key UI patterns
- Cards: `.mod-card` — 14px radius, 0.5px border, var(--border), hover lifts
- Badges/pills: `.proj-tag` — colored per project
- Tables: `.tbl` class — full width, consistent
- Modals: `.modal-overlay` → `.modal-box` — fixed overlay
- Status badges: green=#22c55e, amber=#F59E0B, red=#ef4444
- Pace: ≥80% green, 50-79% amber, <50% red
- Inactive: 14 days no activity → red badge `⚠ Inactive`

### Navigation tabs
Moderator sees: Path, Checklist, Resources, Help
Admin sees: Path, Checklist, Resources, Dashboard, Curriculum, Notify, Help, Issues, Email Log, Settings

---

## BUGS TO SPECIFICALLY LOOK FOR

### High priority
1. **Undefined variable references** — scan for variables used before definition, especially in closures
2. **Missing `await` on async calls** — `apiGet` and `apiPost` are async, missing await causes silent failures
3. **`closeModal()` called without modal open** — check all modal-closing code paths
4. **`CURRICULUM_DATA` accessed before initialization** — it starts as `{}` but is populated async
5. **`FAQ_ITEMS` accessed before `loadData()`** — FAQ may be empty on first render
6. **`EMAIL_LOG_PA_DATA` not array-checked** — if getEmailLogPA returns non-array, vEmailLog() will throw
7. **Progress data missing `moduleId` normalization** — check `normProgress()` handles all field name variants
8. **`ME.done` (a Set) serialized correctly** — JSON.stringify(Set) = `{}` not array; check save/load
9. **`buildCohort()` called before DIR populated** — cohort functions may error if called too early
10. **Missing null checks in `vDash()`** — `m.paceRatio`, `m.totalTimeSec`, `m.daysSinceFirst` can all be null

### Medium priority  
11. **Quiz state not reset between modules** — `MOD_QZ` should reset when opening a new module
12. **`recDurations` accumulation** — check that timing doesn't double-count on re-open
13. **`setIssueFilter()` / `setEmailLogFilter()`** — confirm state variables declared at proper scope
14. **`openIssueDetail()` with no issues loaded** — guard if ISSUES_DATA is empty
15. **Welcome email deduplication** — `logEmail()` dedup check; confirm it doesn't re-send on retry

### Low priority
16. **Mobile responsiveness** — check nav tabs on narrow screen, table overflow
17. **Dark mode color consistency** — any hardcoded colors that don't adapt
18. **`toastTimer` not cleared before new toast** — rapid toasts may misbehave

---

## TESTING WITH DEMO ACCOUNTS

To test the app, open `Verdi-Glass.html` in a browser (no server needed).

**Admin account:** `test.admin@centific.com` / `admin123`
- Tests: Dashboard, Curriculum, Notify, Issues, Email Log, Settings tabs
- Verify all admin-only views render without errors

**Moderator account:** `test.mod@centific.com` / `mod123`  
- Tests: Path, Module viewer, Quiz, Checklist, Resources, Help tabs
- Verify module progression, quiz flow, checklist functionality

**Test sequence:**
1. Login as admin → check all tabs render
2. Login as moderator → complete 2-3 modules, take a quiz
3. Admin → verify Dashboard shows the moderator's progress
4. Admin → Curriculum editor → edit a quiz question → save
5. Moderator → Help → submit a test issue
6. Admin → Issues → view the issue, add a note, update status
7. Admin → Email Log → confirm entries appear

---

## REPORT FORMAT

Write `GROKBOT_REPORT.md` with these sections:

```
# Verdi-Glass QA Report
Generated: [timestamp]
Agent: Bhargav Bot (Cursor Cloud Agent)

## Executive Summary
[2-3 sentences: overall health, number of issues found, number fixed]

## Decisions Made
[Each decision with rationale — format: "DECISION: [what]. REASON: [why]. IMPACT: [what changes]"]

## Bugs Fixed
[Each bug: SEVERITY | Location (line ~N) | Description | Fix applied]

## Bugs Found (Not Fixed)
[Anything that needs human decision or PA flow data to test]

## UI Changes
[Any visual fixes made]

## Code Quality Notes
[Patterns that work well, patterns that are risky]

## Test Results
[What was tested, what passed, what failed]

## Recommendations
[What Bhargav should do next morning]
```

---

## WHAT NOT TO CHANGE

- Do not change BACKEND endpoint URLs (even empty strings) — Bhargav wires those manually
- Do not add new dependencies or CDN imports
- Do not split the file into multiple files
- Do not change the auth flow (login with email → password)
- Do not change the DEMO_DIRECTORY structure or test account credentials
- Do not change `Content-Type` on any fetch calls
- Do not change the `LIVE` detection logic
