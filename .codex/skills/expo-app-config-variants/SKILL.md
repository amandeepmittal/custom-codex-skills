---
name: expo-app-config-variants
description: Convert app.json to app.config.ts and manage multiple app variants (dev/staging/prod) in Expo projects using expo-dev-client. Use when introducing typed config, variant-specific identifiers/schemes/icons/env, or building custom dev clients.
license: MIT
metadata:
  author: amannhimself.dev
---

# Expo App Config & Variants

## Overview
How to migrate from `app.json` to `app.config.ts` and define multiple variants (dev/staging/prod) that work with `expo-dev-client`, bundlers, and EAS builds.

## Quick start
- Ensure `expo-dev-client` is installed (check `package.json` deps). If missing: `bunx expo install expo-dev-client`.
- Rename `app.json` to `app.config.ts` (keep contents as a starting point).
- Set an env flag for variants (e.g., `APP_VARIANT=dev` or use `.env` + scripts).
- Run with a variant: `APP_VARIANT=dev bunx expo start` (or set in your shell/env).
- Replace placeholders in the template (name/slug/scheme/bundle IDs/package IDs/extra) with your app’s values before building.

## Workflow
1) **Convert config**: create `app.config.ts` exporting a function; remove/ignore `app.json`. Update `package.json` scripts if they referenced `app.json`.  
2) **Define variants**: keep a base config and merge variant overrides (name/slug/scheme/bundle IDs/package IDs, icons, `extra`, runtime values).  
3) **Wire env**: read `process.env.APP_VARIANT ?? "dev"`; default to `dev`.  
4) **Dev client**: build a client per platform/profile so native IDs match the variant you’re testing.  
5) **Start apps**: set `APP_VARIANT` when running `expo start` or `expo run:*` so the right config is applied.

## Template: app.config.ts
```ts
import { ConfigContext, ExpoConfig } from "@expo/config";

type Variant = "dev" | "staging" | "prod";

const variants: Record<Variant, Partial<ExpoConfig>> = {
  dev: {
    name: "MyApp Dev",
    slug: "myapp-dev",
    scheme: "myapp-dev",
    ios: { bundleIdentifier: "com.example.myapp.dev" },
    android: { package: "com.example.myapp.dev" },
    extra: { apiUrl: "https://api-dev.example.com" },
  },
  staging: {
    name: "MyApp Staging",
    slug: "myapp-staging",
    scheme: "myapp-staging",
    ios: { bundleIdentifier: "com.example.myapp.staging" },
    android: { package: "com.example.myapp.staging" },
    extra: { apiUrl: "https://api-staging.example.com" },
  },
  prod: {
    name: "MyApp",
    slug: "myapp",
    scheme: "myapp",
    ios: { bundleIdentifier: "com.example.myapp" },
    android: { package: "com.example.myapp" },
    extra: { apiUrl: "https://api.example.com" },
  },
};

export default ({ config }: ConfigContext): ExpoConfig => {
  const variant = (process.env.APP_VARIANT as Variant) || "dev";
  const override = variants[variant] ?? variants.dev;
  return {
    ...config,
    ...override,
    extra: {
      ...config.extra,
      ...override.extra,
      variant,
    },
  };
};
```

## Dev client and variants
- Build a client per variant/profile so native IDs match: `APP_VARIANT=dev bunx expo run:ios --device` or `bunx expo run:android` (or EAS: `APP_VARIANT=staging bunx eas build --profile development --platform ios`).
- If using EAS, set `env` per profile in `eas.json` (e.g., `APP_VARIANT`, API keys). Ensure `runtimeVersion`/`appVersion` stay consistent per channel if you use OTA.

## Tips
- Icons/splash per variant: add platform assets per variant and merge in `variants[...]` overrides (e.g., `icon`, `ios.icon`, `android.adaptiveIcon`).  
- Schemes/deeplinks: ensure `scheme` and `intentFilters` match each bundle/package ID.  
- Caches: if config changes aren’t picked up, try `bunx expo start -c`.  
- Keep secrets out of `app.config.ts`; inject via env and read with `process.env` or `extra`.  
