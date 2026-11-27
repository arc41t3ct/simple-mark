# Specification Quality Checklist: Markdown Reader (macOS first)

**Purpose**: Validate specification completeness and quality before proceeding to planning
**Created**: 2026-11-27
**Feature**: [spec.md](../spec.md)

## Content Quality

- [x] No implementation details (languages, frameworks, APIs)
- [x] Focused on user value and business needs
- [x] Written for non-technical stakeholders
- [x] All mandatory sections completed

## Requirement Completeness

- [x] No [NEEDS CLARIFICATION] markers remain
- [x] Requirements are testable and unambiguous
- [x] Success criteria are measurable
- [x] Success criteria are technology-agnostic (no implementation details)
- [x] All acceptance scenarios are defined
- [x] Edge cases are identified
- [x] Scope is clearly bounded
- [x] Dependencies and assumptions identified

## Feature Readiness

- [x] All functional requirements have clear acceptance criteria
- [x] User scenarios cover primary flows
- [x] Feature meets measurable outcomes defined in Success Criteria
- [x] No implementation details leak into specification

## Notes

- Iteration 1: 3 open clarifications: restoring the last document, ways to open a file, and file
  changes on disk.
- Iteration 2: all 3 resolved by Andre.
- Iteration 3 (2026-11-27): scope rework from Andre's follow-up messages:
  - macOS only. Windows 11 and Linux moved to `specs/backlog.md` (BL-001, BL-002).
  - Delivery restaged: User Stories 1–3 (all P1) are the minimum viable round trip, and stories
    4–9 are increments on top.
  - Folder opening and a recursive tree with a configurable depth (default 5, range 0–20).
  - A simple persistent Settings view (User Story 4).
  - macOS sandbox and folder-access requirements (FR-027 to FR-032).
  - Resizable left menu (FR-020).
  - Folder toggles whose state is remembered for each tree root across restarts (FR-014a,
    SC-005a).
  - Graceful handling of remembered state that points to deleted or renamed folders and files
    (FR-014b, FR-014c, SC-005b, and 7 new edge cases).
- Requirements deliberately stay in plain language. Apple-specific terms such as entitlements
  and security-scoped bookmarks, and `window_manager`, appear only under "Implementation notes
  for planning" in Assumptions, because Andre named them explicitly. They are deferred to
  `/speckit-plan`.
- CommonMark and the GitHub extensions in FR-003 name the content format the reader supports,
  not an implementation choice.
