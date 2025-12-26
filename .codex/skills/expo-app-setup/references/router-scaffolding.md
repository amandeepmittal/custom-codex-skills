# Router Scaffolding Checklist

Use this when creating or fixing Expo Router navigation so tab stacks are wired correctly and root screens behave as expected.

## Common mistakes to avoid
- Missing `_layout.tsx` inside each tab folder under `app/(tabs)/`; every stack needs one.
- Auto-generated `app/index.tsx` that just redirects; only add `Redirect` when explicitly asked.
- Registering a tab in `app/(tabs)/_layout.tsx` without a backing folder/screen.
- Leaving headers enabled on nested stacks when the tab container should own headers.

## Canonical folder shape
```
app/
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

## Add a tab (repeatable steps)
1) Make `app/(tabs)/<tab>/` and add `_layout.tsx` with `<Stack screenOptions={{ headerShown: false }} />`.
2) Add `index.tsx` (and any detail screens) inside the tab folder.
3) Register the tab in `app/(tabs)/_layout.tsx` with `Tabs.Screen name="<tab>" options={{ headerShown: false }}`.
4) Keep `app/index.tsx` as a real screen unless the task says to redirect.

## Validation pass
- Confirm `_layout.tsx` exists at: `app/_layout.tsx`, `app/(tabs)/_layout.tsx`, and each `app/(tabs)/*/_layout.tsx`.
- Confirm `app/index.tsx` renders a screen by default.
- Run `npm run lint` before handing off.
