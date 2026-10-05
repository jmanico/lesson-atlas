# Teacher Planning Assistant: Requirements

Product scope and acceptance live here. Visual conventions: [DESIGN.md](DESIGN.md). Implementation: [ARCHITECTURE.md](ARCHITECTURE.md). Security requirements, including ACCESS-01–05, AI-03–05, HISTORY-06, and SEC-01–08: [SECURITY.md](SECURITY.md).

LessonAtlas helps teachers organize student information, prepare standards-aligned lessons, identify support needs, plan small groups, and manage classroom schedules.

All requirements apply to the initial release unless deferred. Assumptions are proposed defaults awaiting confirmation.

## Initial scope and assumptions

| ID | Proposed default |
| --- | --- |
| A-01 | Responsive web scope; acceptance is defined by QUALITY-06 and UI-01. |
| A-02 | Each teacher initially has a private workspace containing multiple classes, subjects, and academic terms. Shared teaching and school administration are deferred. |
| A-03 | Teachers and students are application users. Student workflows, age range, and account provisioning require the decisions in SECURITY.md SQ-01–03; guardians remain contact records without an approved login scope. |
| A-04 | Teachers enter or import their own educational standards. No particular country, grade range, curriculum, or standards catalog is assumed. |
| A-05 | Notifications are delivered inside the app. Email, SMS, push notifications, parent messaging, and external calendar synchronization are deferred. |
| A-06 | Manual entry and CSV import cover structured student and academic records. Standards and reference materials can be entered or pasted as text with source metadata. Arbitrary document ingestion is deferred. |
| A-07 | AI is optional: with the provider unavailable, teachers can complete all manual recordkeeping and planning workflows. |

### Explicitly outside the initial release

- Autonomous grading, diagnosis, special-education eligibility decisions, or disciplinary recommendations.
- Automatic changes to student records, lesson plans, groups, or calendars based on AI output.
- Student-facing tutoring, parent portals, and direct messaging; student workflows beyond FR-01 remain TO BE DECIDED.
- Student information system or learning management system integrations.
- Billing, district administration, public content sharing, and simultaneous collaborative editing.
- Clinical records or full individualized education plan management. Store only teacher-authorized instructional support information needed for planning.

## User and access model

**Users:** Teachers manage their classes and instructional planning; students use the student experience defined by FR-01.

**Product owner:** Maintains scope and resolves changes to this document; this is not automatically an application role.

**Parents/guardians:** Contacts in teacher-managed records; contact association is not an application role.

Authentication and workspace access: [SECURITY.md](SECURITY.md), ACCESS-01–05.

## Functional requirements

| ID | Requirement | Acceptance criteria | Driver |
| --- | --- | --- | --- |
| FR-01 | Support distinct teacher and student user experiences. | After SQ-01–03 are resolved, role-specific acceptance tests enumerate approved student workflows; teacher planning workflows remain available to teachers. Student actions, routes, and fields are TO BE DECIDED until that review. | T-2, T-3, T-23 (SECURITY.md). |
| FR-02 | Present understandable processing notices. | Each approved student workflow links to the current age- and language-appropriate notice; test recipients can identify processing purpose, recipients, and where to request help. Content/authority approval follows SEC-DATA-07. | T-25, T-26. |
| FR-03 | Provide a discoverable institution-approved route for data requests and concerns. | Authorized requesters can submit access/correction/deletion or processing concerns and obtain receipt, status, and outcome; distinguish archive from deletion and explain permitted retention exceptions without exposing others' records. Channel, actors, deadlines: SQ-03 and SQ-05. | T-25, T-28. |
| FR-04 | Show correction context on retained derived recommendations. | After evidence is corrected or retired, related retained recommendations show obsolete/corrected status and an authorized link to current context; DATA-02 history is preserved. | T-21, T-28. |
| FR-05 | Disclose freshness of derived summaries. | When a projection trails its source, the view shows its as-of status; no delayed summary implies a save or acceptance failed. | T-8, T-28. |

### Classes, rosters, and contacts

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| ROSTER-01 | Create, edit, archive, and view classes with name, academic term, subject associations, and timezone. | Archived classes retain history and are excluded from active-class views by default. |
| ROSTER-02 | Maintain students with a stable internal ID, display name, optional external ID, and active/archive status. | Students with the same name remain distinct; renaming a student preserves related records. |
| ROSTER-03 | Enroll students in one or more classes with start/end dates. | Ending enrollment preserves prior attendance, scores, notes, and lesson associations. |
| ROSTER-04 | Maintain optional student contact details and multiple parent/guardian contacts, including relationship, email, phone, and preferred contact method. | A student can have multiple contacts; a contact can be associated with multiple students; missing contact values remain unknown. |
| ROSTER-05 | Import rosters through CSV with field mapping, preview, validation, and explicit confirmation. | Invalid rows are identified; duplicate candidates are shown; records are never merged solely by name; no changes occur before confirmation. |

### Scores, grades, notes, and attendance

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| RECORD-01 | Record assessments with class, subject, title, date, scoring scale, and optional learning-objective or standard links. | Assessment scores retain their scale and are not compared as equivalent across incompatible scales. |
| RECORD-02 | Record and correct student scores and historical grades with academic period and source. | Missing, exempt, and not-yet-assessed values remain distinct from zero; corrections retain prior values in history. |
| RECORD-03 | Display progress by student, subject, objective, and date range. | A teacher can inspect the underlying records for a summary; missing data is shown explicitly. |
| RECORD-04 | Maintain dated teacher notes with optional subject, objective, and lesson links. | Notes record author and timestamps and distinguish teacher observations from accepted AI-authored text. |
| RECORD-05 | Record attendance per class session using present, absent, late, excused, or unrecorded. | Only one current attendance entry exists per student/session; corrections are traceable; unrecorded is not counted as absent. |
| RECORD-06 | Support CSV import/export of scores, historical grades, and attendance. | Imports validate student IDs, class membership, dates, and scales; preview additions and updates; retries cannot silently duplicate records. |

### Student learning profiles

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| PROFILE-01 | Maintain strengths, demonstrated capabilities, support needs, instructional accommodations, and teacher-provided preferences. | Every profile entry has an author/source, recorded date, and optional subject or objective association. |
| PROFILE-02 | Link profile observations to supporting assessments, notes, or teacher-entered evidence where available. | A teacher can open the evidence behind an observation and see when evidence is absent. |
| PROFILE-03 | Allow review, correction, and retirement of profile entries. | Retired entries remain in authorized history but are excluded from new recommendations by default. |
| PROFILE-04 | Keep AI-proposed profile changes separate until accepted. | Teachers can edit or reject proposals; prohibited inferences are governed by SECURITY.md SEC-DATA-09. |

### Subjects, standards, and instructional scope

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| CURR-01 | Organize subjects, units, topics, prerequisites, and learning objectives into a scope and sequence. | Teachers can reorder units and associate them with classes and academic periods. |
| CURR-02 | Store standards with identifier, wording, issuing organization, version/year when known, and source reference. | Unknown metadata remains unknown; AI cannot manufacture an official standard or citation. |
| CURR-03 | Link objectives, assessments, and lesson plans to standards. | Teachers can inspect coverage by class/unit and distinguish planned coverage from teacher-recorded completed coverage. |
| CURR-04 | Preserve the standard text/version used by a lesson revision. | Updating a standard does not silently change the basis of an existing lesson or analysis. |

### Lesson planning and teaching preparation

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| LESSON-01 | Create lessons manually with title, class, subject/unit, objectives, standards, prerequisites, duration, activities, materials, assessment approach, and support strategies. | A teacher can save an incomplete draft and see missing information before marking it ready. |
| LESSON-02 | Draft lesson-preparation checklists from selected standards and teacher context. | Generated items cite selected standards where relevant and remain editable proposals until accepted. |
| LESSON-03 | Draft lesson content and teaching materials, including activity instructions, practice questions, and teacher notes. | The teacher can choose the requested output, review it, edit individual sections, and accept or reject it. |
| LESSON-04 | Manage preparation checklist items with completion status and optional due date. | Teachers can add, reorder, edit, complete, and remove items; completion changes appear in history. |
| LESSON-05 | Support draft, ready, taught, and archived lesson states. | Only a teacher action changes state; taught lessons retain the revision used and allow follow-up reflection. |
| LESSON-06 | Duplicate a lesson into a new draft and print/export a lesson with its checklist. | The copy has a new ID and a source link; export scope follows SECURITY.md SEC-DATA-03. |

### Lesson review and auditing

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| AUDIT-01 | Review a selected lesson revision for objective/standards alignment, missing prerequisites, activity-duration fit, materials, assessment coverage, and planned student support. | Results identify the criterion, relevant lesson section, evidence, and proposed improvement. |
| AUDIT-02 | Distinguish identified concerns from questions caused by incomplete information. | Missing evidence is labeled as insufficient information rather than a proven deficiency. |
| AUDIT-03 | Allow teachers to dismiss findings, record a reason, or accept an editable proposed change. | Findings never modify the lesson automatically; accepted changes create a new revision. |
| AUDIT-04 | Tie each audit to exact input versions. | Editing the lesson or relevant standards marks prior findings stale and offers a fresh review. |

### Cross-reference upcoming lessons with student needs

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| MATCH-01 | Analyze a teacher-selected upcoming lesson or date range against enrolled students' relevant profiles and academic evidence. | The teacher sees which students may need support for specific objectives or prerequisites. |
| MATCH-02 | Explain each match using the lesson objective and dated supporting records. | Each recommendation links to its evidence and identifies insufficient, old, or conflicting evidence without inventing certainty. |
| MATCH-03 | Suggest practical preparation, reinforcement, scaffolding, or extension activities. | The teacher can accept, edit, or reject each suggestion and choose whether to attach it to the lesson. |
| MATCH-04 | Keep recommendations temporary and contextual. | A recommendation does not alter a grade or profile, and is flagged stale when relevant input records change. |
| MATCH-05 | Avoid treating attendance, missing scores, or lack of evidence as proof of low capability. | Test cases containing only absences or missing scores produce insufficient-evidence outcomes rather than ability judgments. |

### Small-group planning

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| GROUP-01 | Suggest groups for an activity using selected participants, activity goals, target group size/count, and a teacher-selected strategy. | Strategies include similar current support needs, mixed demonstrated capabilities, and random grouping. |
| GROUP-02 | Support teacher constraints such as students who must remain together/apart and students excluded from the activity. | Infeasible constraints are explained; the system does not silently ignore them. |
| GROUP-03 | Show a concise rationale and allow manual moves, locked assignments, and regeneration of unlocked assignments. | Each included student appears exactly once, locked assignments persist, and size/constraint violations are clearly shown. |
| GROUP-04 | Save an accepted grouping to a specific lesson/activity. | Proposed groups do not become the active grouping until accepted; groups use neutral names and do not display ability labels. |

### Calendar, notifications, and reminders

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| CAL-01 | Provide day, week, and month views of lessons, class sessions, activities, and preparation deadlines. | Entries link to their underlying records and display in the configured timezone. |
| CAL-02 | Create, move, cancel, and reschedule entries with conflict warnings. | Overlapping events are identified; the teacher decides whether to retain an overlap. |
| CAL-03 | Support recurring class sessions with edits to one occurrence or future occurrences. | Exceptions persist and daylight-saving transitions preserve the intended local class time. |
| NOTIFY-01 | Create in-app reminders for selected events and checklist deadlines. | Teachers choose lead time, dismiss or snooze reminders, and mark them read. |
| NOTIFY-02 | Keep reminders consistent with calendar changes and user preferences. | Rescheduling updates pending reminders; cancellation suppresses them; delivery retries do not create duplicates. |
| NOTIFY-03 | Show reminders missed while signed out on the next visit. | The notification center separates unread, read, snoozed, and overdue items with overdue status independent of read status. |

### Teacher control, versions, and history

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| HISTORY-01 | Keep chronological change history for student records, profiles, lessons, checklists, groups, and schedules. | Authorized teachers can see actor, timestamp, operation, and changed fields with previous/new values where retained. |
| HISTORY-02 | Distinguish human edits, AI proposals, and teacher-accepted AI changes. | An accepted AI change records the approving teacher and the originating proposal. |
| HISTORY-03 | Provide full revision comparison and restoration for lesson plans and teaching materials. | Restoring creates a new current revision and preserves intervening revisions. |
| HISTORY-04 | Correct scores, attendance, and profiles through traceable edits. | Corrections preserve history; there is no bulk restore that silently overwrites unrelated student records. |
| HISTORY-05 | Detect stale writes and proposal acceptance against changed inputs. | A conflict requires comparison or regeneration; the application never silently overwrites a newer version. |

## AI behavior and processing requirements

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| AI-01 | Use an explicit draft-review-accept workflow for generated changes. | Acceptance applies only selected changes; SECURITY.md SEC-AZ-03 owns write enforcement. |
| AI-02 | Record task, input record/version references, generation time, model/provider identifier, and proposal status. | A teacher can determine which information produced a recommendation using input references and concise explanations; internal reasoning traces are not displayed. |
| AI-06 | Handle timeout, provider failure, cancellation, and retry. | Failures preserve teacher work, show a useful status, and do not duplicate accepted changes or consume unbounded retries. |
| AI-07 | State uncertainty and distinguish source facts from generated interpretations. | Unsupported claims and fabricated standards/citations fail the release evaluation. Numeric confidence is not displayed without a validated calibration method. |
| AI-08 | Provide request-size, execution-time, and usage limits with user-visible failure states. | Oversized requests are rejected or deliberately narrowed; processing never silently drops students from a class-wide analysis. |

Proposal lifecycle: draft → review → accepted, partially accepted, or rejected; changed input can mark it stale. Unselected items remain pending unless explicitly rejected.

## Record semantics

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| DATA-01 | Archive records without deleting them. | Archived records leave ordinary active views and remain available in history. |
| DATA-02 | Preserve the evidence behind accepted recommendations. | Later evidence edits do not rewrite historical explanations or evidence references. |
| DATA-03 | Calculate composite grades only from a teacher-defined formula and explicit scale mapping. | No composite or cross-scale equivalence appears until its formula/mapping is defined and inspectable. |

## Interface requirements

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| UI-01 | Provide responsive desktop/tablet workflows and essential compact-screen workflows. | Class setup, record entry, lesson editing, review, and calendar use remain operable at DESIGN.md breakpoints. |
| UI-02 | Show loading, empty, error, permission-denied, and unsaved-change states. | Each major screen exercises these states; interrupted saves retain edits in the active view, offer retry, and warn before navigation. Security-sensitive recovery follows SEC-WEB-02 and SEC-AUTH-06. |
| UI-03 | Provide a task overview and scoped navigation. | Today links to upcoming lessons, incomplete preparation, unread reminders, and pending proposals; settings expose timezone, notification, import/export, AI, language, and account preferences. |
| UI-04 | Make bulk and generated edits deliberate. | Proposal selection begins empty, acceptance previews affected sections, and multi-page selections never silently include other pages. |
| UI-05 | Support deliberate attendance entry. | Opening attendance never defaults students to present; an optional “Mark remaining present” action shows its affected count and supports undo. |
| UI-06 | Support calendar move review and undo. | Rescheduling shows the resulting date/time and permits undo after a successful save. |
| UI-07 | Report import outcomes. | Results distinguish added, updated, invalid, and duplicate-candidate rows with exact counts and a downloadable error report when needed. |

## Operational requirements

| ID | Requirement | Verification |
| --- | --- | --- |
| OPS-01 | Back up persistent records and verify restoration. | Before production, a restore exercise recovers a consistent dataset; recovery-time and recovery-point targets are documented. |
| OPS-02 | Make multi-record acceptance/import operations atomic or show precise, recoverable partial outcomes. | Fault injection produces no unexplained partial updates; retries are idempotent. |

## Quality and measurable targets

The following are proposed initial engineering targets; revise explicitly if deployment constraints require different targets.

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| QUALITY-01 | Meet the accessibility target owned by DESIGN.md DR-A11Y-01. | Retain the automated and manual verification evidence specified there for the complete release. |
| QUALITY-02 | Support a teacher workspace with 20 classes, 600 active students, 2,000 lessons, and 100,000 academic/attendance records. | Representative seeded-load tests exercise dashboards, search, history, and imports at that scale. |
| QUALITY-03 | Keep ordinary reads/saves responsive. | At the reference scale and 25 concurrent active teachers, p95 server response time is under 2 seconds for ordinary paginated reads/saves, excluding AI, bulk imports, and exports; test environment is documented. |
| QUALITY-04 | Run long operations with visible progress and recoverable status. | AI requests show a processing state promptly; a configurable timeout produces a retryable failure without losing edits. |
| QUALITY-05 | Preserve data across refresh, logout/login, and supported deployment migrations. | End-to-end persistence and migration checks retain record relationships, versions, and approval history. |
| QUALITY-06 | Support English (`en`), Spanish (`es`), and Simplified Chinese (`zh-Hans`) throughout teacher-facing workflows. | Navigation, forms, validation, status messages, notifications, exports, print views, and accessibility labels are localized; locale-specific dates, numbers, and times are formatted correctly; a teacher can switch language without changing stored academic values or translating authored content. Missing translations and malformed message placeholders fail release. AI requests distinguish requested output language from original evidence language; translated interpretations never replace source text or represent a translated standard as official. |
| QUALITY-07 | Support an optionally installable app shell. | The app works in an ordinary browser without installation; offline private workflows show unavailable status. Offline synchronization is deferred pending a separate security/conflict design. |

## Legacy scope/workflow references

A-08 refers to ARCHITECTURE.md DEP-01–02. Retired workflow IDs W-01, W-02, W-03, and W-04 refer respectively to ROSTER-01–05; LESSON-01–06 with AUDIT/MATCH/GROUP; RECORD-01–06 with LESSON-05; and HISTORY-01–05. These are navigation aliases, not duplicate requirements.

## Release acceptance and open decisions

The initial release requires every applicable acceptance criterion to pass. AI evaluation uses a repeatable dataset with known standards, incomplete and contradictory evidence, and results recorded by model/configuration version; evaluate AI-07 and MATCH-05. Security evaluation is owned by SECURITY.md SEC-VERIFY-01.

| Decision | Current constraint |
| --- | --- |
| Traditional Chinese | Confirm whether `zh-Hant` is needed in addition to QUALITY-06; obtain education-domain translation review. |
| Scoring/rubrics | Detailed conventions remain open under DATA-03. |
| Recovery and capacity targets | Set production RPO/RTO for OPS-01 and regional availability, traffic peaks, and cost envelope beyond QUALITY-02–03. |

Security production gates: SECURITY.md SEC-DATA-04–06. Provider and regional implementation choices: ARCHITECTURE.md DEP-01.
