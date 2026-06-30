---
name: visily-to-app
description: >
  This skill should be used when the user wants to convert or replicate a Visily UI/UX
  design into a web or mobile app. Trigger when the user asks to "convert a Visily design",
  "replicate Visily screens", "build the app from Visily exports", "turn Visily into
  React/Vue/Next/Compose/SwiftUI/Flutter/React Native", mentions a `visily_exports` folder,
  or points at Visily HTML/PNG exports and wants screens, a design system, or a build brief —
  for web (React, Next.js, Vue, HTML+Tailwind) or mobile (Android Jetpack Compose, iOS
  SwiftUI, Flutter, React Native). Triggers regardless of the user's language (e.g. requests
  written in Spanish).
argument-hint: "[path to visily_exports] [--brief|--code] [--platform web|mobile] [--stack react|next|vue|html|android|ios|flutter|rn] [--lang en|es|...]"
allowed-tools: [Read, Glob, Grep, Bash, Write, Edit, AskUserQuestion]
version: 0.3.0
---

# Visily → App (web or mobile)

Convert Visily exports into a high-fidelity **web or mobile** app UI, in whichever stack the
user picks. The source of truth is the **HTML** from `Export to Code` (self-describing: exact
colors, sizes, radii, shadows, and fonts per element); the **PNGs** are the visual ground
truth for comparison. Even though Visily can export React/Vue/HTML, **rebuild from the HTML**
(same pipeline for web and mobile) for clean code and to avoid the native export's absolute
positioning.

Do not try to connect to the Visily URL or an API: there is no programmatic access and the
URL renders on a canvas (returns an empty shell). The only valid bridge is the exports. Full
rationale: `<skill-dir>/references/method.md`.

Per-stack translation detail (colors, typography, shapes, icons, layout, scroll, lists, web
responsive): `<skill-dir>/references/stack-mapping.md`. Load that file before generating code.

## Interaction language (settle before asking anything)

Detect the language the user is writing in and conduct **all questions, explanations, and
status updates in that language**. If it can't be determined (e.g. the skill was invoked with
no natural-language text — only a path/flags), **ask first**: `English` | `Español` | `Other`
(free text), then continue in the chosen language. Honor `--lang` if passed. The skill files
themselves stay in English; only the interaction adapts. Generated code keeps English
identifiers and comments by convention; the delivered **brief** uses the interaction language
(unless the user says otherwise).

## Flow

### 1. Locate and pair the exports

If the user passed paths in `$ARGUMENTS`, use them. Otherwise auto-detect from the working
directory:

- PNG:  `Glob` → `**/visily_exports/**/*-multiscreens/*.png` (also accept any PNG folder under
  `visily_exports/`).
- HTML: `Glob` → `**/visily_exports/**/*-multiscreens-html/*/index.html`.

Pair by **normalized name**: from the PNG strip the `visily-` prefix and the extension; from
the HTML directory strip the `visily-static-` prefix and the `-html` suffix. Compare in
lowercase, preserving accents and apostrophes (e.g. `today's-route`, `exhibición`,
`validación-de-exhibiciones`). Build a `{ screen, png, html }` inventory and **report it as a
table**, flagging screens that have a PNG but no HTML, or vice versa.

If neither folder is found, **ask via AskUserQuestion**: (a) the PNG path and (b) the HTML
path. HTML is required to generate code; without it, offer brief-only mode from PNGs (lower
fidelity, estimated colors).

### 2. Ask the scope (AskUserQuestion)

Honor any flags already passed in `$ARGUMENTS` (`--brief`/`--code`, `--platform`, `--stack`,
`--lang`) and only ask what's missing. Group into a few AskUserQuestion calls.

**Question 0 — Language (only if it couldn't be detected):** `English` | `Español` | `Other`.
Resolve this before any other question and ask everything below in that language.

**Question 1 — Platform (ALWAYS first after language):** `Web` | `Mobile`. Drives everything
else.

**Stack by platform** (+ ask the target framework/library version):
- **Mobile:** `Android (Jetpack Compose)` | `iOS (SwiftUI)` | `Flutter` | `React Native`.
- **Web:** `React (Vite/SPA)` | `Next.js` | `Vue` | `HTML + Tailwind`.

**Common questions** (same set, worded for the chosen platform):
- **Depth** — `Brief only` (structured document) | `Full code`.
- **Scope** — `Screens only (static UI)` | `+ Navigation/routing` | `+ Functionality` (state,
  mock data, validation).
- **Destination** — `New project (scaffold)` | `Existing project` (ask path + id: package/
  bundle id on mobile; package/repo name on web).
- **Versions/targets** — mobile: minSDK/target (Android), deployment target (iOS), Flutter/RN
  version. Web: Node/framework version and target browsers.
- **Calibration screen** — which to build first (default: `Login` or the simplest one).

**Web-only questions (at runtime — the user chose not to fix a default):**
- **Styling** — `Tailwind` (the export is already Tailwind: maximum fidelity) | `CSS / CSS
  Modules` | `CSS-in-JS` (styled-components/emotion).
- **Responsive** — `Responsive with breakpoints (mobile→desktop)` | `Faithful to the fixed
  frame` | `Per Visily frames` (use the different sizes designed in the board).

**Design details** (confirmable defaults, in one shot):
- Common: icons = import the **exported SVGs**; fonts = **bundle/register** every family found
  in the HTML; dark mode = **light only**; i18n = **embedded source-language strings** (as
  exported).
- Mobile-only: discard the **simulated status bar** (the top ~40px with "9:41" and signal
  icons) and use the system bar + safe areas; px→dp/pt **1:1**.
- Web-only: CSS px **1:1**; read the **frame width** from the HTML (don't assume 390 — a web
  frame may be 1280/1440).

Confirm the resulting plan in one sentence before generating.

### 3. Extract the design system (once, stack-agnostic)

Parse the HTML's Tailwind classes (all screens) to consolidate tokens. Follow
`<skill-dir>/references/stack-mapping.md`. Extract:

- **Colors** from `bg-[#hex]`, `text-[#hex]`, `border-[#hex]`, honoring the `/[n]` alpha
  suffix. Group duplicates and name them by role (primary, surface, text, border, accent).
- **Typography** from `font-[...]` + `text-[Npx]` + `leading-[Npx]` + `font-[weight]` +
  `tracking-[...]`. **Do not hardcode families**: take every one that appears (seen: Inter,
  Hanken Grotesk, Lexend) and declare them all.
- **Shapes/radii** (`rounded-[Npx]`; `9999px` = pill) and **shadows** (`shadow-[...]`).
- **Recurring spacing** for a minimal scale.

Generate the tokens in the stack's format (mobile: `Color.kt`/`Type.kt`/`Shape.kt`,
`Theme.swift`, `ThemeData`; web: CSS variables / extended `tailwind.config` / TS theme) and
copy the assets (`assets/*.svg` and raster `.webp`/`.png`).

### 4. Calibrate with one screen

Build **only the calibration screen**, translating the HTML's absolute positioning into
**idiomatic layouts** for the stack (do not copy `top/left`). Show the result and **compare it
against its PNG**. Ask for a fidelity sign-off and adjust the method/mapping before
continuing. Do not scale without approval.

### 5. Scale and deliver

After approval, generate the rest reusing the tokens and extracting repeated structures (list
cards, badges, headers) into reusable components with their data model when scope includes
navigation/functionality. Deliver per mode:

- **Brief**: a structured `.md` with tokens, screen inventory, icon mapping, detected
  components, and scope decisions.
- **Code**: files in the destination project, organized by theme + screens + components.

## Fidelity rules (critical)

- Reinterpret `position: absolute` as idiomatic layout (`Column`/`Row`/`Box`,
  `VStack`/`HStack`/`ZStack`, or **flexbox/grid** on web). The nested `<div>` tree already
  encodes the hierarchy; use it.
- **Scroll (mobile):** if the PAGE's `h-[Npx]` exceeds the frame (~844), the screen scrolls →
  use a scrollable container (LazyColumn / ScrollView / SingleChildScrollView). On web the page
  scrolls naturally.
- Parse alpha (`/[n]`), pills (`rounded-[9999px]`), `backdrop-blur`, and rotated dividers
  (`rotate-[90deg]` = a 1px vertical line).
- Icons: the `data-icon` attribute gives the Lucide name; by default import the SVG from
  `assets/` for guaranteed fidelity.
- Detect repeated structures and don't copy them N times: one component + a data list.
- **Mobile only:** discard the exports' simulated status bar (top ~40px with "9:41" and signal
  icons) and use the real system bar + safe areas/insets.

## Notes

- Work screen by screen; don't dump them all without calibrating.
- Preserve the screen copy verbatim from the HTML (do not translate UI text).
- This is the plain-text working version (Phase 1). Once stable it gets packaged as a plugin
  and published; see the project plan.
