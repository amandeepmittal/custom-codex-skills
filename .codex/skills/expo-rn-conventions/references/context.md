# React Native + Expo Context (Do and Don'ts)

This document captures reusable patterns for React Native + Expo projects.

## Tooling

Do:

- Use bun for installs and running scripts.
- Use bunx for one-off tooling; when installing Expo packages, use `bunx expo install <package>`.
- Keep `bun.lock` committed and in sync with `package.json`.

Don't:

- Do not use npm/yarn/pnpm in this project to avoid lockfile churn.

## Project structure and imports

Do:

- Keep app routes under `src/app` with Expo Router and route groups like `(tabs)`.
- Keep shared UI in `src/components`, themes in `src/theme`, data in `src/data`, utilities in `src/utils`.
- Use the `@/*` path alias for imports from `src`.
- Prefer named function declarations for components and hooks.
- Use `PropsWithChildren` for children props and type-only imports for types.

Don't:

- Do not add `import React from "react";` in components or screens.
- Do not use deep relative imports like `../../..`.

## Navigation and layout

Do:

- Keep `package.json` `main` set to `expo-router/entry`.
- Use Expo Router `NativeTabs` (`expo-router/unstable-native-tabs`) for the bottom bar.
- Use a `Stack` layout inside each tab to configure headers.
- On iOS, use `headerLargeTitle` and `headerTransparent` where appropriate.

Don't:

- Do not wrap the app in `SafeAreaProvider`; use `SafeAreaView` at the screen level only.
- Do not wrap the app in `GestureHandlerRootView` unless a gesture component requires it.

## Theming and UI primitives

Do:

- Use `ThemeProvider` at the app root and `useTheme` in components.
- Keep tokens in `src/theme/theme.ts` and reference `theme.colors`, `theme.spacing`, `theme.radii`, and `theme.shadow`.
- Use shared primitives (`Screen`, `Surface`, `Text`, `Button`, `SettingsRow`, `SettingsSection`).
- Keep touch targets >= 44 and provide pressed state feedback in `Pressable` style callbacks.
- Use `StyleSheet.hairlineWidth` for subtle borders and `theme.shadow.card` for surfaces.

Don't:

- Do not introduce a new styling system outside `src/theme` and `src/components`.
- Do not hardcode colors outside small, local palettes (for example, color swatches).

## Icons

Do:

- Use `expo-symbols` with `SymbolView` and SF Symbols on iOS (prefer circle variants for tabs).
- Use `@expo/vector-icons/MaterialIcons` on other platforms; select with `Platform.select`.
- Use typed `SFSymbol` names when applicable.

Don't:

- Do not mix icon systems within the same platform UI.

## Data and storage (expo-sqlite)

Do:

- Use `openDatabaseAsync` and cache a single db instance.
- Apply migrations with `PRAGMA user_version` and `withTransactionAsync`.
- Enable `PRAGMA journal_mode = WAL` and `foreign_keys = ON`.
- Keep database logic in `src/data` and expose typed helpers.
- Call `ensureDailyReset` before reading or writing daily counts.

Don't:

- Do not run sqlite writes during render or keep queries inside components.

## UX, state, and accessibility

Do:

- Use `useFocusEffect` to refresh data when screens are focused.
- Use `useMemo` and `useCallback` for derived or shared logic.
- Use `Alert` for destructive actions and user-facing errors.
- Add `accessibilityRole`, `accessibilityState`, and labels to interactive controls.
- Use `expo-haptics` for primary feedback actions when it improves UX.

Don't:

- Do not allow negative counts or invalid input; validate in data and UI layers.

## App config

Do:

- Keep `userInterfaceStyle` set to `automatic` to respect the system theme.
- Keep `expo-router`, `expo-splash-screen`, and `expo-sqlite` plugins in `app.json`.

Don't:

- Do not add a custom `babel.config.js` unless there is a documented need.
