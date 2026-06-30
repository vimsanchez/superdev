# Mapping: HTML Tailwind classes → each stack

Technical reference for `visily-to-app`. Load before generating code. First the parsing rules
(shared); then the per-stack mapping: **mobile** (§2) and **web** (§3).

## 1. Class parsing (shared by all stacks)

| Export Tailwind class            | Meaning                              | Token to produce |
|----------------------------------|--------------------------------------|------------------|
| `w-[390px] h-[844px]`            | frame size (read it, don't assume!)  | reference width px |
| `bg-[#RRGGBB]/[a]`               | background color + alpha             | color with opacity `a` |
| `text-[#RRGGBB]/[a]`             | text color                           | text color |
| `border-[#RRGGBB]` `border-[1px]`| border                               | 1px stroke |
| `rounded-[Npx]`                  | radius                               | corner N (`9999px` ⇒ pill/circle) |
| `shadow-[x_y_blur_spread_#hexAA]`| shadow; trailing `AA` is hex alpha   | elevation/shadow |
| `font-[Inter]` / `font-['Hanken_Grotesk']` | family (varies per element!) | fontFamily |
| `text-[16px] leading-[26px]`     | size and line-height                 | fontSize / lineHeight |
| `font-[600]`                     | weight                               | fontWeight 600 |
| `tracking-[0.35px]` `uppercase`  | letter-spacing / uppercase           | letterSpacing / textTransform |
| `opacity-[a]`                    | node opacity                         | component alpha |
| `rotate-[90deg]` on a 1px line   | vertical divider                     | vertical separator |
| `backdrop-blur-[4px]`            | background blur                      | blur |
| `data-icon="Lucide_x_Outlined"`  | Lucide icon name                     | SVG from `assets/` or Lucide package |

Rules:
- **Color+alpha:** `#4F46E5/[0.1]` ⇒ that color at 10%. `/[1]` ⇒ opaque.
- **Shadow:** the color usually comes as `#1E2A5A0f` (8 digits: RGB + hex alpha `0f`≈6%).
- **Frame width:** read `w-[Npx]` from the PAGE. Mobile ≈ 390; web may be 1280/1440.
- **Scroll (mobile):** if the PAGE's `h-[N]` > ~844, the screen scrolls. On web it scrolls
  naturally.
- **Status bar (mobile only):** ignore the first ~40px container with the status-bar images. It
  doesn't appear in web frames.
- **Fonts:** collect ALL `font-[...]` families and declare/bundle them. The export's `<head>`
  already includes the Google Fonts `<link>`s as a hint.

## 2. Mobile — tokens and layout per stack

### Android — Jetpack Compose
- `ui/theme/Color.kt`: `val Primary = Color(0xFF4F46E5)`. Alpha: `Primary.copy(alpha = 0.1f)`.
- `ui/theme/Type.kt`: `Typography` per role; a `FontFamily` per family from `res/font/`.
- `ui/theme/Shape.kt`: `RoundedCornerShape(14.dp/18.dp)`; pill = `RoundedCornerShape(50)`.
- Layout: `Column`/`Row`/`Box`, `Modifier.padding/size`, `Spacer`. Scroll:
  `Modifier.verticalScroll(...)` or `LazyColumn`.
- SVG icons → `res/drawable/` (vector) + `Icon(painterResource(...))`. Raster `.webp` →
  `res/drawable/` + `Image(painterResource(...))`.
- Safe areas: `Scaffold` + `WindowInsets`; discard the simulated status bar. px→dp 1:1.

### iOS — SwiftUI
- `Theme/Colors.swift`: `Color(hex: 0x4F46E5)` (+ an `init(hex:)` helper). Alpha: `.opacity(0.1)`.
- `Theme/Typography.swift`: `Font.custom("Inter", size: 16)`; register fonts in Info.plist
  (`UIAppFonts`).
- Shapes: `RoundedRectangle(cornerRadius: 18)`; pill = `Capsule()`.
- Layout: `VStack`/`HStack`/`ZStack`, `.padding`, `Spacer`. Scroll: `ScrollView`; lists `List`.
- Icons: add SVGs to the asset catalog (preserve vector) → `Image("ic_user")`; or SF Symbols.
- Safe areas: respect `safeAreaInsets`; don't draw the status bar.

### Flutter
- `theme/app_colors.dart`: `static const primary = Color(0xFF4F46E5);`. Alpha: `.withOpacity(0.1)`.
- `ThemeData` + `TextTheme`; fonts in `pubspec.yaml`.
- Shapes: `BorderRadius.circular(18)`; pill = `StadiumBorder`.
- Layout: `Column`/`Row`/`Stack`, `Padding`, `SizedBox`. Scroll: `SingleChildScrollView`;
  lists `ListView.builder`.
- SVG icons: `flutter_svg` (`SvgPicture.asset`); raster `Image.asset`. Declare in pubspec.
- Safe areas: `SafeArea`.

### React Native
- `theme/colors.ts`: `{ primary: '#4F46E5' }`. Alpha: `'rgba(79,70,229,0.1)'`.
- Fonts via `expo-font` / asset linking. Shapes: `borderRadius: 18` (pill `9999`).
- Layout: `View` with Flexbox; scroll `ScrollView`; lists `FlatList`.
- Icons: `react-native-svg` + `react-native-svg-transformer`; raster `<Image source=require>`.
- Safe areas: `react-native-safe-area-context`.

## 3. Web — tokens and layout per stack

**Source:** rebuild from the HTML (not from Visily's native React/Vue export). Absolute
positioning is reinterpreted as **flexbox/grid**. The frame width is read from the HTML; per
the "responsive" answer: breakpoints (Tailwind `sm/md/lg` or media queries), fixed width with
`max-width`, or multiple frames.

**Tokens per the styling chosen at runtime:**
- **Tailwind** (recommended, the export already is): extend `tailwind.config.js`
  (`theme.extend.colors`, `fontFamily`, `borderRadius`, `boxShadow`). Reuse the export's
  classes almost as-is. Alpha via `bg-[#4F46E5]/10` or a color with `/opacity`.
- **CSS / CSS Modules**: variables in `:root` (`--color-primary:#4F46E5; --radius-card:18px;`)
  and per-component classes.
- **CSS-in-JS** (styled-components/emotion): a `theme` object + styled components.

**Typography/fonts:** keep the export's Google Fonts `<link>`s, or self-host. Declare every
family (Inter, Hanken Grotesk, Lexend).

**Icons:** on web it's idiomatic to use the Lucide package (`lucide-react`, `lucide-vue-next`):
`data-icon` gives the exact name. Alternative: import the SVG from `assets/` as a component.

**Raster images** (`.webp`): to `public/` (or import via the bundler).

### React (Vite/SPA)
- Function components in `src/components` + screens in `src/screens`/`src/pages`.
- Routing: `react-router-dom`. State/functionality: hooks + (if needed) a light store.
- Layout with flex/grid; no `position:absolute` except deliberate overlays (chips over an
  image, badges).

### Next.js
- App Router (`app/`) or Pages Router per version; each screen = a route. SSR/SSG where useful.
- `next/font` for fonts; `next/image` for raster. Tailwind supported out of the box.

### Vue
- SFCs `.vue` (`<template>`/`<script setup>`/`<style>`). Routing: `vue-router`.
- Icons: `lucide-vue-next`. Tailwind or `<style scoped>` per the chosen styling.

### HTML + Tailwind (no framework)
- One `.html` per screen (or partials) + Tailwind via CDN/build. It's the closest to the
  export: clean up the markup, drop absolutes, group into semantic sections, and extract
  repeated fragments. No routing/state (static UI only).

## 4. Recurring component patterns (seen in the exports)

- **Stat card**: metrics separated by vertical dividers (`rotate-[90deg]`) ⇒ a row of columns
  (big value + uppercase label).
- **Badge/pill**: `rounded-[9999px]` with small text ⇒ a `Pill` component.
- **Icon in a square/circle**: container with a tinted `bg` + centered icon.
- **Primary button**: `bg-[#4F46E5] rounded-[18px]` with optional icon + white 600 text.
- **Input with label**: 600 label + a `border-[#DFE0E7] rounded-[18px]` box with a leading icon.
- **List card** (visits/products): header + expandable body ⇒ a component + data model; lazy
  list (mobile) / map (web).
- **Bottom navigation** (mobile): bottom bar with icon+label items, one active. On web this is
  usually a top nav / sidebar per the design.
- **Overlay over image**: a `bg-[...]/[0.95]` chip with `backdrop-blur` over a photo.

## 5. Per-screen fidelity checklist

- [ ] Exact colors and alpha (don't approximate hex).
- [ ] Family + size + weight + line-height + tracking per text block.
- [ ] Correct radii and shadows (including the 9999 pill).
- [ ] Icons = the exported SVGs (or the Lucide package on web), correct size and color.
- [ ] Hierarchy via idiomatic layouts (no absolute offsets).
- [ ] Scroll if the page exceeds the height (mobile) / natural scroll (web).
- [ ] **Web:** responsive per the chosen option (breakpoints / fixed width / frames).
- [ ] **Mobile:** system status bar (not the simulated one) and safe areas.
- [ ] Repeated structures extracted to a component + data.
- [ ] Compared against the screen's PNG.
