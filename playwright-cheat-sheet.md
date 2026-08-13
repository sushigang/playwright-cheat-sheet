# Playwright Cheat Sheet (TypeScript / JavaScript, `@playwright/test`)

## Install & scaffold

```bash
npm init playwright@latest        # scaffolds config, example tests, CI file
npm install -D @playwright/test   # add to an existing project
npx playwright install            # download browser binaries
npx playwright install --with-deps chromium   # one browser + OS deps (CI)
```

## Anatomy of a test

```ts
import { test, expect } from '@playwright/test';

test.describe('login', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/login');
  });

  test('shows error on bad password', async ({ page }) => {
    await page.getByLabel('Email').fill('a@b.com');
    await page.getByLabel('Password').fill('wrong');
    await page.getByRole('button', { name: 'Sign in' }).click();
    await expect(page.getByText('Invalid credentials')).toBeVisible();
  });
});
```

## Locators — priority order

Prefer user-facing, resilient locators over CSS/XPath — they describe what a person actually sees and interacts with, so they survive markup refactors and, as a side effect, nudge the app toward being more accessible.

| Locator | Matches | Use it when |
|---|---|---|
| `getByRole('button', { name })` | ARIA role + accessible name | **default first choice** for anything interactive — buttons, links, checkboxes, headings. Mirrors what a screen reader sees; if it can't find the element, that's often a real accessibility gap worth fixing, not just a flaky test |
| `getByLabel('Password')` | form control associated with a `<label>` | any input/textarea/select with a real `<label>` — the most reliable way to grab form fields |
| `getByPlaceholder('Search…')` | `placeholder` attribute | a fallback for inputs with no associated `<label>` — note placeholder-only inputs are themselves a mild accessibility smell, so this often signals something worth flagging to the app team |
| `getByText('Welcome back')` | rendered text content | static, non-interactive content — headings, confirmation banners, paragraph copy. Don't reach for it on a *clickable* element; use `getByRole` there even if you're matching the same text |
| `getByAltText('Company logo')` | `alt` attribute | images specifically |
| `getByTitle('Close')` | `title` attribute (tooltip) | rare — only when the element is genuinely identified by its tooltip and nothing else distinguishes it |
| `getByTestId('checkout-btn')` | `data-testid` (configurable attribute) | escape hatch when nothing above uniquely or stably identifies the element — icon-only buttons with no accessible name, dynamically generated rows, dense tables. Requires the app to actually set the attribute |
| `locator('.class-name')` | CSS class selector | last resort — ties the test to a styling class that's free to change for purely visual reasons (a designer renaming `.btn-primary` shouldn't break a test) |
| `locator('#element-id')` | CSS ID selector | one step better than class, since IDs are usually unique and less likely to be styling-driven, but still couples to markup rather than user-facing behavior. Fine for legacy apps where IDs are stable and meaningful |
| `locator('.css-selector')` | any other CSS selector (attribute, descendant, combinator) | last resort, same reasoning as class/ID — reach for it only when nothing semantic is available |
| `locator('xpath=//div[@id="x"]')` | XPath | avoid unless you're stuck with markup you can't improve — verbose, and the hardest to read back later |

```ts
page.getByRole('button', { name: 'Submit' })
page.getByLabel('Password')
page.getByPlaceholder('Search…')
page.getByText('Welcome back')
page.getByAltText('Company logo')
page.getByTitle('Close')
page.getByTestId('checkout-btn')
page.locator('.class-name')          // class, e.g. <div class="class-name">
page.locator('#element-id')          // ID, e.g. <div id="element-id">
page.locator('.css-selector')        // any other CSS selector
page.locator('xpath=//div[@id="x"]')
```

**Example — matching real markup**

```html
<button aria-label="Submit">Submit</button>

<label for="pwd">Password</label>
<input id="pwd" type="password" />

<input type="text" placeholder="Search…" />

<p>Welcome back</p>

<img src="logo.png" alt="Company logo" />

<button title="Close">×</button>

<button data-testid="checkout-btn">Checkout</button>

<div class="card">…</div>
<div id="summary">…</div>
```

```ts
page.getByRole('button', { name: 'Submit' })   // the aria-labelled button
page.getByLabel('Password')                    // input#pwd, via its <label for="pwd">
page.getByPlaceholder('Search…')               // the search input
page.getByText('Welcome back')                 // the <p>
page.getByAltText('Company logo')              // the <img>
page.getByTitle('Close')                       // the × button
page.getByTestId('checkout-btn')               // the checkout button
page.locator('.card')                          // div.card
page.locator('#summary')                       // div#summary
```

**Chaining & filtering**

| Method | Does | Use it when |
|---|---|---|
| `.filter({ hasText })` | narrows to elements containing given text | picking one item out of a repeated list — e.g. the row for "Pear" among many `listitem`s |
| `.filter({ has: locator })` | narrows to elements containing a matching descendant | the item itself has no unique text, but something inside it does — e.g. the row that contains an "Add" button |
| `.nth(n)` / `.first()` / `.last()` | selects by position | order is stable and meaningful (e.g. "first row of a sorted table") — avoid otherwise, since reordering silently breaks the test without pointing at why |
| `.and(locator)` | both conditions must match the same element | narrowing by a second attribute at once, e.g. a specific title *and* a specific role |
| `.or(locator)` | matches either locator | the same intent can render as one of two elements — e.g. a "Sign in" button on desktop vs. a menu item on mobile |

```ts
page.getByRole('listitem').filter({ hasText: 'Pear' })
page.getByRole('listitem').filter({ has: page.getByRole('button', { name: 'Add' }) })
page.locator('table').getByRole('row').nth(2)
page.getByRole('button').first()   // .last(), .nth(n)
locator.and(page.getByTitle('x'))  // narrow further
locator.or(otherLocator)           // match either
```

## Actions

```ts
await locator.click({ button: 'right', clickCount: 2, modifiers: ['Shift'] });
await locator.dblclick();
await locator.fill('text');                 // clears + sets value directly
await locator.pressSequentially('text');    // types char by char (triggers key events)
await locator.press('Enter');
await locator.check(); await locator.uncheck();
await locator.selectOption('value');        // or { label } / { index } / array for multi
await locator.hover();
await locator.dragTo(targetLocator);
await locator.setInputFiles('path/to/file.pdf');   // [] to clear
await locator.focus(); await locator.blur();
await locator.scrollIntoViewIfNeeded();
await page.mouse.wheel(0, 300);
await page.keyboard.down('Shift'); await page.keyboard.up('Shift');
```

## Assertions (`expect`) — auto-retrying, web-first

```ts
// Locator
await expect(locator).toBeVisible();      // .toBeHidden()
await expect(locator).toBeEnabled();      // .toBeDisabled()
await expect(locator).toBeChecked();
await expect(locator).toBeEditable();
await expect(locator).toBeEmpty();
await expect(locator).toBeFocused();
await expect(locator).toHaveText('exact');            // substring: {exact:false}
await expect(locator).toContainText('partial');
await expect(locator).toHaveValue('input value');
await expect(locator).toHaveAttribute('href', '/x');
await expect(locator).toHaveClass(/active/);
await expect(locator).toHaveCount(3);
await expect(locator).toHaveCSS('display', 'flex');
await expect(locator).toHaveScreenshot('name.png');   // visual regression

// Page
await expect(page).toHaveTitle(/Dashboard/);
await expect(page).toHaveURL(/\/orders\/\d+/);

// Not / soft / generic
await expect(locator).not.toBeVisible();
expect.soft(actual).toBe(expected);        // keeps running test on failure
expect(value).toBe(42);                    // plain (non-retrying) value assertion
```

## Waiting

Most actions and assertions **auto-wait** (actionability: visible, stable, enabled, receives events) — you rarely need manual waits. When you do:

```ts
await locator.waitFor({ state: 'visible' });     // 'attached' | 'detached' | 'hidden'
await page.waitForURL('**/success');
await page.waitForLoadState('networkidle');       // 'load' | 'domcontentloaded'
await page.waitForResponse(r => r.url().includes('/api/orders') && r.status() === 200);
await page.waitForRequest('**/api/**');
await page.waitForEvent('popup');
```
Avoid `page.waitForTimeout()` (arbitrary sleeps) except as a last-resort debugging aid.

## Navigation & multiple pages/tabs

```ts
await page.goto('/dashboard', { waitUntil: 'domcontentloaded' });
await page.goBack(); await page.goForward(); await page.reload();

const [newPage] = await Promise.all([
  context.waitForEvent('page'),
  page.getByRole('link', { name: 'Open in new tab' }).click(),
]);
await newPage.waitForLoadState();
```

## Network mocking / interception

```ts
await page.route('**/api/orders', async route => {
  await route.fulfill({ json: [{ id: 1, total: 42 }] });
});
await page.route('**/*.{png,jpg}', route => route.abort());
await page.route('**/api/slow', async route => {
  await route.continue();  // pass through, optionally after modifying request
});
await page.unroute('**/api/orders');
await page.routeFromHAR('recorded.har');   // replay a HAR file

// Direct API calls (no browser) via the `request` fixture
test('api', async ({ request }) => {
  const res = await request.post('/api/login', { data: { user: 'a', pass: 'b' } });
  expect(res.ok()).toBeTruthy();
});
```

## Screenshots & tracing

```ts
await page.screenshot({ path: 'out.png', fullPage: true });
await locator.screenshot({ path: 'el.png' });
```
Config-driven (see below): `screenshot: 'only-on-failure'`, `video: 'retain-on-failure'`, `trace: 'on-first-retry'`.

```bash
npx playwright show-trace trace.zip     # or drag the .zip into trace.playwright.dev
```

## Test hooks, steps & organization

| Hook | Runs | Use it for |
|---|---|---|
| `test.beforeAll` | once per **worker**, before all tests in the file/describe block | expensive one-time setup shared across tests — seeding a DB, starting a mock server, logging in once and reusing the result |
| `test.afterAll` | once per worker, after all tests in the file/describe block | one-time teardown — closing a shared resource, deleting seeded data |
| `test.beforeEach` | before every test | anything a test needs fresh — navigating to the starting page, resetting mocks. This is the one you'll reach for most, since fresh state keeps tests independent |
| `test.afterEach` | after every test, even on failure | cleanup, plus conditional debug output via `testInfo.status` |

`beforeAll`/`afterAll` run once **per worker**, not once globally — if a file's tests get sharded across workers, the hook runs again on each worker that picks up tests from it. `page`/`context` aren't available inside them by default (those fixtures are per-test); use `browser.newPage()` directly if a `beforeAll` needs one.

```ts
test.beforeAll(async () => { /* once per file/worker */ });
test.afterAll(async () => { /* teardown */ });
test.beforeEach(async ({ page }) => { /* per test */ });
test.afterEach(async ({ page }, testInfo) => {
  if (testInfo.status !== testInfo.expectedStatus) {
    await page.screenshot({ path: `failure-${testInfo.title}.png` });
  }
});
```

**`test.step`** — wraps related actions under a named, collapsible label in the HTML report and trace viewer. Reach for it to keep a long test scannable, or to group a repeated multi-action sequence without pulling it into a full fixture.
```ts
test('checkout flow', async ({ page }) => {
  await test.step('add item to cart', async () => { /* ... */ });
  await test.step('go to checkout', async () => { /* ... */ });
});
```

**Annotations** — these mark *intent*, and each communicates something different to whoever reads the report:

| Call | Meaning | When to use it |
|---|---|---|
| `test.skip(condition, reason)` | don't run, at all | the test doesn't apply here — e.g. a desktop-only feature, skipped on the mobile project |
| `test.fixme()` | don't run, flagged as broken | you know it's currently failing and want that visibly tracked as "needs a fix," not silently excluded |
| `test.fail()` | run it, but expect it to fail | documents a known bug while still exercising the code path; if it starts passing, Playwright flags *that* as the failure — an early signal the bug got fixed |
| `test.slow()` | runs normally, timeout ×3 | the test is legitimately slow (large upload, heavy computation) — avoids raising the timeout globally for everything else |

```ts
test.skip(browserName === 'webkit', 'feature not supported in Safari yet');
test.fixme('checkout total is off by rounding, see JIRA-123', async ({ page }) => { /* ... */ });
test('legacy API responds eventually', async ({ page }) => { test.fail(); /* ... */ });
test('uploads a 2GB file', async ({ page }) => { test.slow(); /* ... */ });
```

**Tags** — filter which tests run without commenting anything out:
```ts
test('smoke check', { tag: '@smoke' }, async ({ page }) => { /* ... */ });
```
```bash
npx playwright test --grep @smoke         # only smoke tests — fast CI gate
npx playwright test --grep-invert @flaky  # skip known-flaky tests
```

**Serial vs. parallel `describe` blocks** — tests in a file run in parallel by default and shouldn't share state. When a group genuinely must run in order — a multi-step wizard where each test builds on the last — opt in explicitly:
```ts
test.describe.serial('checkout wizard', () => {
  test('step 1: add to cart', async ({ page }) => { /* ... */ });
  test('step 2: enter address', async ({ page }) => { /* ... */ }); // same worker, runs after step 1
});
```
Prefer independent tests where you can — in serial mode, one failure skips the rest of the group.

## Fixtures & Page Object Model

```ts
// fixtures.ts
import { test as base } from '@playwright/test';
import { LoginPage } from './pages/LoginPage';

export const test = base.extend<{ loginPage: LoginPage }>({
  loginPage: async ({ page }, use) => {
    const loginPage = new LoginPage(page);
    await page.goto('/login');
    await use(loginPage);
  },
});
export { expect } from '@playwright/test';
```

```ts
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}
  get email() { return this.page.getByLabel('Email'); }
  async login(user: string, pass: string) {
    await this.email.fill(user);
    await this.page.getByLabel('Password').fill(pass);
    await this.page.getByRole('button', { name: 'Sign in' }).click();
  }
}
```

**Reusing authentication** (skip logging in per test):
```ts
// global setup project
setup('authenticate', async ({ page }) => {
  await page.goto('/login');
  // ... perform login
  await page.context().storageState({ path: 'auth.json' });
});
```
```ts
// playwright.config.ts
projects: [
  { name: 'setup', testMatch: /global\.setup\.ts/ },
  { name: 'chromium', use: { ...devices['Desktop Chrome'], storageState: 'auth.json' }, dependencies: ['setup'] },
]
```

## `playwright.config.ts` essentials

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30_000,
  expect: { timeout: 5_000 },
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  reporter: [['html'], ['list']],
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    headless: true,
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
    { name: 'mobile-chrome', use: { ...devices['Pixel 7'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## CLI commands

```bash
npx playwright test                       # run all tests
npx playwright test tests/login.spec.ts   # single file
npx playwright test -g "shows error"      # by title regex
npx playwright test --project=chromium
npx playwright test --headed              # watch the browser
npx playwright test --debug               # Playwright Inspector, step through
npx playwright test --ui                  # interactive UI mode (best for local dev)
npx playwright test --workers=4
npx playwright test --shard=1/3           # split across CI machines
npx playwright codegen playwright.dev     # record actions → generate code
npx playwright show-report                # open last HTML report
```

## GitHub Actions CI (minimal)

```yaml
- uses: actions/setup-node@v4
  with: { node-version: 20 }
- run: npm ci
- run: npx playwright install --with-deps
- run: npx playwright test
- uses: actions/upload-artifact@v4
  if: always()
  with: { name: playwright-report, path: playwright-report/ }
```

## Common gotchas

- `fill()` sets the value directly (fast, no keystroke events); use `pressSequentially()` when the app listens for individual keystrokes (autocomplete, masked inputs).
- Assertions like `toBeVisible()` retry automatically — don't wrap them in your own `waitFor` loops.
- `page.waitForTimeout()` is a smell; prefer waiting for a specific state, response, or assertion.
- Strict mode: a locator matching more than one element throws — narrow with `.filter()`, `.nth()`, or a more specific role/name instead of catching the error.
- Run `npx playwright test --ui` locally before debugging via screenshots/logs — it's usually faster.
- Keep tests independent; don't rely on execution order even with `fullyParallel: false`.

---
*Playwright ~1.6x, TS/JS API. Same concepts apply in Python, Java, and .NET bindings — method names shift to each language's casing convention (e.g. `get_by_role`, `GetByRoleAsync`). Ask if you want a version in one of those.)
