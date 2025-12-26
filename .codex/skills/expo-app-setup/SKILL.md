---
name: expo-app-setup
description: Guidance for building, refactoring, and debugging Expo + React Native apps (including Expo Router). Use when wiring screens/layouts, navigation (tab/stack) scaffolding with `_layout.tsx` per group, avoiding unwanted redirects in `app/index.tsx`, theming, data fetching with React Query/fetch, Expo module usage, offline handling, and running local Expo tooling (install/start/lint).
license: MIT
compatibility: Expo CNG project with Expo Router + TypeScript strict; needs Expo CLI/bun/npm for installs and linting; no extra system packages beyond tooling defaults.
metadata:
  author: amannhimself.dev
---

# Expo App Setup

## Overview

Actionable playbook for adding features, managing boilerplate code, implementing modern navigation patterns using Expo Router, state management, fixing bugs, and shipping UI and logic in cross-platform mobile applications using Expo and React Native projects.

## Navigation guardrails (avoid common mistakes)

- Every stack group needs its own `_layout.tsx`: root `app/_layout.tsx`, `app/(tabs)/_layout.tsx`, and a `_layout.tsx` inside each tab folder.
- Root `app/index.tsx` should render a landing screen; only add `Redirect` when explicitly requested.
- When adding a tab: create `app/(tabs)/<tab>`, add `_layout.tsx` with a `Stack` (headers off unless needed), add `index.tsx`, then register the tab in `app/(tabs)/_layout.tsx`.
- Keep imports ordered (React/React Native, third-party, then `@/` aliases) and follow repo style (2-space indent, double quotes, semicolons).

Canonical scaffold:

```txt
src/app/
  _layout.tsx          // root Stack -> (tabs)
  index.tsx            // landing screen (no redirect unless requested)
  (tabs)/
    _layout.tsx        // Tabs navigator
    home/
      _layout.tsx      // Stack for Home tab
      index.tsx
    profile/
      _layout.tsx      // Stack for Profile tab
      index.tsx
```

Minimal layout snippets:

```ts
// app/_layout.tsx
import { Stack } from "expo-router";

export default function RootLayout() {
  return (
    <Stack screenOptions={{ headerShown: false }}>
      <Stack.Screen name="(tabs)" />
    </Stack>
  );
}
```

```ts
// app/(tabs)/_layout.tsx
import { Tabs } from "expo-router";

export default function TabLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="home"
        options={{ title: "Home", headerShown: false }}
      />
      <Tabs.Screen
        name="profile"
        options={{ title: "Profile", headerShown: false }}
      />
    </Tabs>
  );
}
```

```ts
// app/(tabs)/home/_layout.tsx
import { Stack } from "expo-router";

export default function HomeStack() {
  return <Stack screenOptions={{ headerShown: false }} />;
}
```

```ts
// app/index.tsx
import { View, Text, StyleSheet } from "react-native";

export default function IndexScreen() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>MovieKnight</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, alignItems: "center", justifyContent: "center" },
  title: { fontSize: 24, fontWeight: "600" },
});
```

## Core workflow

- Clarify the task: screen/flow/bug, target platforms (iOS/Android/web), offline/low-connectivity expectations.
- Locate context: relevant route file, component, provider, and shared utilities/theme.
- Implement using the patterns below; reuse shared components and helpers before adding new ones.
- Verify: lint, run the flow on target platforms, and check loading/error/offline paths.
- Identify routing system (Expo Router vs plain navigation). Mirror existing patterns and folder structure.

## Quick start

- For package manager of choice, use `bun` or `bunx`.
- Install dependencies with project's tool (`bunx expo install` or equivalent).
- To start the development server locally, use `bunx expo start` or `bunx expo start -c`.
- Lint before shipping (`bunx expo lint` or the project’s lint script).

## Patterns

- **Images/media**: Build URLs via helpers (e.g., `makeImageUrl`); handle null paths; use `expo-image`.
- **Platform concerns**: Use `Platform.select` for variant styling/behavior. Avoid Node-only modules; keep code Expo-compatible.
- **Animations**: Keep animations within Reanimated limits; wrap UI in performant components when needed.
- **State and props**: Type all props; use `PropsWithChildren` instead of `ReactNode` where linted. Keep derived state memoized; avoid heavy work in render.
- **Accessibility**: Add labels on non-obvious pressables, ensure comfortable hit areas, and keep touch targets platform-appropriate.

## Instructions

### 1. Project Setup and Navigation

#### Basic setup

```tsx
// app/_layout.tsx
// Create root layout for the app
import { Stack, useRouter } from "expo-router";

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
    </Stack>
  );
}
```

#### Create a tab navigator

```tsx
// app/(tabs)/_layout.tsx
// Create a tab navigator and use the NativeTabs component from expo-router/unstable-native-tabs
import Ionicons from "@expo/vector-icons/Ionicons";
import {
  Icon,
  Label,
  NativeTabs,
  VectorIcon,
} from "expo-router/unstable-native-tabs";
import { Platform } from "react-native";

export default function TabLayout() {
  return (
    <NativeTabs
      backgroundColor={Platform.select({
        android: "#FFFFFF",
      })}
      minimizeBehavior="onScrollDown"
      iconColor={Platform.select({
        android: {
          default: "#000000",
          selected: "#007AFF",
        },
      })}
      labelVisibilityMode="labeled"
      labelStyle={Platform.select({
        android: {
          default: {
            color: "#000000",
          },
          selected: {
            color: "#007AFF",
          },
        },
      })}
      indicatorColor={Platform.select({
        android: "#E9F3FF",
      })}
    >
      {/* Home tab will always be a nested stack */}
      <NativeTabs.Trigger name="(home)">
        {Platform.select({
          ios: <Icon sf={{ default: "film", selected: "film.fill" }} />,
          android: <Icon src={<VectorIcon family={Ionicons} name="film" />} />,
        })}
        <Label>Home</Label>
      </NativeTabs.Trigger>
      <NativeTabs.Trigger name="settings">
        {Platform.select({
          ios: (
            <Icon
              sf={{ default: "gear.circle", selected: "gear.circle.fill" }}
            />
          ),
          android: (
            <Icon src={<VectorIcon family={Ionicons} name="settings" />} />
          ),
        })}
        <Label>Settings</Label>
      </NativeTabs.Trigger>
      {/* Optional */}
      <NativeTabs.Trigger name="search" role="search">
        {Platform.select({
          ios: (
            <Icon
              sf={{ default: "magnifyingglass", selected: "magnifyingglass" }}
            />
          ),
          android: (
            <Icon src={<VectorIcon family={Ionicons} name="search" />} />
          ),
        })}
        <Label>Search</Label>
      </NativeTabs.Trigger>
    </NativeTabs>
  );
}
```

#### Boilerplate screens

```tsx
// app/(tabs)/(home)/index.tsx
// Create a home screen
import { Link } from "expo-router";
import { View } from "react-native";

export default function Home() {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: "center",
        alignItems: "center",
      }}
    >
      <Link href="/[id]/1">Go to details</Link>
    </View>
  );
}
```

```tsx
// app/(tabs)/(home)/[id].tsx
// Create a details screen
import { useLocalSearchParams } from "expo-router";
import { Text, View } from "react-native";

export default function Details() {
  const { id } = useLocalSearchParams<{ id: string }>();

  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Text>Details for {id}</Text>
    </View>
  );
}
```

```tsx
// app/(tabs)/(home)/_layout.tsx
// Create a layout for the home screen
import { Stack } from "expo-router";
import { Platform } from "react-native";

function getIOSVersion(): number {
  if (Platform.OS !== "ios") return 0;

  return parseInt(Platform.Version as string, 10);
}

function isIOS26OrLater(): boolean {
  return getIOSVersion() >= 26;
}

export default function HomeLayout() {
  return (
    <Stack>
      <Stack.Screen
        name="index"
        options={{
          title: "Home",
          headerLargeTitle: true,
          headerTransparent: Platform.OS === "ios",
          headerBlurEffect: isIOS26OrLater() ? undefined : "regular",
        }}
      />
      <Stack.Screen
        name="[id]"
        options={{
          title: "Movie Details",
          headerTransparent: Platform.OS === "ios",
          headerBlurEffect: isIOS26OrLater() ? undefined : "regular",
        }}
      />
    </Stack>
  );
}
```

```tsx
// app/(tabs)/settings/index.tsx
// Create a settings screen
import { Text, View } from "react-native";

export default function Settings() {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: "center",
        alignItems: "center",
      }}
    >
      <Text>Edit app/(tabs)/settings.tsx to edit this screen.</Text>
    </View>
  );
}
```

```tsx
// app/(tabs)/settings/_layout.tsx
// Create a layout for the settings screen
import { Stack } from "expo-router";

export default function SettingsLayout() {
  return (
    <Stack>
      <Stack.Screen
        name="index"
        options={{
          title: "Settings",
          headerLargeTitle: true,
          headerTransparent: true,
        }}
      />
    </Stack>
  );
}
```

```tsx
// app/(tabs)/search/index.tsx
// Create a search screen
import { Text, View } from "react-native";

export default function Search() {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: "center",
        alignItems: "center",
      }}
    >
      <Text>Edit app/(tabs)/search/index.tsx to edit this screen.</Text>
    </View>
  );
}
```

```tsx
// app/(tabs)/search/_layout.tsx
// Create a layout for the search screen
import { Stack } from "expo-router";

export default function SearchLayout() {
  return (
    <Stack>
      <Stack.Screen
        name="index"
        options={{
          title: "Search",
          headerSearchBarOptions: {
            placeholder: "Search ...",
            placement: "automatic",
            onChangeText: () => {},
          },
        }}
      />
    </Stack>
  );
}
```

### 2. API integration

Add all config and API calls in the `services` folder.

```tsx
// services/config.ts

// Boilerplate
export const API_KEY = "";
export const API_BASE_URL = "";

// For example apps using TMDB API
export const TMDB_API_BASE_URL = "https://api.themoviedb.org/3";
export const TMDB_API_KEY = "";

export const TMDB_IMAGE_BASE_URL = "https://image.tmdb.org/t/p";
export const TMDB_POSTER_SIZE = "w500";
export const TMDB_PROFILE_SIZE = "w185";

export const makeImageUrl = (
  path?: string | null,
  size = TMDB_POSTER_SIZE
): string | null => {
  if (!path) return null;
  return `${TMDB_IMAGE_BASE_URL}/${size}${path}`;
};
```

```tsx
// services/index.ts
// Generic helper (optional)
// import { API_BASE_URL, API_KEY } from "./config";
// export const fetchData = async (url: string) => {
//   const response = await fetch(`${API_BASE_URL}${url}?api_key=${API_KEY}`);
//   if (!response.ok) {
//     throw new Error(`API request failed: ${response.status}`);
//   }
//   return response.json();
// };

// TMDB example
import { TMDB_API_BASE_URL, TMDB_API_KEY } from "./config";

export type Movie = {
  id: number;
  title: string;
  overview: string;
  poster_path: string | null;
  backdrop_path: string | null;
  release_date: string;
  vote_average: number;
  runtime: number | null;
};
type PopularMoviesResponse = {
  results: Movie[];
};

export type CastMember = {
  id: number;
  name: string;
  character: string;
  profile_path: string | null;
  order: number;
};

export type CrewMember = {
  id: number;
  name: string;
  job: string;
  department: string;
  profile_path: string | null;
};

export type CreditsResponse = {
  id: number;
  cast: CastMember[];
  crew: CrewMember[];
};

/**
 * Fetches a single page of TMDB's "popular" movies list.
 */
export const fetchPopularMovies = async (page = 1): Promise<Movie[]> => {
  const response = await fetch(
    `${TMDB_API_BASE_URL}/movie/popular?language=en-US&page=${page}&api_key=${TMDB_API_KEY}`
  );

  if (!response.ok) {
    throw new Error(`TMDB popular movies request failed: ${response.status}`);
  }

  const data = (await response.json()) as PopularMoviesResponse;

  return data.results;
};

export const popularMoviesQuery = (page = 1) => ({
  queryKey: ["popularMovies", page],
  queryFn: () => fetchPopularMovies(page),
});

/*
 * Fetches the details of a single movie.
 */
export const fetchMovieDetails = async (movieId: number) => {
  const response = await fetch(
    `${TMDB_API_BASE_URL}/movie/${movieId}?language=en-US&api_key=${TMDB_API_KEY}`
  );

  if (!response.ok) {
    throw new Error(`TMDB movie details request failed: ${response.status}`);
  }

  const data = (await response.json()) as Movie;

  return data;
};

export const movieDetailsQuery = (movieId: number) => ({
  queryKey: ["movieDetails", movieId],
  queryFn: () => fetchMovieDetails(movieId),
});
```

### 3. Project structure

Use a `src/` directory for everything, including the Expo Router `app/` folder (routes live at `src/app`). Update `tsconfig.json` paths to `@/*` -> `./src/*` and set the Expo Router app root (e.g., `EXPO_ROUTER_APP_ROOT=src/app` in env or `expo-router` plugin config) so routing resolves correctly.

```
├── assets/
└── src/
    ├── app/
    │   ├── (tabs)/
    │   └── _layout.tsx
    ├── components/
    ├── services/
    ├── providers/
    ├── constants/
    ├── utils/
    ├── hooks/
    └── types/
├── app.json/app.config.ts
├── eas.json
└── package.json
```

## Best practices

### DO

- Use functional components with React Hooks
- Implement proper error handling and loading states
- Use React Query for data fetching and state management
- Leverage Expo Router for routing
- Optimize list rendering with `FlatList`
- Handle platform-specific code elegantly
- Use TypeScript for type safety; keep shared domain types in a common `types/` module and keep component-prop types next to the component
- Test on both iOS and Android platforms
- Implement proper memory management

### DON'T

- Use inline styles excessively (use StyleSheet)
- Make API calls without error handling
- Store sensitive data in plain text
- Ignore platform differences
- Create large monolithic components
- Use index as key in lists
- Make synchronous operations
- Ignore battery optimization
- Deploy without testing on real devices
- Forget to unsubscribe from listeners

## Verification

- Pre-flight: confirm `_layout.tsx` exists at root, tab container, and each tab folder; ensure `app/index.tsx` matches requested behavior (screen by default, redirect only when asked); align imports; run lint.
- Lint before shipping (`bun run lint` or the project’s lint script). Fix type errors.
- Run the affected flow on target platforms. Test offline/poor network if you touched requests.
- Check for regressions in navigation (back/replace), loading/error states, and skeletons/placeholders.

## Common commands (adapt to project scripts)

- Install: `bunx expo install` / `npx expo install` (match repo)
- Start: `bunx expo start` (or `npx expo start --ios/--android/--web`)
- Lint: `bunx expo lint` (or `npx expo lint`)

## References

- Router scaffolding checklist: `references/router-scaffolding.md`.
