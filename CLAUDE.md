# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

> **Expo SDK 56 is a hard requirement.** APIs in this version differ from older Expo/React
> Native knowledge. Confirm against https://docs.expo.dev/versions/v56.0.0/ before writing
> Expo/React Native code — do not rely on memory of earlier SDKs.

## What this project is

A **fresh Expo SDK 56 starter** (the `create-expo-app` default that nests source under `src/`),
checked in as the seed for a future calling app. Despite the `calling-mobile-app` name there is
**no telephony/calling code yet** — current surface is a welcome Home screen, an Explore screen,
a two-tab navigator, and a light/dark theming system. Build new features on top of this scaffold;
don't assume calling infrastructure exists.

Stack: Expo Router (file-based routing), React 19.2 + React Native 0.85, TypeScript (`strict`),
universal (iOS / Android / **web**). This is part of the larger `mainn/` SeloraX workspace
(see `../CLAUDE.md`), but it is an independent git repo on branch `master` and shares no tooling
with the sibling projects.

## Commands

```bash
npm install          # uses npm — package-lock.json is the committed lockfile
npm start            # expo start (dev server + QR / launcher)
npm run ios          # expo start --ios     (iOS simulator)
npm run android      # expo start --android (Android emulator)
npm run web          # expo start --web
npm run lint         # expo lint
npm run reset-project # moves current src/ to app-example/ and scaffolds a blank app — DESTRUCTIVE, avoid
```

No test runner is configured (`reset-project.js` is a scaffolding script, not a test).
Type-check via `npx tsc --noEmit`. There are no committed `ios/`/`android/` folders
(gitignored, see below) — native builds are prebuild/CNG-generated.

## Architecture

### File-based routing lives under `src/app/`
`package.json` `main` is `expo-router/entry`, and the project root is `src/` (path aliases below).
Routes:
- `src/app/_layout.tsx` — root layout. Notably it does **not** mount a `<Stack>`/`<Tabs>`
  navigator. It wraps the app in expo-router's `ThemeProvider` (Default/Dark by color scheme),
  drops an `<AnimatedSplashOverlay />`, then renders the custom `<AppTabs />`.
- `src/app/index.tsx` — Home (`/`), `src/app/explore.tsx` — Explore (`/explore`).

`app.json` enables two experiments that affect how you write code:
- **`typedRoutes`** — route hrefs are type-checked; new routes regenerate `.expo/types`.
- **`reactCompiler`** — the React Compiler auto-memoizes; **don't add manual
  `useMemo`/`useCallback`** unless profiling proves a need.

### Platform-split files (`.web.tsx` / `.tsx`) are a core pattern
Metro picks `*.web.*` on web and the plain file on native. Several modules ship two divergent
implementations — edit the right one (often **both**):
- `components/app-tabs.tsx` — native bottom tabs via `expo-router/unstable-native-tabs`
  (`NativeTabs`, PNG icons in `assets/images/tabIcons/`, `renderingMode="template"`).
- `components/app-tabs.web.tsx` — a custom floating pill built from `expo-router/ui`
  primitives (`Tabs`/`TabList`/`TabTrigger`/`TabSlot`) with brand text and a Docs link.
- `hooks/use-color-scheme.ts` (re-exports RN) vs `.web.ts` (adds a hydration guard returning
  `'light'` until mounted, for static web rendering).
- `components/animated-icon.tsx` / `.web.tsx` (+ `animated-icon.module.css` for web).

### Theming
- `constants/theme.ts` is the single source of truth: `Colors` (`light`/`dark` with keys
  `text`, `background`, `backgroundElement`, `backgroundSelected`, `textSecondary`), the
  `Spacing` scale (`half`…`six`), `Fonts` (platform-selected font-family vars), `BottomTabInset`,
  and `MaxContentWidth`. The `ThemeColor` type is derived from `Colors` keys.
- `hooks/use-theme.ts` resolves the active `Colors[scheme]` (treating `'unspecified'` as light).
- **Use the themed primitives, not raw RN ones**: `ThemedText` (variant via `type`, e.g.
  `title`/`small`/`code`/`link`, color via `themeColor`) and `ThemedView` (background via `type`).
  Layout should reference `Spacing.*` constants rather than hard-coded numbers.
- Web typography comes from CSS variables defined in `src/global.css` (imported at the top of
  `theme.ts`), consumed via `Fonts.web` `var(--font-*)` values.

### Path aliases (`tsconfig.json`)
- `@/*` → `src/*`
- `@/assets/*` → `assets/*`
Always import via aliases (e.g. `@/components/themed-text`, `@/constants/theme`,
`require('@/assets/images/...')`) — relative `../` imports are the exception in this codebase.

## Conventions

- **Filenames are kebab-case** (`themed-text.tsx`, `use-color-scheme.ts`); React components are
  PascalCase exports.
- `assets/` sits at the repo root (not under `src/`); reference images with the `@/assets/*` alias.
- `ios/` and `android/` are **gitignored** (generated). Native config changes go in `app.json`
  (and config plugins), then are realized via `expo prebuild` — don't hand-edit generated native
  folders expecting them to persist.
