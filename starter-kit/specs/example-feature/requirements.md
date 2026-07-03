# Requirements — Dark Mode Toggle

> Example spec. Delete the `example-feature/` folder once your team has seen it.

## User stories

**US1 — Toggle theme.** As a user, I want to switch between light and dark
themes, so that I can read comfortably in different lighting.

**US2 — Remember my choice.** As a returning user, I want my theme to persist,
so that I don't re-select it every visit.

**US3 — Respect system preference.** As a first-time user, I want the app to
match my OS theme by default, so that it feels native.

## Acceptance criteria (EARS)

- **R1** — WHEN the user clicks the theme toggle THE SYSTEM SHALL switch the
  active theme between light and dark immediately, without a page reload.
- **R2** — WHEN the user selects a theme THE SYSTEM SHALL persist that choice to
  local storage.
- **R3** — WHEN a returning user loads the app AND a stored theme exists THE
  SYSTEM SHALL apply the stored theme before first paint.
- **R4** — WHEN a first-time user loads the app AND no stored theme exists THE
  SYSTEM SHALL apply the theme matching the OS `prefers-color-scheme` setting.
- **R5** — IF local storage is unavailable THE SYSTEM SHALL fall back to the OS
  preference and SHALL NOT throw.
