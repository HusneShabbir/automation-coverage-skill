# Plugin Playwright

Use this only after `generate_playwright_brief`. The brief is the scenario. The browser is the source of the locators.

## Explore, then write

For each brief, with Playwright MCP (`--caps=testing`):

1. `browser_navigate` to the URL in the brief.
2. `browser_snapshot` before acting.
3. Interact (`browser_click`, `browser_type`, `browser_fill_form`) the way a user would.
4. `browser_generate_locator` for every element the spec will touch.
5. `browser_verify_element_visible` or `browser_verify_text_visible` for the assertion.
6. Then write `@playwright/test` TypeScript at `outputPath`, mirroring the brief's template (page objects, `getTranslations()`, `APP_MODE`).
7. Run `runCommand` (often `APP_MODE=legacy npx playwright test --project=en`). Iterate until it passes. `browser_close` when done.

## Locators

- Prefer user-facing locators (role, label, text) over CSS or XPath.
- Do not pin a specific heading level.
- Labels that include a count, such as `All (22)`, should omit the count unless the test just changed that count to a known number.

## Internationalization

If the plugin has a `translations` folder (usually `src/translations` or `src/alpha/translations`):

- `ref.ts` exports the English messages. The export name ends with `Messages`.
- Locale files (`fr.ts`, …) default-export a `messages` object, usually flat dot keys. `{{name}}` is a replacement template.
- Add a test util that loads `ref.ts` as `en`, flattens each locale into the same shape, returns messages for a language code, and fills templates.
- In `playwright.config.ts`, add one project per locale. Set the browser locale; keep the rest of the project the same.
- Before assertions, read `browserName` / locale from the project, load those messages, open settings, and select that language.
- Assert translated strings from the loaded messages, not hardcoded English.

## Harness

- New spec files must be included by the workspace `tsconfig.json`.
- When Playwright `webServer` starts the app, wait on a stdout line such as `Plugin initialization complete` instead of a fixed sleep.
- Add `@axe-core/playwright`. One shared helper runs `AxeBuilder` and asserts no violations. Call it on every new page the plugin contributes.
- If ESLint fails on the specs or `playwright.config.ts`, add those paths to `.eslintignore`.
