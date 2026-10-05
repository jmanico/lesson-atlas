# Teacher Planning Assistant: Requirements

Version: 1.0  
Date: 2026-10-05  
Status: Initial implementation baseline  
Product owner: Jim Manico

## 1. Purpose and authority

Build an application that helps teachers organize student information, prepare standards-aligned lessons, identify individual support needs, plan small groups, and manage classroom schedules. AI assists with drafting and analysis; the teacher controls every accepted change.

This document is the source of truth for product scope, behavior, and acceptance. Implementation plans, designs, code, and tests must reference the requirement IDs below. If another artifact conflicts with this document, update the conflicting artifact or obtain a product-owner decision and revise this document before changing scope.

“Must” denotes a required capability. All requirements apply to the initial complete release unless explicitly deferred. Delivery phases are implementation order, not permission to omit capabilities. Assumptions below are proposed implementation defaults, not previously confirmed user decisions.

## 2. Outcomes and principles

- Reduce the work required to prepare lessons and teaching materials.
- Give teachers an organized view of student progress, attendance, strengths, and support needs.
- Connect upcoming instruction to documented student needs and relevant past evidence.
- Make AI suggestions editable, explainable, attributable, and reversible.
- Preserve teacher judgment and prevent silent changes to educational records.
- Protect student information throughout storage, retrieval, AI processing, exports, and history.

## 3. Initial scope and assumptions

| ID | Proposed default |
| --- | --- |
| A-01 | A responsive, English-language web application for teachers, usable on desktop and tablet. |
| A-02 | Each teacher initially has a private workspace containing multiple classes, subjects, and academic terms. Shared teaching and school administration are deferred. |
| A-03 | Students and parents are contact records, without login accounts or direct access in the initial release. |
| A-04 | Teachers enter or import their own educational standards. No particular country, grade range, curriculum, or standards catalog is assumed. |
| A-05 | Notifications are delivered inside the app. Email, SMS, push notifications, parent messaging, and external calendar synchronization are deferred. |
| A-06 | Manual entry and CSV import cover structured student and academic records. Standards and reference materials can be entered or pasted as text with source metadata. Arbitrary document ingestion is deferred. |
| A-07 | AI is optional for daily recordkeeping. Core manual workflows remain usable when AI is unavailable. |
| A-08 | Technology stack, hosting provider, identity provider, and AI provider remain implementation choices, subject to the requirements below. |

### Explicitly outside the initial release

- Autonomous grading, diagnosis, special-education eligibility decisions, or disciplinary recommendations.
- Automatic changes to student records, lesson plans, groups, or calendars based on AI output.
- Student-facing tutoring, parent/student portals, and direct messaging.
- Student information system or learning management system integrations.
- Billing, district administration, public content sharing, and simultaneous collaborative editing.
- Clinical records or full individualized education plan management. Store only teacher-authorized instructional support information needed for planning.

## 4. User and access model

**Primary user:** A teacher managing their own classes and instructional planning.

**Product owner:** Maintains scope and resolves changes to this document; this is not automatically an application role.

**Students and parents/guardians:** People represented in teacher-managed records. Contact association does not grant application access.

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| ACCESS-01 | Require authenticated access to all workspace data. | An unauthenticated request cannot retrieve, modify, export, or search student or lesson data. |
| ACCESS-02 | Enforce workspace ownership on the server for every record and operation. | A teacher cannot access another workspace through altered IDs, search, exports, attachments, history, or AI requests. |
| ACCESS-03 | Support sign-out and revocation of active sessions. | A revoked session cannot perform subsequent protected operations. |

## 5. Functional requirements

### 5.1 Classes, rosters, and contacts

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| ROSTER-01 | Create, edit, archive, and view classes with name, academic term, subject associations, and timezone. | Archived classes retain history and are excluded from active-class views by default. |
| ROSTER-02 | Maintain students with a stable internal ID, display name, optional external ID, and active/archive status. | Students with the same name remain distinct; renaming a student preserves related records. |
| ROSTER-03 | Enroll students in one or more classes with start/end dates. | Ending enrollment preserves prior attendance, scores, notes, and lesson associations. |
| ROSTER-04 | Maintain optional student contact details and multiple parent/guardian contacts, including relationship, email, phone, and preferred contact method. | A student can have multiple contacts; a contact can be associated with multiple students; contacts are never AI-invented. |
| ROSTER-05 | Import rosters through CSV with field mapping, preview, validation, and explicit confirmation. | Invalid rows are identified; duplicate candidates are shown; records are never merged solely by name; no changes occur before confirmation. |

### 5.2 Scores, grades, notes, and attendance

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| RECORD-01 | Record assessments with class, subject, title, date, scoring scale, and optional learning-objective or standard links. | Assessment scores retain their scale and are not compared as equivalent across incompatible scales. |
| RECORD-02 | Record and correct student scores and historical grades with academic period and source. | Missing, exempt, and not-yet-assessed values remain distinct from zero; corrections retain prior values in history. |
| RECORD-03 | Display progress by student, subject, objective, and date range. | A teacher can inspect the underlying records for a summary; missing data is shown explicitly. |
| RECORD-04 | Maintain dated teacher notes with optional subject, objective, and lesson links. | Notes record author and timestamps and distinguish teacher observations from accepted AI-authored text. |
| RECORD-05 | Record attendance per class session using present, absent, late, excused, or unrecorded. | Only one current attendance entry exists per student/session; corrections are traceable; unrecorded is not counted as absent. |
| RECORD-06 | Support CSV import/export of scores, historical grades, and attendance. | Imports validate student IDs, class membership, dates, and scales; preview additions and updates; retries cannot silently duplicate records. |

### 5.3 Student learning profiles

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| PROFILE-01 | Maintain strengths, demonstrated capabilities, support needs, instructional accommodations, and teacher-provided preferences. | Every profile entry has an author/source, recorded date, and optional subject or objective association. |
| PROFILE-02 | Link profile observations to supporting assessments, notes, or teacher-entered evidence where available. | A teacher can open the evidence behind an observation and see when evidence is absent. |
| PROFILE-03 | Allow review, correction, and retirement of profile entries. | Retired entries remain in authorized history but are excluded from new recommendations by default. |
| PROFILE-04 | Keep AI-proposed profile changes separate until accepted. | AI cannot silently assign diagnoses, fixed ability labels, or support needs; teachers can edit or reject proposals. |

### 5.4 Subjects, standards, and instructional scope

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| CURR-01 | Organize subjects, units, topics, prerequisites, and learning objectives into a scope and sequence. | Teachers can reorder units and associate them with classes and academic periods. |
| CURR-02 | Store standards with identifier, wording, issuing organization, version/year when known, and source reference. | Unknown metadata remains unknown; AI cannot manufacture an official standard or citation. |
| CURR-03 | Link objectives, assessments, and lesson plans to standards. | Teachers can inspect coverage by class/unit and distinguish planned coverage from teacher-recorded completed coverage. |
| CURR-04 | Preserve the standard text/version used by a lesson revision. | Updating a standard does not silently change the basis of an existing lesson or analysis. |

### 5.5 Lesson planning and teaching preparation

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| LESSON-01 | Create lessons manually with title, class, subject/unit, objectives, standards, prerequisites, duration, activities, materials, assessment approach, and support strategies. | A teacher can save an incomplete draft and see missing information before marking it ready. |
| LESSON-02 | Draft lesson-preparation checklists from selected standards and teacher context. | Generated items cite selected standards where relevant and remain editable proposals until accepted. |
| LESSON-03 | Draft lesson content and teaching materials, including activity instructions, practice questions, and teacher notes. | The teacher can choose the requested output, review it, edit individual sections, and accept or reject it. |
| LESSON-04 | Manage preparation checklist items with completion status and optional due date. | Teachers can add, reorder, edit, complete, and remove items; completion changes appear in history. |
| LESSON-05 | Support draft, ready, taught, and archived lesson states. | Only a teacher action changes state; taught lessons retain the revision used and allow follow-up reflection. |
| LESSON-06 | Duplicate a lesson into a new draft and print/export a lesson with its checklist. | The copy has a new ID and a source link; student-specific notes are excluded from exports unless deliberately included. |

### 5.6 Lesson review and auditing

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| AUDIT-01 | Review a selected lesson revision for objective/standards alignment, missing prerequisites, activity-duration fit, materials, assessment coverage, and planned student support. | Results identify the criterion, relevant lesson section, evidence, and proposed improvement. |
| AUDIT-02 | Distinguish identified concerns from questions caused by incomplete information. | Missing evidence is labeled as insufficient information rather than a proven deficiency. |
| AUDIT-03 | Allow teachers to dismiss findings, record a reason, or accept an editable proposed change. | Findings never modify the lesson automatically; accepted changes create a new revision. |
| AUDIT-04 | Tie each audit to exact input versions. | Editing the lesson or relevant standards marks prior findings stale and offers a fresh review. |

### 5.7 Cross-reference upcoming lessons with student needs

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| MATCH-01 | Analyze a teacher-selected upcoming lesson or date range against enrolled students' relevant profiles and academic evidence. | The teacher sees which students may need support for specific objectives or prerequisites. |
| MATCH-02 | Explain each match using the lesson objective and dated supporting records. | Each recommendation links to its evidence and identifies insufficient, old, or conflicting evidence without inventing certainty. |
| MATCH-03 | Suggest practical preparation, reinforcement, scaffolding, or extension activities. | The teacher can accept, edit, or reject each suggestion and choose whether to attach it to the lesson. |
| MATCH-04 | Keep recommendations temporary and contextual. | A recommendation does not alter a grade or profile, and is flagged stale when relevant input records change. |
| MATCH-05 | Avoid treating attendance, missing scores, or lack of evidence as proof of low capability. | Test cases containing only absences or missing scores produce insufficient-evidence outcomes rather than ability judgments. |

### 5.8 Small-group planning

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| GROUP-01 | Suggest groups for an activity using selected participants, activity goals, target group size/count, and a teacher-selected strategy. | Strategies include similar current support needs, mixed demonstrated capabilities, and random grouping. |
| GROUP-02 | Support teacher constraints such as students who must remain together/apart and students excluded from the activity. | Infeasible constraints are explained; the system does not silently ignore them. |
| GROUP-03 | Show a concise rationale and allow manual moves, locked assignments, and regeneration of unlocked assignments. | Each included student appears exactly once, locked assignments persist, and size/constraint violations are clearly shown. |
| GROUP-04 | Save an accepted grouping to a specific lesson/activity. | Proposed groups do not become the active grouping until accepted; groups use neutral names and do not display ability labels. |

### 5.9 Calendar, notifications, and reminders

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| CAL-01 | Provide day, week, and month views of lessons, class sessions, activities, and preparation deadlines. | Entries link to their underlying records and display in the configured timezone. |
| CAL-02 | Create, move, cancel, and reschedule entries with conflict warnings. | Overlapping events are identified; the teacher decides whether to retain an overlap. |
| CAL-03 | Support recurring class sessions with edits to one occurrence or future occurrences. | Exceptions persist and daylight-saving transitions preserve the intended local class time. |
| NOTIFY-01 | Create in-app reminders for selected events and checklist deadlines. | Teachers choose lead time, dismiss or snooze reminders, and mark them read. |
| NOTIFY-02 | Keep reminders consistent with calendar changes and user preferences. | Rescheduling updates pending reminders; cancellation suppresses them; delivery retries do not create duplicates. |
| NOTIFY-03 | Show reminders missed while signed out on the next visit. | The notification center separates unread, read, snoozed, and overdue items without exposing student details outside the authenticated app. |

### 5.10 Teacher control, versions, and history

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| HISTORY-01 | Keep chronological change history for student records, profiles, lessons, checklists, groups, and schedules. | Authorized teachers can see actor, timestamp, operation, and changed fields with previous/new values where retained. |
| HISTORY-02 | Distinguish human edits, AI proposals, and teacher-accepted AI changes. | An accepted AI change records the approving teacher and the originating proposal. |
| HISTORY-03 | Provide full revision comparison and restoration for lesson plans and teaching materials. | Restoring creates a new current revision and preserves intervening revisions. |
| HISTORY-04 | Correct scores, attendance, and profiles through traceable edits. | Corrections preserve history; there is no bulk restore that silently overwrites unrelated student records. |
| HISTORY-05 | Detect stale writes and proposal acceptance against changed inputs. | A conflict requires comparison or regeneration; the application never silently overwrites a newer version. |
| HISTORY-06 | Protect history with the same access restrictions as current records and apply approved retention/deletion policy. | Historical content cannot bypass access restrictions or reintroduce data already purged under policy. |

## 6. AI behavior and processing requirements

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| AI-01 | Use an explicit draft-review-accept workflow for generated changes. | Generation cannot mutate authoritative educational records; acceptance applies only the selected changes. |
| AI-02 | Record task, input record/version references, generation time, model/provider identifier, and proposal status. | A teacher can determine which information produced a recommendation without exposing secrets or internal reasoning traces. |
| AI-03 | Use only relevant, authorized workspace data for student-specific analysis. | The request includes the minimum necessary fields; contact details are excluded; retrieval enforces workspace boundaries before model access. |
| AI-04 | Treat notes, standards, and pasted materials as untrusted input. | Embedded instructions cannot authorize actions, reveal other records, bypass approvals, or expand tool permissions. |
| AI-05 | Validate generated output before displaying or applying it. | Invalid record references, malformed fields, unsafe markup, or unsupported changes are rejected or clearly surfaced for correction. |
| AI-06 | Handle timeout, provider failure, cancellation, and retry. | Failures preserve teacher work, show a useful status, and do not duplicate accepted changes or consume unbounded retries. |
| AI-07 | State uncertainty and distinguish source facts from generated interpretations. | Unsupported claims and fabricated standards/citations fail the release evaluation. Numeric confidence is not displayed without a validated calibration method. |
| AI-08 | Provide request-size, execution-time, and usage limits with user-visible failure states. | Oversized requests are rejected or deliberately narrowed; processing never silently drops students from a class-wide analysis. |

Proposal lifecycle: generated draft → teacher review → accepted, partially accepted, or rejected. A superseded input can make a proposal stale. Acceptance must verify the proposal's permissions and input versions again on the server.

## 7. Core data model

These are logical entities, not a prescribed database schema. Every owned entity must have a stable ID, workspace association, timestamps, and version information where mutable.

| Entity | Required relationships and important fields |
| --- | --- |
| Workspace / Teacher | Owner, timezone, preferences, notification settings. |
| AcademicTerm / Class / Enrollment | Term dates, class subjects, student membership and effective dates. |
| Student / Contact / StudentContact | Student status, optional external ID, contact relationship and communication fields. |
| Subject / Unit / LearningObjective | Scope order, prerequisites, standard associations. |
| Standard / StandardVersion | Identifier, exact text, source, issuing organization, version metadata. |
| Assessment / Score / HistoricalGrade | Student, subject, period, scale, status, evidence date, source. |
| ClassSession / Attendance | Session time, enrolled student, attendance status, correction history. |
| TeacherNote / ProfileEntry | Student, author, evidence links, subject/objective, active/retired status. |
| Lesson / LessonRevision / TeachingMaterial | Class, objectives, standard versions, content, lifecycle state, revision source. |
| ChecklistItem | Lesson, description, order, completion, due date. |
| ReviewFinding / SupportRecommendation | Exact lesson and evidence versions, explanation, proposed change, disposition. |
| ActivityGroup / GroupMembership | Lesson/activity, selected participants, constraints, strategy, accepted assignments. |
| CalendarEvent / Reminder / Notification | Source record, recurrence, timezone, delivery/deduplication state. |
| AIProposal / ChangeEvent | Input versions, provenance, approval state, actor, changed fields, related revision. |

Data rules:

- Historical scores must retain original scales and periods. Do not infer grade equivalence without an explicitly defined mapping.
- All student-specific evidence must belong to the same workspace and reference a valid student.
- Timestamps represent absolute instants; calendar recurrence also stores the intended local timezone.
- Archiving hides records from ordinary active views; it does not delete them.
- Deletion and retention must cover current records, historical versions, derived recommendations, exports retained by the service, and provider-held copies where applicable.
- Accepted recommendations retain evidence references; later evidence changes must not rewrite historical explanations.

## 8. Primary workflows

### W-01: Set up a class

1. Teacher signs in and creates a term and class.
2. Teacher adds subjects and students, manually or through a previewed import.
3. Teacher adds contacts and initial grades, observations, and support needs where available.
4. The app shows incomplete records without inventing missing information.

### W-02: Prepare a lesson

1. Teacher selects the class, objectives, standards, duration, and planned date.
2. Teacher writes a lesson or requests an AI draft and preparation checklist.
3. Teacher reviews, edits, and accepts selected content.
4. Teacher requests a lesson audit and student-support analysis.
5. Teacher accepts appropriate improvements and optionally creates activity groups.
6. Teacher marks the lesson ready, schedules it, and sets preparation reminders.

### W-03: Record teaching and follow up

1. Teacher records attendance, scores, and observations.
2. Teacher marks the lesson taught and adds a reflection.
3. Teacher reviews new support suggestions before updating any profile.
4. Future lesson analysis uses the updated evidence and identifies outdated suggestions.

### W-04: Inspect and correct an edit

1. Teacher opens history for a lesson or student record.
2. Teacher compares changes and identifies human/AI provenance.
3. Teacher restores a lesson revision or corrects an academic record.
4. The app creates a traceable new change and flags affected analysis as stale.

## 9. Interface requirements

- Dashboard: upcoming lessons, incomplete preparation, unread reminders, and pending AI proposals.
- Classes: roster, subject filters, attendance entry, assessments, and progress summaries.
- Student detail: learning profile, scores/grades, attendance, notes, contacts, and authorized history.
- Curriculum: subjects, scope and sequence, standards, objectives, and coverage views.
- Lesson workspace: editor, checklist, teaching materials, audit findings, student support, groups, and revisions.
- Calendar: schedule views, recurrence, deadlines, and reminder controls.
- Settings: timezone, notifications, data import/export, AI processing settings, and account controls.

All major screens must provide loading, empty, error, permission-denied, and unsaved-change states. Teacher-authored drafts may autosave with a visible save indicator; AI proposals remain separate until accepted. Navigation or service failure must not silently discard unsaved work.

## 10. Security, privacy, and operational requirements

These are product requirements, not a claim of legal certification. Applicable jurisdiction and institutional obligations must be determined before processing real student data.

| ID | Requirement | Verification |
| --- | --- | --- |
| SEC-01 | Encrypt data in transit and at rest; keep secrets server-side and out of source control, browser bundles, and logs. | Deployment/configuration review and negative tests for secret exposure. |
| SEC-02 | Apply server-side authorization to every operation, including background jobs, AI retrieval, exports, and history. | Automated cross-workspace access tests for each resource type. |
| SEC-03 | Validate inputs, encode rendered output, parameterize database access, and protect authenticated mutations against applicable web attacks. | Security tests cover stored script payloads, unauthorized object references, forged requests, and malformed imports. |
| SEC-04 | Exclude student content and contact information from routine telemetry and error logs. | Representative logs and error reports contain identifiers needed for operations without educational content. |
| SEC-05 | Require an approved AI-provider data-use and retention configuration before sending real student information. Student data must not be used for model training. | Configuration documents permitted fields, retention, processing location, deletion behavior, and provider terms. AI stays disabled for real data until this is satisfied. |
| SEC-06 | Support authorized data export, archival, and deletion according to an explicit retention policy. | Policy includes historical data and backups; deletion jobs are verified; backup recovery reapplies required deletions. |
| SEC-07 | Use synthetic data in demos and automated testing. | Seed fixtures contain no real student identities or contact information. |
| SEC-08 | Protect CSV exports from spreadsheet formula execution and restrict exported fields to the selected scope. | Dangerous cell prefixes are neutralized; unauthorized records never appear in exports. |
| OPS-01 | Back up persistent records and verify restoration. | Before production, a restore exercise recovers a consistent dataset; recovery-time and recovery-point targets are documented. |
| OPS-02 | Make multi-record acceptance/import operations atomic or show precise, recoverable partial outcomes. | Fault injection produces no unexplained partial updates; retries are idempotent. |

## 11. Quality and measurable targets

The following are proposed initial engineering targets; revise explicitly if deployment constraints require different targets.

| ID | Requirement | Acceptance criteria |
| --- | --- | --- |
| QUALITY-01 | Support keyboard use, labeled controls, visible focus, readable contrast, and accessible form errors. | Core workflows pass keyboard and screen-reader checks; target WCAG 2.2 AA and verify against the authoritative standard during implementation. |
| QUALITY-02 | Support a teacher workspace with 20 classes, 600 active students, 2,000 lessons, and 100,000 academic/attendance records. | Representative seeded-load tests exercise dashboards, search, history, and imports at that scale. |
| QUALITY-03 | Keep ordinary reads/saves responsive. | At the reference scale and 25 concurrent active teachers, p95 server response time is under 2 seconds for ordinary paginated reads/saves, excluding AI, bulk imports, and exports; test environment is documented. |
| QUALITY-04 | Run long operations with visible progress and recoverable status. | AI requests show a processing state promptly; a configurable timeout produces a retryable failure without losing edits. |
| QUALITY-05 | Preserve data across refresh, logout/login, and supported deployment migrations. | End-to-end persistence and migration checks retain record relationships, versions, and approval history. |

## 12. Delivery sequence and release acceptance

| Phase | Scope | Exit condition |
| --- | --- | --- |
| 1. Recordkeeping foundation | Authentication, workspace boundaries, classes, rosters, contacts, records, profiles, and change history. | Teachers can maintain records manually; authorization and history tests pass. |
| 2. Planning foundation | Curriculum, lesson editor, checklists, materials, revisions, calendar, and in-app reminders. | A lesson can be prepared, scheduled, taught, printed, and restored without AI. |
| 3. AI assistance | Drafting, audits, student-support matching, and small-group suggestions. | Every AI path enforces review/acceptance, evidence links, stale-input handling, and failure recovery. |
| 4. Complete initial release | Imports/exports, accessibility, performance, privacy controls, backup recovery, and end-to-end verification. | All requirements and release scenarios pass, or exceptions are explicitly approved and recorded here. |

Required end-to-end release scenarios:

1. Import a roster containing duplicate names and invalid rows; preview and resolve without accidental merging.
2. Record absent, missing-score, zero-score, and exempt cases; verify that summaries distinguish them.
3. Draft a checklist from known standards; reject all suggestions and confirm that authoritative content is unchanged.
4. Accept selected lesson edits, inspect their provenance, compare revisions, and restore an earlier revision.
5. Match an upcoming lesson to a documented student need; inspect the source evidence and accept a support activity.
6. Analyze a student with no relevant evidence; show insufficient information without an inferred deficit.
7. Build groups with locked students and conflicting constraints; preserve locks and explain infeasibility.
8. Reschedule a recurring lesson across a daylight-saving change; preserve local time and avoid duplicate reminders.
9. Change evidence or a lesson after generating a proposal; prevent blind acceptance of the stale result.
10. Simulate provider failure, import retry, and interrupted save; preserve work and avoid duplicate mutations.
11. Attempt access from a different teacher account, including history, exports, and AI retrieval; deny access.
12. Exercise approved retention/deletion and backup restoration without restoring access to purged student content.

AI evaluation must use a repeatable synthetic dataset with known standards, student evidence, incomplete records, contradictory evidence, and malicious embedded instructions. Release checks must include fabricated citations, unsupported capability claims, unauthorized data disclosure, and unapproved mutations. Record evaluation results by model/configuration version.

## 13. Decisions to resolve before production

These do not block a synthetic-data prototype. Record each decision here before activating the affected production capability.

| Decision | Current default or constraint |
| --- | --- |
| Target ages, school types, and jurisdiction | Unspecified; no jurisdiction-specific compliance claim. |
| Institution authorization and data stewardship | Must be established before loading real student information. |
| Retention, deletion, backup retention, and recovery targets | Must be explicitly configured before production. |
| Identity, hosting, region, and AI provider | Unselected; must satisfy Sections 6 and 10. |
| Standards catalogs and content licensing | Teacher-provided text/source metadata initially; licensed integrations deferred. |
| Shared teaching and school administration | Deferred; no implicit sharing between teacher workspaces. |
| External messaging and calendar integrations | Deferred; in-app notifications and calendar initially. |
| Detailed scoring/rubric conventions | Preserve entered scales; calculated composite grades require an explicitly defined teacher-controlled formula. |

## 14. Change control and definition of done

- Every implementation issue and meaningful acceptance test must reference the requirement IDs it satisfies.
- Changes to scope or behavior must update this document, its version, and the change log before being treated as the new baseline.
- Architecture and design documents may explain implementation choices but may not silently change these requirements.
- A feature is complete when its acceptance criteria pass, persistence and authorization work, failure states are handled, and applicable history/AI-approval behavior is verified.
- The application is ready for production only when the full release criteria pass and all production-gating decisions in Section 13 are resolved.

### Change log

| Version | Date | Change |
| --- | --- | --- |
| 1.0 | 2026-10-05 | Initial source of truth derived from the project idea and categorized feature list; includes proposed build defaults, acceptance criteria, and production decisions. |
