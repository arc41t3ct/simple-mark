# SimpleMark Constitution

SimpleMark is a Flutter application that makes reading and reviewing Markdown comfortable.
This constitution governs how its code is structured, tested, and maintained.

## Core Principles

### I. Feature-First Layout

The codebase is organized by product feature, not by technical layer.

- Every feature lives in `lib/src/<feature>/` and owns its models, ports, use cases, state,
  adapters, pages, and widgets. A feature includes only the subfolders it needs.
- The canonical feature layout is:

  ```text
  lib/src/<feature>/
  ├── <feature>.dart      # Root barrel: the feature's only public surface
  ├── models/             # Entities, value objects, sealed results (pure Dart)
  ├── ports/              # abstract interface classes the feature depends on (pure Dart)
  ├── use_cases/          # Optional orchestration across ports (pure Dart)
  ├── state/              # Cubits/Blocs and their states
  ├── adapters/           # Implementations of ports (file system, parser, platform)
  ├── pages/              # Routed screens
  └── widgets/            # Feature-local widgets
  ```

- `lib/src/app/` contains only bootstrapping and wiring: the composition root (`AppState`),
  the router, localization, and the root widget. It MUST NOT hold business logic.
- `lib/src/core/` holds only code that two or more features genuinely share (app-wide
  widgets, shared state, shared models, and validation). Code moves into `core/` when a
  second feature needs it, not before.
- Each subfolder exposes a barrel file. Only symbols used outside that folder are exported.
- Inside a feature, imports are relative. Across features, imports MUST go through the target
  feature's root barrel (`package:simple_mark/src/<feature>/<feature>.dart`). Importing an
  individual file from another feature is forbidden.
- Feature dependencies MUST be acyclic. If two features need each other, the shared concept
  moves to `core/` or into a new feature.
- Tests mirror the layout under `test/src/<feature>/`.

**Rationale**: a feature-first layout keeps a change inside one folder, keeps agent and human
context small, and makes deleting a feature a single-directory operation.

### II. Ports and Adapters Behind the Clean Dependency Rule

Dependencies point inward. Business rules never depend on frameworks, I/O, or UI.

- **Layers per feature** (inner → outer): `models` → `ports` → `use_cases` → `state` →
  `adapters` / `pages` / `widgets`.
- `models/`, `ports/`, and `use_cases/` MUST be pure Dart. They MUST NOT import
  `package:flutter/*`, `dart:io`, `dart:ui`, platform plugins, or any adapter.
- Every interaction with the outside world (file system, Markdown parsing engine, clipboard,
  window and platform channels, persistence, network) MUST be reached through a port declared
  as an `abstract interface class` in `ports/`. The concrete class lives in `adapters/`.
- Cubits/Blocs and use cases depend on ports only, never on adapter classes.
- Adapters translate external types into feature models at the boundary. Third-party types
  (for example a parser's AST nodes or a plugin's result objects) MUST NOT leak past an
  adapter.
- Wiring is done only in the composition root (`lib/src/app/app_state.dart`) through
  constructor injection and `RepositoryProvider` / `BlocProvider`. Service locators and
  global singletons are forbidden.
- A use case class is created only when logic spans more than one port or is reused by more
  than one Cubit/Bloc. Otherwise the Cubit/Bloc calls the port directly. Pass-through use
  cases are forbidden.
- Data crosses layers as immutable models. Function inputs at port boundaries are validated,
  and repeated validation lives in shared validators in `core/`.

**Rationale**: ports keep the domain testable without Flutter. They also let an engine such as
the Markdown parser or the storage backend be replaced without touching business rules. The
use-case rule keeps the benefits of Clean Architecture without per-call ceremony.

### III. Predictable State with flutter_bloc

All application and feature state is managed with `flutter_bloc`.

- Use `Cubit` by default. Use `Bloc` only when event semantics are needed: event
  transformers (debounce, throttle, `restartable`, `droppable`), replayable events, or
  traceable event logs. The plan MUST record why a `Bloc` was chosen.
- State classes are handwritten, immutable, and extend `Equatable`. A state uses a sealed
  hierarchy or an explicit status enum. Mutable fields, `late` fields, and nullable flags
  that encode status are forbidden.
- Cubits/Blocs contain presentation logic only. They MUST NOT import widgets, a
  `BuildContext`, or adapters, and they MUST NOT perform I/O except through ports.
- Widgets rebuild only for the fields they read. Use `BlocSelector`, `context.select`,
  `buildWhen`, and `listenWhen` with named predicate functions, and use per-item selectors
  in lists. Watching a whole state for one field is forbidden.
- Selectors MUST NOT assert (`!`) or throw. They select nullable values, and `build`
  unwraps them.
- Side effects that the UI handles once (navigation, snackbars, dialogs) go through
  `BlocListener`, never through `build`.
- Every Cubit/Bloc closes its subscriptions and resources in `close()`.
- No other state-management library (Provider-based ChangeNotifier state, Riverpod, MobX,
  GetX) is introduced. `Provider` is used only as the dependency-injection mechanism that
  `flutter_bloc` builds on.

**Rationale**: one state model makes behavior predictable and reviewable. Equatable states make
rebuilds and tests deterministic. Narrow selectors keep large Markdown documents responsive.

### IV. Test-First, Layered Testing (NON-NEGOTIABLE)

Tests define behavior before code exists, and each layer is tested at its own boundary.

- **Red-Green-Refactor**: for models, use cases, and Cubits/Blocs, a failing test MUST be
  written and observed to fail before the implementation is written.
- **Every bug fix** starts with a regression test that reproduces the defect.
- **Unit tests** cover all models, validators, use cases, and Cubits/Blocs (with `bloc_test`).
  Ports are replaced by in-memory fakes in `test/.../fakes/`, or by `mocktail` mocks when
  verifying an interaction is the point of the test.
- **Port contract tests**: every port has one shared contract test suite that runs against
  every adapter and against the fake, so the fake cannot drift from real behavior.
- **Widget tests** cover every page and every widget with conditional rendering or user
  interaction, including empty, loading, error, and large-document states.
- **Golden tests** cover Markdown rendering output for a maintained corpus of fixture
  documents in `test/fixtures/markdown/` (headings, tables, code blocks, nested lists,
  links, images, front matter, and malformed input).
- **Integration tests** (`integration_test/`) cover each user journey in the spec on at
  least one supported platform.
- **Coverage**: `models/`, `ports/`, `use_cases/`, and `state/` MUST reach at least 90% line
  coverage. The whole of `lib/` MUST reach at least 80%. Generated code is excluded
  consistently. Thresholds are never lowered to pass a gate.
- Tests use `package:flutter_test/flutter_test.dart`, run in randomized order, and record
  the seed. A flaky test is a defect and is fixed, not retried or skipped.
- Interactive controls carry stable `ValueKey<String>` keys so tests and runtime
  verification never depend on display text.

**Rationale**: the pure-Dart inner layers make test-first cheap. Contract tests make fakes
trustworthy. Golden tests catch rendering regressions that unit tests cannot see.

### V. DRY With Deliberate Abstraction

Every piece of knowledge has one authoritative representation. Duplication of *knowledge* is
removed, and coincidental similarity is not.

- Business rules, validation, constants, theme values, and Markdown-handling logic each have
  exactly one source in the codebase.
- Extraction follows the rule of three: the second occurrence is noted, and the third is
  extracted. Knowledge duplication (the same rule in two places) is extracted at the second
  occurrence.
- Shared code goes to the narrowest owner: the same feature first, then `core/`, then a
  separate package only when another repository needs it.
- An abstraction MUST have a current second consumer or a port boundary that justifies it.
  Speculative generality (interfaces with one implementation and no boundary, configuration
  for unrequested variants) is forbidden.
- UI is composed from ForUI primitives and theme values. App layout constants are centralized
  in `core/`. Magic numbers and hard-coded colors in widgets are forbidden.
- Test setup is shared through builders and fixtures, never copied between test files.

**Rationale**: DRY reduces the cost of change only when it targets the same knowledge.
Premature abstraction creates coupling that is harder to undo than duplication.

### VI. Honest Quality Gates

Static analysis and gates report evidence. They are never weakened to produce a green result.

- `analysis_options.yaml` enables `strict-casts`, `strict-inference`, and
  `strict-raw-types`, plus a reviewed strict lint set that includes
  `public_member_api_docs`. Analyzer infos are failures.
- The codebase has zero analyzer, lint, and format findings in `lib/` and `test/`.
- `// ignore:` and `// ignore_for_file:` are allowed only for generated code or a confirmed
  false positive, on the narrowest line, with a written reason, and they are reported in
  the handoff.
- Never add fallbacks, silent defaults, swallowed exceptions, or retries that hide failures.
  Errors are modeled as typed results or typed exceptions at port boundaries and logged
  where they happen, with the underlying cause.
- A boundary check (script or `import_lint`-style rule) MUST fail the gate when an inner
  layer imports Flutter, `dart:io`, or an adapter, or when a feature imports another
  feature's internal files.

**Rationale**: a gate that can be bent stops being evidence. Typed errors keep failure paths
as testable as success paths.

### VII. Small, Documented, Disposable Code

The codebase stays small enough to be understood quickly by humans and agents.

- Source and test files MUST NOT exceed 500 lines. Split by responsibility before the limit
  is reached.
- Functions do one thing. Control flow uses full `if` blocks and no one-line `if`
  statements. Lines are at most 80 characters.
- Every public member has a `///` doc comment that states intent, not mechanics.
- Dead code, unused files, unused dependencies, and commented-out code are removed in the
  same change that makes them obsolete.
- When this constitution changes, code touched by a later change is brought into compliance
  in that change.
- User-facing strings are localized through ARB files and `flutter gen-l10n`. Literal
  user-facing strings in widgets are forbidden.

**Rationale**: small, documented units keep reviews fast and context windows low, and they let
features be replaced instead of patched.

## Technology Stack & Constraints

- **SDK**: latest stable Flutter and its bundled Dart SDK, managed through FVM (Flutter
  Version Manager), with no fixed SDK pin, following the Arka version policy.
- **State management**: `flutter_bloc` with `bloc_test` (Principle III).
- **Value equality**: `equatable` with handwritten states and models. Code generators for
  state (`freezed` and similar) are not used unless an amendment approves them.
- **Navigation**: `go_router`, configured only in `lib/src/app/app_router.dart`.
- **UI components**: ForUI for primitives and theming.
- **Testing**: `flutter_test`, `bloc_test`, `mocktail`, golden tests, and
  `integration_test`.
- **Target platforms**: TODO(TARGET_PLATFORMS) — to be decided and recorded in `README.md`.
- **Untrusted content**: every Markdown document is treated as untrusted input. Raw HTML is
  not executed. Links open only after an explicit user action and only for allow-listed
  schemes (`http`, `https`, `mailto`, and relative file links). Remote images are not
  loaded without user consent. Parsing has bounded time and memory so a malformed or
  very large file cannot freeze the UI; heavy parsing runs off the UI isolate.
- **Privacy**: SimpleMark processes documents locally. No telemetry or network call is added
  without a spec that states its purpose and legal basis under the GDPR (General Data
  Protection Regulation), in line with ArkaLogic's sovereign European tech stance.
- **Dependencies**: each new package needs a stated reason in the plan, an active maintainer,
  and a compatible license. It is wrapped behind a port when it touches I/O or parsing.
- **Scripts**: repository automation is Bash (`.sh`) or Dart. Python scripts are not stored.

## Development Workflow & Quality Gates

- **Spec-driven flow**: features go through `/speckit-specify` → `/speckit-clarify` →
  `/speckit-plan` → `/speckit-tasks` → `/speckit-implement`. The plan's Constitution Check
  MUST list every principle and either confirm compliance or record a justified exception
  in its Complexity Tracking table.
- **Task ordering**: generated tasks put test tasks before implementation tasks for models,
  use cases, and state (Principle IV).
- **Local gate**: the repository exposes `make quality-gate`, which runs in order:
  1. `fvm dart format --output=none --set-exit-if-changed lib test`
  2. `fvm flutter analyze --fatal-infos`
  3. the layer and feature boundary check (Principle VI)
  4. the file-length check
  5. `fvm flutter test --coverage --test-randomize-ordering-seed=random`, with coverage
     thresholds enforced (Principle IV)
  6. dependency validation, the Trivy security scan, and the documentation gate as the
     Arka software quality rules define them.

  The gate preserves every check's exit status. A change is not ready while any check is
  red.
- **Review**: every change is reviewed against this constitution before merge. Reviewers
  reject boundary violations, untested logic, broad state watching, and unjustified
  abstractions, whatever the size of the change.
- **Handoff**: every skipped, blocked, or failing check and every new suppression is reported
  explicitly. Silence is never evidence of success.
- **Documentation**: a change that alters behavior, configuration, or a command updates the
  owning documentation in the same change.

## Governance

- This constitution supersedes other practices for this repository. The Arka Knowledge Base
  rules (Flutter coding rules, software quality rules, project structure) apply wherever
  they do not conflict with this document. Where they conflict, this document wins and the
  conflict is recorded for a KB update. The known conflict is Principle I splitting the KB's
  `repositories/` folder into `ports/` and `adapters/`.
- **Amendments** are made through `/speckit-constitution` in a dedicated commit. The commit
  states the version change and the reason, and a migration note when existing code becomes
  non-compliant. Andre approves every amendment.
- **Versioning** follows semantic versioning:
  - MAJOR: a principle is removed or redefined in a backward-incompatible way.
  - MINOR: a principle or section is added or guidance is materially expanded.
  - PATCH: clarifications, wording, and typo fixes.
- **Compliance review**: `/speckit-plan` checks compliance before design and again after
  design. `/speckit-analyze` flags constitution violations as CRITICAL. Code review verifies
  compliance on every change.
- Runtime development guidance for agents lives in `AGENTS.md` (TODO(AGENTS_MD): create it),
  which MUST NOT contradict this constitution.

**Version**: 1.0.0 | **Ratified**: 2026-11-24 | **Last Amended**: 2026-11-24
