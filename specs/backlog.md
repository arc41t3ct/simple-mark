# SimpleMark Backlog

Spec Kit has no built-in backlog. Its commands work on one feature at a time. This file is the
project's list of work that has been identified but deliberately deferred.

**How it works**

- Each entry has a stable ID (`BL-NNN`), its origin, the reason it is deferred, and the known
  gaps.
- To start an entry, run `/speckit-specify` with the entry's text. This creates a new
  `specs/NNN-<name>/` feature. Then mark the entry as `Promoted → specs/NNN-<name>` here. Do not
  delete it.
- Gaps discovered during `/speckit-plan`, `/speckit-implement`, or testing that are outside the
  current feature's scope are added here instead of growing the current spec.
- Order follows priority: platform parity first, then reading features.

## Platforms

### BL-001 - Windows 11 support

- **Origin**: [001-markdown-reader](./001-markdown-reader/spec.md), macOS-first decision of
  2026-11-27
- **Status**: Open
- **Why deferred**: the macOS round trip comes first. Andre will verify on a Windows 11 machine.
- **Scope to specify**:
  - custom frame with Windows caption buttons on the right, snap layouts, and the double-click
    maximize behavior
  - "Open with" and double-click registration for `.md` and `.markdown`, through the registry or
    MSIX (Microsoft's app package format)
  - handing a file to an already-running instance, so that a second launch reuses the existing
    window
  - file and folder access rules on Windows (no sandbox equivalent to macOS unless packaged as
    MSIX with capabilities), and long-path handling
  - file change detection on network drives, and restoring the window on multi-monitor setups
    with different DPI (dots per inch) scaling
  - signed installer

### BL-002 - Linux support

- **Origin**: [001-markdown-reader](./001-markdown-reader/spec.md), macOS-first decision of
  2026-11-27
- **Status**: Open
- **Why deferred**: macOS first. Linux gaps are collected here as they surface.
- **Known gaps**:
  - **Window position restore on Wayland**: Wayland does not let apps set their own global
    window position, so only the size and maximized state can be restored there. The success
    criteria must say "where the platform allows". The source is the Wayland `xdg-shell`
    protocol, which has no client request to set a window's position.
  - **Custom frame**: client-side decorations differ between GNOME, KDE (K Desktop
    Environment), and tiling window managers. The window controls must match each
    environment's button layout setting.
  - **"Open with" registration**: done through a `.desktop` file with the
    `text/markdown` MIME (Multipurpose Internet Mail Extensions) type, which differs per package
    format (deb, rpm, Flatpak, Snap).
  - **Flatpak or Snap sandboxing**: folder access goes through the XDG (Cross-Desktop Group)
    desktop portal file chooser. That portal grants persistence differently from macOS
    bookmarks.
  - **Change detection limits**: the system's file-watch limits (inotify watch limits) can be hit
    in large trees.

## Distribution

### BL-003 - macOS signed and notarized distribution

- **Origin**: 001 Assumptions
- **Status**: Open
- **Scope**: Developer ID signing, notarization, a DMG (disk image) or ZIP release, an update
  path, and an optional Mac App Store submission. The sandbox from 001 FR-027 is already
  App Store compatible.

## Reading features

### BL-004 - Tree filter and file search

- **Origin**: 001 Assumptions
- **Status**: Open
- **Scope**: filter the tree by name, and search across file contents within the granted root.

### BL-005 - In-document search

- **Origin**: 001 Assumptions
- **Status**: Open
- **Scope**: find text in the open document, with next and previous navigation and highlights.

### BL-006 - Extended rendering

- **Origin**: 001 Assumptions
- **Status**: Open
- **Scope**: syntax highlighting in code blocks, footnotes, math, and diagrams (for example
  Mermaid).

### BL-007 - TOML and JSON frontmatter

- **Origin**: 001 Assumptions
- **Status**: Open
- **Scope**: TOML (Tom's Obvious, Minimal Language) frontmatter between `+++` lines and JSON
  (JavaScript Object Notation) frontmatter, shown in the same bar.

### BL-008 - Themes beyond black and white

- **Origin**: 001 FR-038
- **Status**: Open
- **Scope**: accent colors and softer palettes, such as off-white and dark grey, which reduce
  halation for long reading sessions.

### BL-009 - Remote images with consent

- **Origin**: 001 FR-047
- **Status**: Open
- **Scope**: an opt-in setting to load remote images, per document or per domain. It needs the
  outgoing-network entitlement and a privacy note under the GDPR (General Data Protection
  Regulation).

### BL-010 - Print and export

- **Origin**: 001 Assumptions
- **Status**: Open
- **Scope**: print the rendered document, and export it to PDF (Portable Document Format) or HTML
  (HyperText Markup Language).

### BL-011 - Multiple windows or tabs

- **Origin**: 001 Assumptions
- **Status**: Open
- **Scope**: read several documents or tree roots side by side.
