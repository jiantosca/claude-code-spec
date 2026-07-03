# Tasks — Dark Mode Toggle

> Example spec. Delete the `example-feature/` folder once your team has seen it.

- [ ] 1. Add `theme.css` with light/dark CSS variables under `[data-theme]`.
  Expected: switching `data-theme` on `<html>` visibly changes colors.
  _Requirements:_ R1
  _Depends:_

- [ ] 2. Create `ThemeProvider` with context, holding `theme` state and a
  `setTheme` that writes to localStorage (try/catch).
  Expected: provider exposes `{ theme, setTheme }`; bad storage doesn't throw.
  _Requirements:_ R1, R2, R5
  _Depends:_

- [ ] 3. Create `useTheme()` hook returning the context value.
  Expected: any component can read/set theme.
  _Requirements:_ R1
  _Depends:_ 2

- [ ] 4. Create `ThemeToggle` button that calls `setTheme(next)`.
  Expected: clicking flips theme with no reload.
  _Requirements:_ R1
  _Depends:_ 1, 3

- [ ] 5. Add the inline pre-paint script to `index.html` (stored theme, else OS
  `prefers-color-scheme`).
  Expected: correct theme on first paint, no flash.
  _Requirements:_ R3, R4
  _Depends:_ 1

- [ ] 6. Manual test pass against R1–R5 (toggle, reload persistence, fresh
  profile OS match, storage-disabled fallback).
  Expected: all five acceptance criteria hold.
  _Requirements:_ R1, R2, R3, R4, R5
  _Depends:_ 4, 5
