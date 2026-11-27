# AGENTS.md

## Purpose

SimpleMark is a Flutter desktop reader that makes reading and reviewing Markdown comfortable. It shows a
rendered document next to a file tree. It is built macOS first, with Windows 11 and Linux to follow, and its work
is driven by Spec Kit. See [README.md](./README.md).

## Commands

```fish
fvm flutter pub get
fvm flutter run -d macos
fvm flutter test --test-randomize-ordering-seed=random
fvm flutter analyze --fatal-infos
make quality-gate
make docs-gate
okf repo . --check agent-context
```

## Where to look

| When | Read |
| --- | --- |
| Before any commit or branch operation | [Git rules](./../ark-ai-agent-knowledge-base/docs/rules/git-rules.md) |
| Before handing off any change | [Software quality rules](./../ark-ai-agent-knowledge-base/docs/rules/software-quality-rules.md) |
| Before writing or editing Markdown | [Markdown writing rules](./../ark-ai-agent-knowledge-base/docs/rules/markdown-writing-rules.md) |
| Before writing or restructuring documentation | [Documentation standards](./../ark-ai-agent-knowledge-base/docs/architecture/standards/documentation/index.md) |
| When a document here deviates from the standards | [Self-healing adoption](./../ark-ai-agent-knowledge-base/docs/architecture/standards/documentation/documentation-standard.md#self-healing-adoption) |
| Before any design, plan, task, or code change | [Constitution](./.specify/memory/constitution.md) |
| Before changing an interface, file layout, or module boundary | [Constitution, Principles I–II](./.specify/memory/constitution.md#core-principles) |
| Before implementing or changing behavior of feature 001 | [Markdown reader spec](./specs/001-markdown-reader/spec.md) |
| When you find work outside the current feature's scope | [Backlog](./specs/backlog.md) |
| Before writing Dart code | [Flutter coding rules](./../ark-ai-agent-knowledge-base/docs/rules/flutter-coding-rules.md) |
| Before starting implementation | [Flutter workflow](./../ark-ai-agent-knowledge-base/docs/workflows/flutter-workflow.md) |
| Before adding or arranging feature folders | [Flutter project structure](./../ark-ai-agent-knowledge-base/docs/technologies/flutter/project_structure.md) |
| Before building UI or theming | [ForUI guide](./../ark-ai-agent-knowledge-base/docs/technologies/flutter/ui_library.md) |
| Before inspecting the running app | [Marionette MCP guide](./../ark-ai-agent-knowledge-base/docs/technologies/flutter/marionette-mcp.md) |

## Knowledge base

Load this file first, then discover task literature through OKF. Do not walk `index.md` files. From this
repository:

```fish
okf query --root ../ark-ai-agent-knowledge-base --repository simple-mark --search "{subject}"
okf related {document-id} --root ../ark-ai-agent-knowledge-base --depth all
```

Replace `{subject}` with your topic and `{document-id}` with an ID returned by `okf query`. Check freshness,
provenance, and scope in the result before opening bodies.

## Conventions

- The constitution wins over the KB Flutter rules where they conflict. The known case: a feature's
  `repositories/` folder is split into `ports/` (interfaces) and `adapters/` (implementations).
- Commit to `master` and push when a Spec Kit step or task is complete. Andre works synchronously here, so no
  feature branches are used, even though Spec Kit names `NNN-*` branches in specs.
- Change scope only through Spec Kit: `/speckit-clarify` or `/speckit-specify` for requirements,
  `/speckit-constitution` for rules. Out-of-scope findings become `BL-NNN` entries in the backlog and are not
  added to the current spec.
- macOS first: Windows and Linux work is out of scope unless a spec names it. Platform code lives only in
  adapters, so a new platform never changes `models/`, `ports/`, `use_cases/`, or `state/`.
- The macOS App Sandbox stays on. Change `macos/Runner/*.entitlements` only to match the spec's permission
  requirements (FR-027 to FR-032, FR-044). Never add outgoing network access to the Release entitlements: the
  reader is offline by design.
- `.specify/feature.json` is git-ignored local state that points Spec Kit commands at the active feature.
- Generated localization files are never edited by hand. Regenerate them with `fvm flutter gen-l10n` after
  changing ARB files.
- Golden files under `test/` are regenerated with `fvm flutter test --update-goldens` only when a rendering change
  is intended. Name that change in the handoff.

## Runtime UI inspection

Debug entry point: `lib/main.dart`. Start the app for inspection with `fvm flutter run -d macos`, then connect
Marionette to the VM service URI that the run prints.

## Documentation

| When you change | Update |
| --- | --- |
| A requirement, user story, or scope | The feature's `spec.md` through `/speckit-clarify`, or a new feature through `/speckit-specify` |
| A project rule or quality gate | `.specify/memory/constitution.md` through `/speckit-constitution` |
| Deferred or out-of-scope work | Add or update a `BL-NNN` entry in `specs/backlog.md` |
| Supported platforms | `README.md` and the constitution's "Target platforms" entry |
| Entitlements or registered document types | The plan's permission section and the spec requirement it satisfies |
| ARB translation files | Run `fvm flutter gen-l10n` |
| A user-visible behavior | Add a line under `Unreleased` in `CHANGELOG.md` |
| This agent file | Run `okf repo . --check agent-context` |

## Stories

Spec Kit owns the stories here: each feature's user stories are in `specs/NNN-<name>/spec.md` and its tasks in
`tasks.md`. Deferred work is in the [backlog](./specs/backlog.md).
