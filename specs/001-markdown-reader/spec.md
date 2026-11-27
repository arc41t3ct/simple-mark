# Feature Specification: Markdown Reader (macOS first)

**Feature Branch**: `001-markdown-reader`

**Created**: 2026-11-27

**Status**: Draft

**Input**: User description: "We want to build a markdown reader that can load a single markdown, in
the left application menu that can be toggled open or closed, which the app will remeber so users
always get the same view when reopening the reader. If we open a single file the app will look into
the folder of the file and see if there are other markdown files and show them in the left menu to
be able to also read them. The markdown text needs to be selectable so we can copy text for pasting
into other areas. Frontmatter should not be displayed raw but if detected should show a nicely
rendered bar at the top. The app will use a custom frame using the window_manager pub package for
windows, linux and mac. The app supports dark mode and light mode with themes but the themes are
plain black on white and white on black for the first itteration."

## Clarifications

### Session 2026-11-27

- Q: Should "the same view" on relaunch also reopen the last document? → A: Yes. Reopen the last
  document with its file tree and scroll position, in addition to the layout.
- Q: Which ways of opening a file are in scope? → A: In-app open dialog, drag and drop,
  launching with a file path, and the operating system's "Open With" / double-click handler.
- Q: What happens when the open file changes on disk? → A: Reload automatically and keep the
  scroll position. The file tree also refreshes when Markdown files are added, removed, or
  renamed.
- Q: Can users open a whole folder? → A: Yes. Opening a folder or a single file both show a
  recursive tree of the Markdown files below the root folder, down to a sane depth limit the user
  can change.
- Q: How are user options stored? → A: In a simple Settings view whose values persist between
  sessions.
- Q: How are operating system permissions handled? → A: Properly. The app keeps the operating
  system's app isolation, asks only for the access it needs, and remembers granted access.
- Q: Which platforms are in scope? → A: macOS only for this feature. Windows 11 and Linux follow
  as separate features, and their known gaps are tracked in the [backlog](../backlog.md).
- Q: How is delivery staged? → A: The simplest working round trip comes first (User Stories
  1–3). Every further story is added on top of it and stays independently shippable.
- Q: Can the left menu be resized? → A: Yes. Users can drag its edge to make it wider or
  narrower, and the width is remembered.
- Q: How do tree folders behave? → A: Each folder expands and collapses with a toggle. Every
  toggle state is remembered when the app is closed and reopened.
- Q: What if remembered state points to folders or files that were deleted? → A: The app handles
  it gracefully. Remembered entries for missing folders and files are dropped without errors,
  the tree shows what exists now, and a note appears only when the user was looking at the
  missing item (the last document or the tree root).

## User Scenarios & Testing *(mandatory)*

**Minimum viable round trip**: User Stories 1–3 together are the first deliverable. They let a
user open Markdown, browse the tree, read and copy text, quit, and get the same view back on
relaunch. Stories 4–9 are increments on top of that round trip, in priority order.

### User Story 1 - Open and read a Markdown file (Priority: P1)

A reader opens a single Markdown file and sees it rendered as formatted text instead of raw
markup. They can select any part of the rendered text and copy it to paste into another
application.

**Why this priority**: reading one file comfortably is the core value of SimpleMark. Every other
story builds on it.

**Independent Test**: open a fixture document containing headings, paragraphs, lists, tables,
code blocks, quotes, links, and images. Check that every element is rendered. Then select text
that spans several paragraphs, copy it, and paste it into a plain-text editor.

**Acceptance Scenarios**:

1. **Given** the reader is running with no document open, **When** the user opens a Markdown
   file through the open dialog, **Then** the document is shown rendered, scrolled to the top,
   and its file name is shown in the window title area.
2. **Given** a rendered document, **When** the user selects text across a heading, a paragraph,
   and a list and copies it, **Then** the clipboard contains that text as plain text, in
   reading order, without Markdown markup characters.
3. **Given** a rendered code block, **When** the user selects and copies part of it, **Then**
   the copied text keeps the original line breaks and indentation.
4. **Given** a document is open, **When** the user opens a different file, **Then** the new
   document replaces the old one.
5. **Given** a file that cannot be read (missing, no permission, or not valid text), **When**
   the user tries to open it, **Then** the reader shows a clear message that names the file and
   the reason, and the previously shown document (if any) stays visible.

---

### User Story 2 - Browse Markdown files as a tree (Priority: P1)

The reader opens a folder, or a single file, and sees all Markdown files below that root folder
as a tree in the left menu, down to the configured depth. One click opens any of them.

**Why this priority**: reviewing AI-generated documentation means moving between many files
spread over subfolders. Without the tree, every file must be opened through the dialog. This is
part of the minimum viable round trip.

**Independent Test**: open a fixture folder containing Markdown files at depths 0 to 7, plus
non-Markdown files, hidden folders, a `node_modules` folder, an empty subfolder, and a symbolic
link loop. With the default depth of 5, confirm that the tree shows exactly the Markdown files at
depths 0–5. Confirm that folders without Markdown, hidden folders, excluded folders, and the loop
are absent. Clicking any file must show it.

**Acceptance Scenarios**:

1. **Given** the reader is running, **When** the user opens a folder through the open dialog,
   **Then** the left menu shows a tree rooted at that folder with every Markdown file within the
   depth limit, folders first, then files, each sorted alphabetically without regard to case.
2. **Given** a folder was opened, **When** its first level contains `README.md` or `index.md`
   (case-insensitive, in that order of preference), **Then** that document is shown. Otherwise
   no document is shown and the reading area asks the user to pick a file.
3. **Given** the user opens a single file, **When** access to its folder is available, **Then**
   the tree is rooted at that file's folder, scanned to the same depth, with the open file
   highlighted and its parent folders expanded.
4. **Given** the user opens a single file, **When** the operating system has granted access to
   that file only, **Then** the file is shown immediately, and the left menu offers a
   non-blocking action "Show Markdown files in ‹folder›". Once the user grants access, the tree
   appears and the grant is remembered.
5. **Given** the tree is visible, **When** the user clicks a file, **Then** that document is
   shown and the highlight moves to it.
6. **Given** the tree is visible, **When** the user clicks a folder's toggle, **Then** the folder
   expands to show its contents or collapses to hide them. Collapsing a folder keeps the
   expansion state of the folders inside it for when it is expanded again.
7. **Given** the user expanded and collapsed folders and then quit the app, **When** the same
   tree root is shown again (on relaunch or when it is opened later), **Then** exactly the same
   folders are expanded and collapsed.
8. **Given** a folder was remembered as expanded and was deleted while the app was closed,
   **When** the tree root is shown again, **Then** the tree shows the folders that exist now,
   with no error and no placeholder for the missing folder. Its remembered state is discarded.
   The remaining folders keep their remembered states.
9. **Given** the last open document or the remembered highlighted file was deleted while the app
   was closed, **When** the app is launched again, **Then** the tree is shown without that file,
   no document is selected, and a short note says that the last file is no longer available.
   Its remembered scroll position is discarded.
10. **Given** a folder contains no Markdown files within the depth limit, **When** it is opened,
   **Then** the left menu says that no Markdown files were found within ‹N› levels and links to
   Settings.
11. **Given** a listed file was deleted or renamed after the tree was built, **When** the user
   clicks it, **Then** the reader shows a message that the file is no longer available and
   refreshes the tree.
12. **Given** a very large folder (for example the home folder), **When** it is scanned, **Then**
   the window stays responsive, the tree fills in as results arrive, and scanning stops at the
   safety limit with a visible notice.

---

### User Story 3 - Toggle, resize, and keep the same view (Priority: P1)

The reader can collapse the left menu to give the document the full width, expand it again, and
drag its edge to make it wider or narrower. When the app is closed and reopened, it comes back
exactly as the user left it.

**Why this priority**: this closes the minimum viable round trip: open, read, quit, and come
back to the same place.

**Independent Test**: open a folder, expand some tree folders, open a document, scroll halfway,
resize the menu, then resize and move the window and quit. Relaunch and confirm that everything
is restored. Repeat with the menu collapsed.

**Acceptance Scenarios**:

1. **Given** the left menu is open, **When** the user activates the menu toggle (button or
   keyboard shortcut), **Then** the menu closes and the document uses the freed width.
2. **Given** the left menu is closed, **When** the user activates the toggle again, **Then** the
   menu opens with the same width, tree, expansion state, and highlight as before.
3. **Given** the left menu is open, **When** the user drags the divider between menu and
   document left or right, **Then** the menu width follows the pointer, within a minimum width
   and a maximum of half the window width.
4. **Given** the menu was resized, **When** the user double-clicks the divider, **Then** the menu
   returns to its default width.
5. **Given** the window becomes narrower than the menu allows, **When** it is resized, **Then**
   the menu shrinks to its maximum and the document keeps at least its minimum readable width.
6. **Given** the user quit the app, **When** it is launched again without a file to open,
   **Then** the menu state and width, the window size, position, and maximized state, the tree
   root and its expanded folders, the last document, and its scroll position are restored.
7. **Given** the last document or tree root no longer exists, cannot be accessed, or is on a
   disconnected drive, **When** the app is launched again, **Then** the reader starts without it
   and shows a short note saying why. Layout settings are still restored.
8. **Given** the app is launched with a file or folder to open, **When** it starts, **Then**
   that item is shown instead of the last one, and the layout is still restored.

---

### User Story 4 - Simple settings (Priority: P2)

The reader opens a small Settings view to change how deep the tree scans, which folder names are
skipped, which theme is used, and which remembered folder grants exist.

**Why this priority**: the defaults work for most folders, but some projects need deeper or
shallower scans, and users must be able to see and revoke folder access.

**Independent Test**: change the depth from 5 to 2 and add `docs-old` to the excluded names. The
tree must update immediately, and the values must survive a relaunch. Remove a remembered folder
grant and confirm that the next relaunch asks for access again.

**Acceptance Scenarios**:

1. **Given** the reader is running, **When** the user opens Settings (standard macOS Settings
   shortcut or a control in the window frame), **Then** Settings shows the scan depth, the
   excluded folder names, the theme mode, and the remembered folder access.
2. **Given** Settings is open, **When** the user changes the scan depth or the excluded names,
   **Then** the current tree is rescanned and updated without a restart.
3. **Given** the user enters an out-of-range depth or an empty folder name, **When** they
   confirm, **Then** the value is rejected with a short explanation and the previous value
   stays.
4. **Given** remembered folder access is listed, **When** the user removes an entry, **Then**
   that access is forgotten and the folder needs to be granted again before its tree is shown.
5. **Given** settings were changed, **When** the user chooses "Reset to defaults", **Then** all
   values return to their defaults. Remembered folder access is cleared only after a separate
   confirmation.
6. **Given** settings were changed, **When** the app is relaunched, **Then** all values are as
   the user left them.

---

### User Story 5 - See frontmatter as a rendered bar (Priority: P3)

When a document starts with a frontmatter block, the reader shows its fields in a clean bar at the
top of the document instead of printing the raw block.

**Why this priority**: AI-generated and documentation-site Markdown often carries frontmatter.
Seeing it raw is noisy, but the body is still readable without this story.

**Independent Test**: open fixture documents with valid frontmatter, malformed frontmatter, no
frontmatter, and a `---` horizontal rule in the middle of the body. Check that only valid
frontmatter at the very start becomes a bar and that nothing raw is shown for it.

**Acceptance Scenarios**:

1. **Given** a document whose first line opens a frontmatter block that is closed correctly,
   **When** it is shown, **Then** a bar at the top lists each field as a label and value, the
   raw block is not shown, and the body starts below the bar.
2. **Given** frontmatter with a `title` field, **When** the bar is shown, **Then** the title is
   the most prominent item in the bar.
3. **Given** frontmatter with list values (for example tags), **When** the bar is shown,
   **Then** each list item is shown as a separate tag-like element.
4. **Given** frontmatter that cannot be read, **When** the document is shown, **Then** the bar
   shows a short notice that the frontmatter could not be read, together with the original
   text, and the body still renders below it.
5. **Given** a `---` line that is not on the first line of the document, **When** the document
   is shown, **Then** it is rendered as a horizontal rule and no bar is shown.
6. **Given** the frontmatter bar, **When** the user selects and copies its text, **Then** the
   field labels and values are copied as plain text.

---

### User Story 6 - Light and dark themes (Priority: P3)

The reader can switch between a light theme (black text on white) and a dark theme (white text on
black), or follow the operating system.

**Why this priority**: comfortable reading in different lighting conditions. In this first
version the themes are deliberately plain.

**Independent Test**: switch between light, dark, and follow-system modes and check that every
visible surface (window frame, left menu, divider, Settings, frontmatter bar, document, code
blocks, messages) changes, with no element keeping the other theme's colors.

**Acceptance Scenarios**:

1. **Given** the reader has never been configured, **When** it is first launched, **Then** it
   follows the macOS appearance setting.
2. **Given** the user chooses light or dark explicitly, **When** the macOS appearance changes,
   **Then** the reader keeps the user's choice.
3. **Given** follow-system mode, **When** macOS switches between light and dark while the reader
   is open, **Then** the reader switches immediately.
4. **Given** either theme, **When** any screen is shown, **Then** text and background use only
   pure black and pure white. Distinctions such as code blocks, the selected tree entry, and the
   frontmatter bar use borders, weight, or inverted blocks instead of other colors.

---

### User Story 7 - Custom window frame (Priority: P4)

The app draws its own title area, so the menu toggle, document name, and Settings control blend
with the reader, while the macOS window controls keep their standard look and position.

**Why this priority**: a consistent, minimal frame is part of the product's look, but the reader
works with the default frame.

**Independent Test**: on macOS, move, resize, minimize, zoom, enter and leave full screen, and
close the window using only the custom frame and the standard window controls.

**Acceptance Scenarios**:

1. **Given** the app is running, **When** the user drags the title area, **Then** the window
   moves.
2. **Given** the app is running, **When** the user double-clicks the title area, **Then** the
   window follows the macOS "double-click a window's title bar" system setting (zoom or
   minimize).
3. **Given** the app is running, **When** the user uses the close, minimize, and zoom/full-screen
   controls, **Then** the window behaves as with the default frame, and the controls sit at the
   top left in their standard macOS look.
4. **Given** the app is running, **When** the user drags any window edge or corner, **Then** the
   window resizes, down to a minimum size at which the menu and document stay usable.
5. **Given** the frame, **When** a document is open, **Then** the title area shows the
   document's file name, and it shows the app name when no document is open.

---

### User Story 8 - Live reload (Priority: P5)

When another program, such as an AI agent or an editor, changes the open document or the Markdown
files in the tree, the reader updates by itself.

**Why this priority**: very valuable when reviewing files an agent is still writing, but the
round trip works without it because the user can reopen the file.

**Independent Test**: while a document is open and scrolled down, rewrite it from a script
several times per second for five seconds, add and delete Markdown files in a subfolder, and save
the document by "write temporary file and rename". Confirm that the latest content and tree are
shown and that the reading position is kept.

**Acceptance Scenarios**:

1. **Given** a document is open and scrolled down, **When** another program changes the file,
   **Then** the reader shows the new content automatically and keeps the reading position as
   close as possible to where it was.
2. **Given** many writes in quick succession, **When** they stop, **Then** the reader shows the
   final content and does not flicker through intermediate states.
3. **Given** a document is open, **When** the file is deleted or moved away, **Then** the last
   content stays visible with a notice that the file is no longer available.
4. **Given** the tree is visible, **When** Markdown files or folders are added, removed, or
   renamed within the scanned depth, **Then** the tree updates and keeps the highlight and the
   expanded folders.

---

### User Story 9 - More ways to open (Priority: P6)

The reader can also open files and folders by drag and drop, by launching with a path, and
through Finder's "Open With" or double-click.

**Why this priority**: these speed up daily use, but the open dialog from User Stories 1 and 2
already covers the need.

**Independent Test**: drop a file and a folder onto the window, launch the app from a terminal
with a path, and double-click a `.md` file in Finder while the app is closed and while it is
running. Each must behave like the open dialog.

**Acceptance Scenarios**:

1. **Given** the reader is running, **When** the user drops a Markdown file or a folder onto the
   window, **Then** it opens exactly as if chosen in the open dialog, including the folder
   access rules.
2. **Given** the app is not running, **When** it is launched with a file or folder path, or a
   Markdown file is opened from Finder, **Then** the app starts and shows that item.
3. **Given** the app is running, **When** a Markdown file is opened from Finder, **Then** it
   opens in the existing window, and the window comes to the front. No second window is
   created.
4. **Given** several items are dropped at once, **When** they are handled, **Then** the first
   Markdown file or folder is opened and a short note says that the rest were ignored.

---

### Edge Cases

- **Empty file**: shows an empty document with a short "this file is empty" note, not an error.
- **Frontmatter only**: shows the bar and an empty body note.
- **Very large file**: files up to 10 MB render without freezing the window. Larger files are
  refused with a message that states the limit.
- **Not valid UTF-8 or binary content**: refused with a message. Partially decoded text is never
  shown.
- **Unsupported extension**: opening, dropping, or passing a file without a Markdown extension is
  refused with a message.
- **Unreadable subfolder during a scan**: it is skipped and marked in the tree as "no access".
  The rest of the scan continues.
- **Symbolic links**: linked files are shown. Linked folders are not followed, which prevents
  loops and scans escaping the root.
- **Scan safety limit**: a scan stops after 10,000 Markdown files. The tree shows what was found
  and a notice that the limit was reached.
- **Depth 0**: only the root folder's own Markdown files are listed.
- **Settings changed during a scan**: the running scan is cancelled and restarted with the new
  values.
- **Long names**: file and folder names that do not fit are shortened with an ellipsis, and the
  full name is shown on hover.
- **Names that differ only in case**: both are listed, and the sort order between them is
  stable.
- **Relative links to another Markdown file**: clicking opens that file. If it is inside the
  current root, the tree highlights it. If it is outside, the folder access rules apply, as for
  opening a single file.
- **Links to headings in the same document**: clicking scrolls to the heading.
- **External links (web, mail)**: open in the system default application only after the user
  clicks them. Other link types are not followed.
- **Relative local images**: rendered when the file exists and access is granted. Missing images
  show their alternative text.
- **Remote images**: not loaded, so that reading never contacts a network. Their alternative
  text and address are shown instead.
- **Raw HTML in the document**: never executed or rendered as HTML. It is shown as literal text.
- **Restored window position on a disconnected display**: the window is placed fully on a
  visible display.
- **Corrupt or missing saved preferences**: the reader starts with defaults and records the
  problem in its log.
- **Remembered folder deleted** (expanded or collapsed): it is absent from the tree, and its
  remembered state is discarded without a message.
- **Remembered folder deleted together with the open document inside it**: the tree is shown
  without it, no document is selected, and a note says that the last file is no longer
  available.
- **Remembered folder renamed**: treated as one folder removed and another added. The new name
  starts collapsed, unless it contains the open document.
- **Folder or file deleted while the app is running**: it disappears from the tree with live
  reload (User Story 8). Before User Story 8 is delivered, it disappears on the next rescan or
  when its entry is clicked (FR-015).
- **Every remembered folder of a root deleted**: the root shows its current contents with
  default expansion.
- **Remembered root on a disconnected drive**: the state is kept (FR-014c) and reapplied when the
  drive is back.
- **Remembered folder access for a deleted folder**: shown in Settings as "no longer available",
  with the remove action (FR-031, FR-032).
- **Revoked or stale folder access**: the reader shows the "Show Markdown files in ‹folder›"
  action again instead of failing silently.
- **Save by replace** (an editor writes a temporary file and renames it over the original): the
  file counts as changed, not deleted, and the reader reloads it.
- **Scroll restore after heavy edits**: if the remembered position no longer exists because the
  document got shorter, the reader shows the end of the document.

## Requirements *(mandatory)*

### Functional Requirements

**Opening and reading**

- **FR-001**: Users MUST be able to open a single Markdown file or a folder through an in-app
  open dialog, reachable from the window frame, the empty state, and the standard Open shortcut.
- **FR-002**: The system MUST accept files with the extensions `.md` and `.markdown`, matched
  regardless of case.
- **FR-003**: The system MUST render CommonMark plus the common GitHub extensions: tables, task
  lists (read-only), strikethrough, fenced code blocks, and autolinks.
- **FR-004**: All rendered document text, including headings, lists, tables, code blocks, and the
  frontmatter bar, MUST be selectable as one continuous selection across elements.
- **FR-005**: Copying a selection MUST put plain text on the clipboard in reading order, without
  Markdown markup. Code blocks MUST keep their line breaks and indentation.
- **FR-006**: Users MUST be able to select all document text and copy it with the standard macOS
  shortcuts and the context menu.
- **FR-007**: The system MUST show a clear, non-technical message when a file cannot be opened,
  naming the file and the reason, and MUST NOT replace the currently shown document.
- **FR-008**: The system MUST refuse files larger than 10 MB with a message stating the limit.

**File tree**

- **FR-009**: Opening a folder MUST make it the tree root. Opening a single file MUST make that
  file's folder the tree root, subject to FR-030.
- **FR-010**: The system MUST scan the root recursively for Markdown files down to the configured
  depth (default 5 levels below the root, allowed range 0–20).
- **FR-011**: The scan MUST skip hidden files and folders, folders whose name is in the excluded
  list (default: `node_modules`, `build`, `dist`, `vendor`), and linked folders. Folders that
  contain no Markdown file within the depth limit MUST NOT appear.
- **FR-012**: The scan MUST stop after 10,000 Markdown files and show a notice that the limit was
  reached.
- **FR-013**: The tree MUST list folders first, then files, each sorted alphabetically without
  regard to case, with a stable order for names that differ only in case.
- **FR-014**: The currently shown file MUST be highlighted, and its parent folders MUST be
  expanded.
- **FR-014a**: Every folder in the tree MUST have a toggle that expands or collapses it. The
  system MUST remember the expanded or collapsed state of each folder for each tree root, across
  restarts and when a root is opened again later. Collapsing a parent MUST NOT reset the state
  of the folders inside it. Folders that never had their toggle used start collapsed, except
  the parents of the open document (FR-014).
- **FR-014b**: Remembered state that refers to folders or files that no longer exist MUST be
  handled without errors, crashes, or empty placeholders:
  - expansion states of missing folders are discarded once the scan of their root completes
  - a missing remembered highlighted file or last document is dropped together with its scroll
    position, and the user sees a short note only for the last document
  - a missing tree root is skipped on relaunch with a short note (FR-022)
  - if a folder or file with the same path appears again later, it is treated as new, so
    folders start collapsed

  Discarding stale entries is an expected condition, not a failure. It is recorded in the log at
  information level and never shown as an error dialog.
- **FR-014c**: Remembered expansion state MUST be kept for the 50 most recently used tree roots.
  A root that is temporarily unavailable, such as one on a disconnected drive, keeps its state
  until it falls out of that list, so it is not lost while the drive is away.
- **FR-015**: Clicking a file entry MUST show that file. If it no longer exists, the system MUST
  tell the user and refresh the tree.
- **FR-016**: Scanning MUST NOT block the window. The tree fills in progressively while the scan
  runs.
- **FR-017**: When a folder is opened, the system MUST show its first-level `README.md`, or else
  its first-level `index.md` (both case-insensitive). If neither exists, it MUST ask the user to
  pick a file.
- **FR-018**: Before anything is opened, the left menu MUST show an empty state with actions to
  open a file or a folder.

**Left menu, layout, and remembered view**

- **FR-019**: Users MUST be able to open and close the left menu with a control in the window
  frame and with a keyboard shortcut.
- **FR-020**: Users MUST be able to resize the left menu by dragging the divider between menu and
  document. The width is bounded by a minimum and by half the window width, and a double-click
  on the divider resets the default width.
- **FR-021**: The system MUST remember, across restarts, the menu open or closed state, the menu
  width, the window size, position, maximized and full-screen state, the tree root, the expanded
  folders, the last open document, and its scroll position.
- **FR-022**: On relaunch without an item to open, the system MUST restore everything in FR-021.
  An item that is no longer available is skipped with a short note. An item passed at launch
  MUST take precedence over the last one.

**Settings**

- **FR-023**: The system MUST provide a Settings view, opened with the standard macOS Settings
  shortcut and from the window frame. It contains: scan depth, excluded folder names, theme
  mode, and the list of remembered folder access.
- **FR-024**: Settings values MUST be validated when entered, persist across restarts, and apply
  immediately. Changing the depth or the exclusions rescans the current tree.
- **FR-025**: Users MUST be able to reset settings to their defaults. Clearing remembered folder
  access requires a separate confirmation.
- **FR-026**: If saved settings or view state are missing or unreadable, the system MUST start
  with defaults (menu open, default width, depth 5, default exclusions, follow-system theme,
  centered default-size window) and record the problem in its log.

**Operating system permissions (macOS)**

- **FR-027**: The app MUST run inside the macOS app sandbox and request only the capabilities it
  uses: read-only access to files and folders the user selected, and remembering that access
  across restarts. It MUST NOT request network, camera, microphone, location, contacts, or full
  disk access.
- **FR-028**: Choosing a folder in the open dialog or dropping it MUST grant read access to that
  folder and everything below it. That grant MUST be remembered.
- **FR-029**: Access remembered for a folder MUST cover later opens of any file inside it,
  without asking again.
- **FR-030**: When a single file is opened and its folder is not yet granted, the system MUST
  show the file immediately and offer a non-blocking "Show Markdown files in ‹folder›" action.
  The action opens the folder chooser preset to that folder. Until access is granted, the tree
  shows only that file.
- **FR-031**: When remembered access is revoked, stale, or points to a missing location, the
  system MUST detect it, tell the user, and offer to grant access again. It MUST NOT fail
  silently or fall back to wider access.
- **FR-032**: Users MUST be able to see and remove remembered folder access in Settings.

**Frontmatter**

- **FR-033**: The system MUST detect a frontmatter block only when it starts on the first line of
  the file with `---` and is closed by a matching `---` line.
- **FR-034**: A detected frontmatter block MUST NOT be displayed raw. The system MUST render it as
  a bar at the top of the document that shows each field as a label and value. A `title` field
  is the most prominent item, list values are shown as separate tag-like elements, and nested
  values are shown in an indented label/value form.
- **FR-035**: Frontmatter that cannot be read MUST produce a bar with a notice and the original
  text. The body MUST still render.

**Themes**

- **FR-036**: The system MUST offer three theme modes: light (black on white), dark (white on
  black), and follow system. Follow system is the default.
- **FR-037**: Theme changes MUST apply immediately to every surface without a restart.
- **FR-038**: In this version, both themes MUST use only pure black and pure white. Visual
  distinction comes from borders, font weight, spacing, and inverted blocks.

**Window frame (macOS)**

- **FR-039**: The system MUST replace the default title bar with its own title area. The title
  area contains the draggable region, the document name, the menu toggle, and the Settings
  control, and it keeps the standard macOS window controls at the top left.
- **FR-040**: The window MUST support moving, edge and corner resizing, the system double-click
  behavior, full screen, and a minimum window size.

**Live reload**

- **FR-041**: While a document is open, the system MUST detect changes to that file and reload it
  automatically, keeping the reading position as close as possible. Bursts of writes MUST
  result in showing the latest content, not every intermediate state.
- **FR-042**: If the open file is deleted, moved away, becomes unreadable, or exceeds the size
  limit, the system MUST keep the last good content visible with a notice that gives the
  reason.
- **FR-043**: The system MUST update the tree automatically when Markdown files or folders are
  added, removed, or renamed within the scanned depth, keeping the highlight and expansion
  state.

**More ways to open**

- **FR-044**: Users MUST be able to open a Markdown file or a folder by dropping it onto the
  window, by launching the app with a path, and through Finder's "Open With" and double-click.
  For Finder, the app registers as a viewer for `.md` and `.markdown` files.
- **FR-045**: When the app is already running, an item opened through Finder MUST open in the
  existing window and bring that window to the front.

**Content safety and privacy**

- **FR-046**: The system MUST NOT execute or render raw HTML from documents. It is shown as
  literal text.
- **FR-047**: The system MUST NOT make any network request. Remote images are not loaded, and
  external links open in the system default application only after an explicit click and only
  for web and mail addresses.
- **FR-048**: Relative links to Markdown files MUST open in the reader. Links to headings MUST
  scroll to the heading.
- **FR-049**: The system MUST keep the window responsive while a document is loaded and rendered
  and while a folder is scanned.

### Key Entities

- **Document**: one Markdown file that is shown. It has a file location, a file name, the body
  text, an optional frontmatter, and a load result (loaded, empty, refused with a reason).
- **Frontmatter**: the metadata block at the start of a document. It is an ordered list of fields
  (label and value, where a value can be text, a list, or nested fields), or a "could not be
  read" result that carries the original text.
- **File Tree**: the Markdown files found below a root folder. It has the root location, the
  nested folder and file entries, the selected entry, a scan status (scanning, complete, limit
  reached), and entries marked "no access".
- **Folder Expansion State**: for each of the 50 most recently used tree roots, the set of
  folders whose toggle is expanded. It is kept across restarts, reapplied whenever that root is
  shown, and cleaned of entries for folders that no longer exist.
- **Folder Access Grant**: the user's remembered permission to read one folder and everything
  below it. It has the folder location, the time it was granted, and its status (valid, stale,
  revoked).
- **Settings**: scan depth, excluded folder names, and theme mode, each with a default and a
  validation rule.
- **View State**: what the reader restores on relaunch: menu open or closed, menu width, window
  size, position, maximized and full-screen state, tree root, last document, and scroll
  position. View State, Folder Expansion State, Settings, and Folder Access Grants are stored
  only on the local machine.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A Markdown file of up to 1 MB is shown fully rendered within 1 second of the user
  opening it.
- **SC-002**: Switching to another file from the tree shows it within 0.5 seconds for files up to
  1 MB.
- **SC-003**: A folder with 1,000 Markdown files spread over 5 levels is fully listed within
  2 seconds. The first entries appear within 0.5 seconds.
- **SC-004**: Scanning the user's home folder never makes the window unresponsive for more than
  100 milliseconds and ends at the depth limit or the safety limit with a notice.
- **SC-005**: In 100% of relaunch tests, the menu state and width, window size, position, theme,
  tree root, and expanded folders match the state at the last quit. The scroll position is
  within one screen height of where the user left it.
- **SC-005a**: In 100% of relaunch tests, every folder toggle in the tree is in the same
  expanded or collapsed state as at the last quit, including folders nested inside collapsed
  parents.
- **SC-005b**: In stale-state tests (remembered folders and files, including the last document
  and highlighted file, deleted or renamed while the app is closed), 100% of relaunches show the
  current tree with zero error dialogs and zero crashes. After the first completed scan, the
  stored state contains no entries for missing items within available roots.
- **SC-006**: After a folder has been granted once, 100% of relaunches restore its tree and last
  document without asking for access again.
- **SC-007**: In sandbox verification tests, the app cannot read any file outside the locations
  the user selected or granted, and it makes zero network requests.
- **SC-008**: For every document in the test corpus, text copied from the reader and pasted into
  a plain-text editor contains no Markdown markup characters and keeps reading order.
- **SC-009**: For every corpus document with valid frontmatter, no raw frontmatter text is
  visible, and every field appears in the bar.
- **SC-010**: Both themes reach a text-to-background contrast ratio of 21:1, the maximum defined
  by WCAG (Web Content Accessibility Guidelines).
- **SC-011**: On macOS, all window operations (move, resize, minimize, zoom, full screen, close)
  succeed with the custom frame.
- **SC-012**: When another program saves the open file, the new content is visible within
  1 second after the last write. Tree changes appear within 2 seconds.
- **SC-013**: Double-clicking a `.md` file in Finder opens it in SimpleMark, both when the app is
  closed and when it is running.
- **SC-014**: A first-time user can open a folder, read a file, copy a paragraph, and find the
  same view after relaunch, without help, in under 2 minutes.

## Assumptions

- **Users**: one person reading local Markdown files on a Mac. There are no accounts, sync, or
  sharing.
- **Platform**: macOS only in this feature, on the current and the previous major macOS release.
  Windows 11 and Linux are separate follow-up features, listed in the [backlog](../backlog.md).
  This resolves the constitution's `TODO(TARGET_PLATFORMS)` as "macOS first".
- **Read-only**: the reader never modifies, creates, renames, or deletes files.
- **Default limits**: scan depth 5, safety limit 10,000 files, and a file size limit of 10 MB.
  They are sane defaults for documentation and code repositories. Only the depth and the
  exclusions are user-configurable, so that Settings stays simple.
- **Frontmatter format**: YAML (YAML Ain't Markup Language) frontmatter between `---` lines. TOML
  (Tom's Obvious, Minimal Language) and JSON (JavaScript Object Notation) frontmatter are in the
  backlog.
- **Rendering scope**: syntax highlighting, math, diagrams, footnotes, tree filtering,
  in-document search, printing, and export are out of scope and listed in the backlog.
- **Theme palette**: only pure black and pure white in this version.
- **Single window**: one reader window at a time.
- **Keyboard shortcuts** follow macOS conventions. The exact bindings are decided in the plan.
- **Distribution**: for this feature the app runs as a local development and test build. Signed
  and notarized distribution, and a possible App Store release, are backlog items. Keeping the
  sandbox (FR-027) leaves both paths open.
- **Implementation notes for planning (user-specified, not requirements)**:
  - The custom frame uses the `window_manager` package.
  - The macOS entitlements must be updated to match FR-027 to FR-032 and FR-044. That means app
    sandbox on, user-selected files read-only, app-scoped security bookmarks, no outgoing
    network in release builds, and document type registration for `.md` and `.markdown`.
    `/speckit-plan` owns the exact entitlement keys and verifies them.
