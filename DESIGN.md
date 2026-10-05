# LessonAtlas: Product Design Specification

This file owns brand, visual tokens, layout, component conventions, and accessibility. Product behavior: [REQUIREMENTS.md](REQUIREMENTS.md). Security: [SECURITY.md](SECURITY.md). Implementation: [ARCHITECTURE.md](ARCHITECTURE.md).

## Design direction

**Brand idea:** A clear view of the learning ahead.

**Product descriptor:** Lesson planning and student support, guided by you.

The experience should feel like a well-organized teaching desk: calm, readable, practical, and attentive to detail. Use warm neutral surfaces, deep teal navigation and actions, generous space around editing tasks, and compact structured tables where teachers need to scan many records.

Reserve saturated color for actions and meaningful states. Avoid decorative metrics, childlike illustrations, gradients, oversized welcome banners, glass effects, and animated AI mascots.

## Identity and logo

### Concept

The logo combines an open book with an upward compass needle integrated into its central fold. The book expresses teaching; the compass expresses planning and direction. A restrained amber tip supplies a recognizable detail. The wordmark is exactly **LessonAtlas**, with capital L and A, no space.

Use the [initial logo artwork](assets/lessonatlas-logo.png) as the reference. Its intended construction is flat and vector-friendly, but a generated bitmap must not be described as an SVG or a finalized vector master. Inspect actual artwork before preparing production exports; do not create a different symbol under the same asset name.

### Usage

| Context | Treatment |
| --- | --- |
| Expanded app sidebar / sign-in | Horizontal symbol and wordmark on white or warm ivory. |
| Collapsed sidebar | Symbol only, derived from the approved artwork. |
| App icon / favicon | Simplified symbol-only export; check legibility at 16, 24, 32, and 48 px. |
| Printed lesson | Small monochrome-compatible mark in the header; prioritize document content. |
| Dark background | Use a separately prepared white/reversed version; do not place the deep teal wordmark on a dark surface. |

Use clear space of at least one-quarter of the emblem height on every side. Target an emblem height of 32 px in the desktop sidebar and 28 px in compact navigation. Do not stretch, rotate, add shadows, recolor arbitrarily, or bake a tagline into the compact logo. If details fail at small sizes, use a deliberately simplified symbol export rather than shrinking the full lockup.

Proposed asset names for implementation: `lessonatlas-logo.png`, `lessonatlas-mark.svg`, `lessonatlas-logo.svg`, and `favicon.svg`. Only the generated logo artwork accompanies this specification; vector and favicon exports remain production asset tasks.

### Voice

Use plain, respectful language. Refer to student support in relation to a task, objective, and evidence. Prefer “May benefit from fraction practice” to a fixed label about a student's ability. State uncertainty directly. Use “AI draft” and “Accept selected changes,” not language implying that the assistant has already changed the record.

## Visual tokens

### Color

| Token | Value | Use |
| --- | --- | --- |
| `canvas` | `#F7F8F4` | Main workspace background. |
| `surface` | `#FFFFFF` | Editor, tables, cards, dialogs. |
| `surface-muted` | `#EFF3EF` | Section headers and quiet grouped areas. |
| `ink` | `#182C2A` | Main text. |
| `ink-muted` | `#526662` | Secondary text, metadata. |
| `brand` | `#164E4A` | Primary buttons, wordmark, active navigation text. |
| `brand-hover` | `#103D3A` | Primary hover/pressed background. |
| `brand-soft` | `#E5F0EA` | Selected navigation and accepted state backgrounds. |
| `border` | `#D6DFD9` | Decorative separators and surface boundaries. |
| `control-border` | `#738880` | Input boundaries when needed for recognition. |
| `accent` | `#D49A38` | Logo detail and limited decorative emphasis. |
| `warning-ink` | `#805512` | Needs review / stale content labels. |
| `warning-soft` | `#FFF4D8` | Warning backgrounds. |
| `danger` | `#A12C3A` | Validation errors and destructive actions. |
| `danger-soft` | `#FCECEF` | Error backgrounds. |
| `info-ink` | `#345D89` | Informational labels. |
| `info-soft` | `#EDF3FA` | Informational backgrounds. |
| `focus` | `#2563EB` | Keyboard focus outline. |

White text is permitted on brand and danger fills. Amber is an accent, not small body text on white. State always combines a label with an icon or structure; color alone is insufficient. Validate final component contrast, including hover, disabled, focus, and selected states, during implementation.

### Typography

Use a system sans-serif stack for the app: `ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif`. This avoids a required external font request. The logo uses its own drawn wordmark and is not dependent on the interface font.

| Role | Size / line height | Weight |
| --- | --- | --- |
| Page title | 28 / 36 px | 650 or nearest available semibold |
| Section title | 20 / 28 px | 600 |
| Card title | 16 / 24 px | 600 |
| Body / editor | 16 / 24 px | 400 |
| Table / form label | 14 / 20 px | 400 / 600 |
| Metadata | 12 / 18 px | 400 |

Use tabular numerals for scores and dates. Use sentence case. Avoid uppercase headings and extended all-caps labels. Keep long-form editing content between approximately 65 and 85 characters per line where practical.

### Geometry and spacing

- Base spacing scale: 4, 8, 12, 16, 24, 32, 48 px.
- Main page padding: 32 px desktop, 24 px tablet, 16 px compact.
- Card radius: 12 px; form controls: 8 px; badges: 6 px.
- Controls: 44 px standard height; table rows: at least 48 px.
- Cards: usually a 1 px border with no shadow. Use a modest shadow only for floating menus/dialogs.
- Icons: 20 px interface icons using one consistent line style and approximately 1.75 px strokes.
- Primary page content: flexible width with a 1440 px maximum; data tables may use the available width.
- Motion: brief opacity/position transitions, approximately 120–180 ms; honor reduced-motion preferences.

## Teacher application shell and navigation

Student navigation and age-specific interaction details remain TO BE DECIDED under REQUIREMENTS.md FR-01; these teacher screen layouts do not define student permissions.

### Desktop: 1200 px and above

Use a 232 px left sidebar and a 64 px content header. The sidebar contains the logo, academic-term selector, primary destinations, and account/settings actions. The content header carries breadcrumbs, scoped search when relevant, and notification access. The page title and principal action sit inside the content area.

Primary destinations, in order:

1. Today
2. Lessons
3. Classes
4. Curriculum
5. Calendar

Keep Notifications in the header and Settings/account at the bottom of the sidebar. Students are accessed through Classes and a workspace-wide student search/filter within that area; do not create two competing student directories.

### Tablet: 768–1199 px

Use a compact 72 px sidebar with labeled tooltips and a menu expansion control. A detail drawer overlays part of the page only while open. The lesson editor switches from three columns to tabs plus one main content area. Preserve action labels where possible.

### Compact: below 768 px

Use a 56 px top bar with a navigation drawer. Stack cards and forms. Use a single-column editor with a “Review suggestions” destination. Tables may scroll horizontally inside a labeled region; provide a condensed student record list where wide tables would obstruct routine entry. Keep primary controls reachable without covering content. Compact layouts support essential workflows but do not introduce a separate mobile product scope.

### Route map

| Route | Purpose | Requirement families |
| --- | --- | --- |
| `/today` | Upcoming teaching and preparation | LESSON, NOTIFY, CAL |
| `/lessons` | Lesson library with class, subject, status, and date filters | LESSON |
| `/lessons/:id` | Lesson workspace | LESSON, AUDIT, MATCH, GROUP, HISTORY |
| `/classes` | Class list and student search | ROSTER |
| `/classes/:id` | Roster, attendance, assessments, progress | ROSTER, RECORD |
| `/students/:id` | Profile and individual records | PROFILE, RECORD, HISTORY |
| `/curriculum` | Subjects, scope, standards, coverage | CURR |
| `/calendar` | Calendar and reminder editing | CAL, NOTIFY |
| `/settings` | Account, preferences, import/export, AI and data settings | ACCESS, SEC, AI |

Route access follows SECURITY.md SEC-AZ-01–02.

## Screen specifications

### Today

Lead with “Today” and the date. A compact class/term filter applies to the page. The primary action is “Create lesson.”

On desktop, use an 8/4 grid: upcoming lessons on the left, preparation and reminders on the right. Show pending AI reviews below the next lessons, with counts of actual pending proposals. Each lesson row shows time, class, title, lesson state, and checklist completion such as “3 of 5 ready.” Clicking a lesson opens its editor.

Preparation items link directly to their checklist context. Reminders offer Dismiss and Snooze. Avoid average student scores, predicted success percentages, or decorative charts on the dashboard. The empty state explains how to create a class and first lesson.

### Classes and class detail

Class cards show name, subject, term, active enrollment count, and next session. The class-detail page contains tabs: Roster, Attendance, Assessments, Progress, and History.

Roster columns: student name, optional external ID, enrollment status, and last profile update. Show names as text, optionally with initials, without requiring photographs. A toolbar offers Add student, Import, and Export. Archive/end-enrollment operations use explicit labels.

Attendance uses a session selector and roster with inline status controls (RECORD-05, UI-05).

Assessments use keyboard-traversable entry tables, visible scale/status selectors, and inline validation (RECORD-01–02).

Progress displays evidence by subject/objective and period (RECORD-03); chart accessibility follows DR-A11Y-01.

### Student detail

Header: name, enrolled classes, and record status. Tabs: Overview, Learning profile, Scores, Attendance, Notes, Contacts, and History. Preserve class context in breadcrumbs while allowing a student to belong to several classes.

Overview shows recent evidence and currently recorded support needs. Each profile entry includes source, recorded date, linked objective, and active/retired state. An evidence drawer opens links in place. AI-proposed profile entries appear in a separate review section.

Contact display restrictions: SECURITY.md SEC-DATA-01. Profile and note text wraps; do not truncate crucial information behind an unexplained ellipsis.

### Lesson library

Default to active lessons sorted by upcoming date, with unscheduled drafts visible in their own filter. Support list view, search, and filters for class, subject/unit, state, and date. A row shows title, class, date or “Unscheduled,” state, and preparation completion. Row actions include Open, Duplicate, Print/export, and Archive.

### Lesson workspace

This is the main working surface. The header shows title, class, state, save status, and contextual actions. Primary action changes with context: “Mark ready” for a draft, “Mark taught” for a ready lesson. Generation is a secondary action labeled “Draft with AI.”

Desktop layout within the shell:

- Outline: approximately 176 px, listing lesson sections and missing-field indicators.
- Editor: flexible, with at least 440 px available before switching layout.
- Review panel: approximately 344 px, opened deliberately for suggestions/evidence.

If the available width cannot support all columns, collapse the outline and use the review panel as a drawer. The editor must not become a narrow strip.

Sections: Overview, Objectives and standards, Prerequisites, Activities and timing, Materials, Assessment, Student support, Preparation checklist, and Reflection. Separate tabs or destinations expose Review, Groups, and History.

Activities show per-activity and total minutes beside lesson duration. Checklists expose completion, due-date, and reorder controls (LESSON-04, AUDIT-01).

Saving labels: “Saving…”, “Saved at 10:42 AM”, “Could not save. Retry.” Mark-ready validation links to missing fields. Show the taught revision beside lesson state (LESSON-01, LESSON-05, UI-02).

### AI draft and review panel

Before generation, show task, selected standards, context, scope, a “Data used” category disclosure, and a Generate action (AI-01–02; SECURITY.md SEC-DATA-04).

Each proposal card includes:

1. “AI draft” label and destination section.
2. Proposed content or a before/after comparison.
3. Brief rationale and evidence links.
4. Evidence age and limitations when relevant.
5. Edit, select, and reject controls.

The footer reads “Accept 3 selected changes” for three selections. Accepted cards show approver/time. Selection behavior: UI-04 and AI-01.

Stale cards say “The lesson or evidence changed since this suggestion was created,” with “Review latest changes” and “Regenerate” actions (HISTORY-05).

Evidence summaries use separate fact/interpretation sections (AI-02, AI-07).

### Lesson review

Organize findings under Alignment, Prerequisites, Timing, Materials, Assessment, and Student support. Label findings “Review suggested” or “More information needed,” rather than presenting a universal lesson-quality score.

Each finding names the affected section and the evidence supporting the concern. Actions: View section, Edit suggestion, Accept change, or Dismiss. Dismissal can include a teacher reason. Show the reviewed lesson revision and a prominent stale state when inputs change.

### Student-support matching

Display a table of student, objective/prerequisite, relevant evidence, and suggested support. Permit class and lesson filters. A detail drawer contains dated evidence, uncertainty, and the proposed activity. Add-to-lesson acceptance specifies which section changes. A separate explicit flow is required to propose any profile update.

Use “Insufficient evidence for this objective” for absent evidence and a side-by-side comparison for conflicts (MATCH-02, MATCH-05).

### Small-group builder

Use a setup panel for GROUP-01–02 options and included/excluded counts. Group cards use names such as Group 1 or Cedar (GROUP-04).

Student chips expose Move to group and Lock assignment, with an assignment summary (GROUP-03; DR-A11Y-01).

Place constraint errors beside editable controls; distinguish saved groups from proposals (GROUP-02–04).

### Curriculum

Use a subject selector, an ordered unit list, and a details pane. Objectives show linked standards with exact identifiers and source versions. Coverage uses explicit text states: Unplanned, Planned, and Teacher-recorded taught. These states describe instructional coverage, not student mastery.

Show version selection alongside standard editing (CURR-04).

### Calendar and notifications

Use Week as the default CAL-01 view. Display timezone and class names alongside event colors. Clicking opens details; move behavior follows UI-06.

Recurring-edit labels: “This occurrence” and “This and future occurrences” (CAL-02–03, NOTIFY-02).

The notification center uses Unread, All, and Snoozed views and restrained counts (NOTIFY-03; scope A-05).

### Version history

Use a revision list at left and comparison at right. Each revision includes timestamp, actor, action, and provenance. Provide “Before” and “After” labels and inline additions/deletions; color is supplementary.

Use “Restore as new revision” and “Correct record” actions with previews (HISTORY-03–04; retention: SECURITY.md SEC-DATA-05).

### Import and export

Use a stepper for ROSTER-05 and RECORD-06, with separate preview/result sections for UI-07 categories.

Export begins with format/scope controls (SECURITY.md SEC-DATA-03). Print uses white paper, dark text, clear breaks, no navigation, and keeps headings with related content.

## Component behavior

| Component | Required behavior |
| --- | --- |
| Primary button | One dominant page action; text describes the effect. Disabled state includes an accessible explanation where needed. |
| Status badge | Text plus optional icon; draft, ready, taught, archived, AI draft, accepted, stale. |
| Data table | Semantic headings, sortable labels, visible filters, pagination, keyboard navigation, persistent row context. |
| Evidence drawer | Source date, record link, original scale, and relevance; returns focus to its opener. |
| Dialog | Labeled title, focused entry point, trapped focus, Escape close where safe, focus restoration. |
| Toast | Brief confirmation with accessible announcement; critical errors remain inline until resolved. |
| Empty state | Names what is missing and offers a relevant next action. |
| Skeleton | Reserves layout without implying that data already exists. |
| Date/time input | Shows timezone and validation; offers typed entry as well as a picker. |
| Selection toolbar | Shows selected count and affected scope (UI-04). |

## State presentation

UI-02 owns state behavior. Use inline Retry/unsaved indicators for save errors, a comparison for conflicts, queued/running/complete/failed labels for long jobs, and a return-to-list action for missing records. First use links class creation, roster entry/import, and first lesson. Security-sensitive messages and recovery follow SECURITY.md SEC-WEB-02, SEC-WEB-04, and SEC-AUTH-06.

## Accessibility and interaction validation

**DR-A11Y-01 — Accessibility target.** Meet all applicable WCAG 2.2 Level A and AA criteria across desktop, tablet, compact layouts, supported locales, and print/export web content. Run automated checks plus manual keyboard and screen-reader verification with a representative browser/screen-reader combination per platform and locale review. Verify contrast in every component state, accessible authentication, and language metadata (`lang` and mixed-language parts). Blocking defects prevent release; this is a target, not a conformance claim.

- Every form control has a visible label, help text association, and announced error state.
- All actions work by keyboard, including grouping, reordering, attendance, and calendar editing.
- Focus remains visible and unobscured by sticky headers, drawers, and notifications.
- Screen readers receive useful names for score statuses, table sorts, checklist completion, and save status.
- Zoom and reflow preserve workflows; test narrow layouts and 200% text zoom.
- Selection, warning, success, and revision differences remain understandable without color.
- Charts include underlying values and equivalent tabular access.
- Touch controls target 44 px or larger where practical, including compact screen actions.
- Reduced-motion settings suppress nonessential transitions.

## Asset status

The logo bitmap is the reference; vector master, reversed mark, and favicon exports remain production tasks. Logo construction may include subtle A geometry and a humanist wordmark. Do not add a globe grid, cap, robot, or sparkles. Inspect artwork before deriving exports.
