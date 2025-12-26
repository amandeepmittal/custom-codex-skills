---
name: expo-react-query-setup
description: Install and wire @tanstack/react-query in Expo/React Native apps (providers, query client, fetch patterns, and screen usage). Use when adding React Query to a project or extending data fetching patterns.
license: MIT
metadata:
  author: amannhimself.dev
---

# Expo React Query Setup

## Overview

How to install, configure, and use @tanstack/react-query in Expo/React Native projects.

## Quick start

- Install deps: `bunx expo install @tanstack/react-query` if a `bun.lock` file is present.
- Create a shared `queryClient` and wrap the app with `QueryClientProvider`.
- Use array query keys and export `fetchX` + `xQuery` helpers for reuse.

## Provider setup (app entry)

```tsx
// app/_layout.tsx (Expo Router example)
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient();

export default function RootLayout() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* your navigation/providers */}
    </QueryClientProvider>
  );
}
```

## Service + query helper pattern

```ts
// services/movies.ts
const API_BASE = "https://api.example.com";

export type Movie = { id: number; title: string };

export const fetchMovies = async (): Promise<Movie[]> => {
  const res = await fetch(`${API_BASE}/movies`);
  if (!res.ok) throw new Error(`Movies request failed: ${res.status}`);
  return res.json();
};

export const moviesQuery = () => ({
  queryKey: ["movies"],
  queryFn: fetchMovies,
});
```

## Screen usage

```tsx
import { useQuery } from "@tanstack/react-query";
import { moviesQuery } from "@/services/movies";

export default function MoviesScreen() {
  const { data, isLoading, error } = useQuery(moviesQuery());
  if (isLoading) return <LoadingView />;
  if (error) return <ErrorView message={error.message} />;
  return <MoviesList movies={data ?? []} />;
}
```

## Tips

- Keep query keys stable and array-based; include params (e.g., `["movie", id]`).
- For mutations, invalidate or refetch related queries after success.
- If you have an offline modal/provider, read connectivity before firing heavy requests.
- Use `staleTime`/`cacheTime` to tune refetching; default is fine for many screens.
- Clear cache with `queryClient.clear()` only in exceptional cases (e.g., logout).

## Offline modal + provider (optional)

- Install: `bunx expo install expo-network` (and keep @tanstack/react-query installed).
- Connectivity provider (create `providers/ConnectivityProvider.tsx`):

```ts
import { onlineManager } from "@tanstack/react-query";
import * as Network from "expo-network";
import {
  createContext,
  PropsWithChildren,
  useCallback,
  useContext,
  useEffect,
  useState,
} from "react";
import { AppState, AppStateStatus } from "react-native";

type ConnectivityContextValue = {
  isOnline: boolean;
  refresh: () => Promise<boolean>;
};

const ConnectivityContext = createContext<ConnectivityContextValue | undefined>(
  undefined
);

const deriveOnlineStatus = (
  state: Network.NetworkState | null | undefined
): boolean => {
  if (!state) return true;
  if (state.isInternetReachable === false) return false;
  return Boolean(state.isConnected);
};

export const ConnectivityProvider = ({ children }: PropsWithChildren) => {
  const [isOnline, setIsOnline] = useState(true);

  const applyState = useCallback((state: Network.NetworkState | null) => {
    const online = deriveOnlineStatus(state);
    setIsOnline(online);
    onlineManager.setOnline(online);
  }, []);

  const refresh = useCallback(async () => {
    try {
      const state = await Network.getNetworkStateAsync();
      applyState(state);
      return deriveOnlineStatus(state);
    } catch {
      return isOnline;
    }
  }, [applyState, isOnline]);

  useEffect(() => {
    refresh();
  }, [refresh]);

  useEffect(() => {
    const subscription = Network.addNetworkStateListener(applyState);
    const handleAppStateChange = (status: AppStateStatus) => {
      if (status === "active") refresh();
    };
    const appStateSubscription = AppState.addEventListener(
      "change",
      handleAppStateChange
    );
    return () => {
      subscription.remove();
      appStateSubscription.remove();
    };
  }, [applyState, refresh]);

  return (
    <ConnectivityContext.Provider value={{ isOnline, refresh }}>
      {children}
    </ConnectivityContext.Provider>
  );
};

export const useConnectivity = () => {
  const ctx = useContext(ConnectivityContext);
  if (!ctx)
    throw new Error("useConnectivity must be used within ConnectivityProvider");
  return ctx;
};
```

- Offline UI (create `components/OfflineModal.tsx` and export from your components index):
  - If you have a custom Text component/alias (e.g., `@/components/Text`), update the import accordingly; otherwise use `import { Text } from "react-native"`.

```tsx
import MaterialIcons from "@expo/vector-icons/MaterialIcons";
import { SymbolView } from "expo-symbols";
import {
  ActivityIndicator,
  Modal,
  Platform,
  Pressable,
  StyleSheet,
  View,
} from "react-native";
import { Text } from "./Text"; // change to your project’s Text component or react-native Text

type OfflineNoticeProps = {
  onRetry?: () => Promise<void> | void;
  isChecking?: boolean;
};
type OfflineModalProps = OfflineNoticeProps & { visible: boolean };

export const OfflineNotice = ({ onRetry, isChecking }: OfflineNoticeProps) => (
  <View style={styles.card}>
    <View style={styles.iconBadge}>
      {Platform.OS === "ios" ? (
        <SymbolView
          name="wifi.slash"
          tintColor="#ef4444"
          style={{ width: 26, height: 26 }}
        />
      ) : (
        <MaterialIcons name="wifi-off" size={26} color="#ef4444" />
      )}
    </View>
    <Text style={styles.title}>You are offline</Text>
    <Text style={styles.subtitle}>
      Connect to Wi-Fi or cellular data to continue browsing.
    </Text>
    {onRetry ? (
      <Pressable
        style={({ pressed }) => [
          styles.button,
          pressed && styles.buttonPressed,
          isChecking && styles.buttonDisabled,
        ]}
        onPress={onRetry}
        disabled={isChecking}
        accessibilityRole="button"
        accessibilityLabel="Retry connection"
      >
        {isChecking ? (
          <ActivityIndicator color="#fff" />
        ) : (
          <Text style={styles.buttonLabel}>Retry</Text>
        )}
      </Pressable>
    ) : null}
  </View>
);

export function OfflineModal({
  visible,
  onRetry,
  isChecking,
}: OfflineModalProps) {
  return (
    <Modal
      animationType="fade"
      transparent
      visible={visible}
      statusBarTranslucent
    >
      <View style={styles.backdrop}>
        <OfflineNotice onRetry={onRetry} isChecking={isChecking} />
      </View>
    </Modal>
  );
}

const styles = StyleSheet.create({
  backdrop: {
    flex: 1,
    backgroundColor: "rgba(0,0,0,0.5)",
    justifyContent: "center",
    alignItems: "center",
    padding: 24,
  },
  card: {
    width: "100%",
    paddingVertical: 22,
    paddingHorizontal: 20,
    borderRadius: 12,
    backgroundColor: "#fff",
    alignItems: "center",
    gap: 12,
    borderWidth: 1,
    borderColor: "#E5E7EB",
  },
  iconBadge: {
    width: 44,
    height: 44,
    borderRadius: 22,
    backgroundColor: "#fee2e2",
    alignItems: "center",
    justifyContent: "center",
  },
  title: { fontSize: 18, textAlign: "center" },
  subtitle: {
    fontSize: 14,
    textAlign: "center",
    lineHeight: 20,
    color: "#6b7280",
  },
  button: {
    marginTop: 4,
    backgroundColor: "#007AFF",
    paddingHorizontal: 18,
    paddingVertical: 11,
    borderRadius: 12,
    minWidth: 120,
    alignItems: "center",
  },
  buttonPressed: { opacity: 0.85 },
  buttonDisabled: { opacity: 0.65 },
  buttonLabel: { color: "#fff", fontWeight: "600" },
});
```

- Modal route (create `app/(modals)/offline.tsx`):

```tsx
import { useRouter } from "expo-router";
import { useState } from "react";
import { StyleSheet, View } from "react-native";
import { OfflineNotice } from "@/components/OfflineModal"; // adjust alias/import if not using @/
import { useConnectivity } from "@/providers/ConnectivityProvider"; // adjust alias/import if not using @/

export default function OfflineScreen() {
  const { refresh, isOnline } = useConnectivity();
  const router = useRouter();
  const [checking, setChecking] = useState(false);

  const handleRetry = async () => {
    setChecking(true);
    try {
      const online = await refresh();
      if (online || isOnline) {
        if (router.canGoBack()) router.back();
        else router.replace("/(tabs)");
      }
    } finally {
      setChecking(false);
    }
  };

  return (
    <View style={styles.container}>
      <OfflineNotice onRetry={handleRetry} isChecking={checking} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "rgba(0,0,0,0.4)",
    justifyContent: "center",
    alignItems: "center",
    padding: 24,
  },
});
```

- Layout guard (in `app/_layout.tsx`): after wrapping with `QueryClientProvider` and `ConnectivityProvider`, watch `isOnline` and `router.replace("/(modals)/offline")` when offline, so queries pause and users see the modal.
