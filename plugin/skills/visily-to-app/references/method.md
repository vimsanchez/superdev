# Method: why HTML is the bridge (not the URL/API)

Context reference for the `visily-to-app` skill. Explains the decisions behind the flow. Load
only if the user asks why, or if detection fails.

## Why there is no direct connection to Visily

- **No public/developer API.** The only things that surface are third-party connectors (e.g.
  ApiX-Drive) that don't extract design data.
- **The URL can't be used to extract the design.** Visily draws screens on a JavaScript
  *canvas*. Fetching a board URL (even a "public" one) returns an empty shell: the word
  "Visily" and test characters, **zero design**. It's not a login problem; the content lives
  in the canvas, not in the served HTML.

Bottom line: don't waste time trying to `WebFetch` the board or hunting for an API. The only
reliable path is the **exports** the user generates from the Visily app.

## How the exports are generated (user side, in Visily)

1. **Code:** `Export to Code → HTML`. Pick HTML, not React/Vue: it gives a clean hierarchy (a
   `<div>` tree) + exact CSS, without framework noise. It's the closest thing to a mobile
   stack's composable/view tree.
2. **Images:** `Export to Image/PDF → PNG`, 2x scale if possible (crisp text and edges; JPG
   blurs them). They're the visual ground truth for comparison, not the source of measurements.

The HTML export produces, per screen, a folder with: `index.html`, `tailwindcss.js`,
`README.md`, and `assets/` (icons/logos as SVG and, sometimes, raster `.webp`/`.png` images).

## What the HTML carries (why it's self-describing)

Each element is a node with literal-valued Tailwind classes. Real examples:

- Root frame: `w-[390px] h-[844px] bg-[#F9F9FB]/[1]` (390×844 = phone size).
- Card: `rounded-[18px] shadow-[0px_4px_8px_0px_#1E2A5A0f]` over `bg-[#ffffff]`.
- Text: `text-[#1E2A5A]/[1] font-[Inter] text-[14px] leading-[20px] font-[600]
  tracking-[0.35px] uppercase`.
- Primary button: `bg-[#4F46E5] rounded-[18px] shadow-[0px_8px_16px_0px_#4F46E533]`.
- Icon: `data-icon="Lucide_user_Outlined"` + `src="./assets/IMG_4.svg"`.

From this you get, **without eyeballing**: the hex palette, alpha, sizes, line-height,
tracking, weights, radii, shadows, fonts, and icons by name.

## Real gotchas found in these exports

- **Fonts vary per element.** The `<head>` declares Inter and Lexend, but headings use
  `font-['Hanken_Grotesk']`. Never assume a single family: parse `font-[...]` per element and
  declare every one that appears.
- **Mixed assets.** Icons as SVG (vector, ideal) but also raster `.webp` photos. Copy both;
  SVGs go as vectors, raster as images.
- **Embedded alpha.** Color and opacity come as `#4F46E5/[0.1]` or `text-...#9295A5]/[0.6]`.
- **Pills** = `rounded-[9999px]`. **Circles** = radius = half the side.
- **Fake status bar.** The top ~40px with "9:41" + signal icons are part of the mockup, not the
  app: discard them and use the system bar + safe areas.
- **Scroll.** If the PAGE's `h-[Npx]` exceeds ~844, the screen scrolls (one is 1434px tall).
- **Rotated dividers.** `rotate-[90deg]` on a 1px line = a vertical separator.
- **Repeated structures.** List cards (visits, products) repeat: extract to a component + data
  model, don't duplicate markup.

## Positioning: absolute → idiomatic

The HTML uses `position: absolute` with per-element `top/left`. **Don't port those offsets.**
The container nesting already describes the structure (header, card, row, etc.); rebuild it
with the stack's layouts (Column/Row/Box, VStack/HStack/ZStack, Flutter Column/Row, RN flex
View, web flexbox/grid). The result should be code a stack dev maintains happily, not a literal
coordinate translation.

## Web: the native export and why we still rebuild from HTML

Unlike mobile (no Kotlin/Swift export), on web Visily **does** export native code: `Export to
Code → React`, `Vue`, or `HTML`. Tempting to use directly, but the team decided to **rebuild
from the HTML** on web too, for the same reasons as mobile: the native export drags along
`position:absolute`, coupled components, and little semantics. Rebuilding gives the same token
pipeline and clean, maintainable, responsive code.

Web-specific considerations:
- **Variable frame width.** A web board may have 1280/1440px (desktop) frames or mobile-width
  ones. Read `w-[Npx]` from the PAGE; don't assume 390.
- **Responsive.** Visily frames are fixed-width. The team chose to **ask at runtime** between:
  responsive with breakpoints (mobile→desktop), faithful to the fixed frame, or using the
  different frames present in the board. The absolute→flex/grid reinterpretation is what
  enables real responsiveness.
- **Styling.** Also asked at runtime: Tailwind (most faithful, the export already is), CSS/CSS
  Modules, or CSS-in-JS.
- **Icons.** On web it's idiomatic to use the Lucide package (`lucide-react`/`lucide-vue-next`)
  since `data-icon` gives the exact name; the exported SVG is still an option.

## Calibration strategy

Build **one** screen first (the simplest, typically Login), compare it against its PNG, and
tune the mapping (colors, font, radii, spacing) until it convinces. Only then scale to the
rest reusing the already-validated tokens. This avoids propagating a misinterpretation across
all screens.
