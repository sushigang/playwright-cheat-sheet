# Playwright Interview Questions & Answers

A practical companion to the Playwright cheat sheet, focused on common interview questions for QA Automation, SDET, and Test Automation Engineer roles.

> Use this as an interview review guide. Focus on understanding the reasoning behind each answer rather than memorizing wording.

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Locators](#locators)
3. [Auto-Waiting and Assertions](#auto-waiting-and-assertions)
4. [Fixtures and Hooks](#fixtures-and-hooks)
5. [Test Isolation and Authentication](#test-isolation-and-authentication)
6. [Page Object Model](#page-object-model)
7. [Network and API Testing](#network-and-api-testing)
8. [Tabs, Popups, Frames, Dialogs, and Downloads](#tabs-popups-frames-dialogs-and-downloads)
9. [Parallelism and Retries](#parallelism-and-retries)
10. [Configuration and Projects](#configuration-and-projects)
11. [Debugging and Flaky Tests](#debugging-and-flaky-tests)
12. [JavaScript and TypeScript Fundamentals](#javascript-and-typescript-fundamentals)
13. [Test Design](#test-design)
14. [Practical Coding Questions](#practical-coding-questions)
15. [Senior-Level Questions](#senior-level-questions)
16. [Scenario-Based Interview Questions](#scenario-based-interview-questions)
17. [Additional 2026 Interview Topics](#additional-2026-interview-topics)
18. [Top 15 Questions to Master](#top-15-questions-to-master)

---

# Core Concepts

## 1. What is Playwright?

Playwright is a browser automation framework from Microsoft used primarily for end-to-end testing.

It supports:

- Chromium
- Firefox
- WebKit
- Auto-waiting
- Browser-context isolation
- Network interception and mocking
- Authentication state reuse
- Tracing
- Screenshots and videos
- Parallel execution

---

## 2. What is the difference between Browser, BrowserContext, and Page?

- **Browser**: the actual browser process.
- **BrowserContext**: an isolated browser session, similar to an incognito profile.
- **Page**: a browser tab inside a context.

```ts
const browser = await chromium.launch();

const context = await browser.newContext();

const page = await context.newPage();
```

A single browser can contain multiple contexts, and each context can contain multiple pages.

### Why use BrowserContext instead of launching a browser for every test?

Creating a context is much cheaper than starting a new browser process while still providing strong test isolation.

---

# Locators

## 3. What is a Locator?

A Locator represents a way to find one or more elements on the page.

```ts
const loginButton = page.getByRole('button', {
  name: 'Login'
});

await loginButton.click();
```

Locators work well with Playwright's retry and auto-waiting behavior because the target is resolved when an action is performed.

---

## 4. What locator strategy should you prefer?

Prefer user-facing locators when possible:

```ts
getByRole()
getByLabel()
getByText()
getByPlaceholder()
getByAltText()
getByTitle()
getByTestId()
```

Example:

```ts
await page
  .getByRole('button', { name: 'Submit' })
  .click();
```

`getByRole()` is often a strong first choice because it reflects how users and assistive technologies identify controls.

---

## 5. Why avoid brittle CSS or XPath selectors?

Selectors tied closely to DOM structure break easily when markup changes.

Brittle:

```ts
page.locator('div:nth-child(3) > div > button');
```

More resilient:

```ts
page.getByRole('button', { name: 'Submit' });
```

Prefer selectors that represent user intent rather than implementation details.

---

## 6. `getByRole()` vs `getByText()`?

Use `getByRole()` for semantic controls:

```ts
page.getByRole('button', { name: 'Save' });
```

Use `getByText()` when the visible text itself is the target:

```ts
page.getByText('Order completed');
```

---

## 7. What is locator strictness?

Actions that require one element generally expect the locator to resolve to exactly one target.

Potentially ambiguous:

```ts
await page.getByRole('button').click();
```

Better:

```ts
await page
  .getByRole('button', { name: 'Submit' })
  .click();
```

Strictness helps catch ambiguous selectors early.

---

## 8. How do you locate an element inside another element?

Use locator chaining or filtering.

```ts
const product = page
  .getByRole('listitem')
  .filter({ hasText: 'iPhone' });

await product
  .getByRole('button', { name: 'Add to cart' })
  .click();
```

This is often more maintainable than one large CSS selector.

---

# Auto-Waiting and Assertions

## 9. What is auto-waiting?

Before performing actions such as `click()`, Playwright automatically waits for required actionability conditions.

For a typical click, Playwright checks conditions such as:

- Element resolves uniquely
- Element is visible
- Element is stable
- Element receives events
- Element is enabled

If the conditions are not satisfied before the timeout, the action fails.

---

## 10. Should you use `waitForTimeout()`?

Usually no.

Avoid:

```ts
await page.waitForTimeout(5000);
```

Prefer waiting for the actual condition:

```ts
await expect(
  page.getByText('Success')
).toBeVisible();
```

Or wait for a specific response:

```ts
await page.waitForResponse(
  response =>
    response.url().includes('/orders') &&
    response.status() === 200
);
```

Hard waits make tests slower and often more flaky.

---

## 11. Auto-waiting vs explicit waiting?

Auto-waiting happens around Playwright actions automatically:

```ts
await button.click();
```

Explicit waiting is useful when synchronizing with a specific application event:

```ts
await page.waitForURL('/dashboard');
```

or:

```ts
const responsePromise = page.waitForResponse(
  response => response.url().includes('/orders')
);
```

---

## 12. What are web-first assertions?

Playwright's web assertions automatically retry until the condition becomes true or the assertion timeout expires.

```ts
await expect(
  page.getByTestId('status')
).toHaveText('Submitted');
```

This is better for dynamic UIs than checking a value once.

---

## 13. What is wrong with this assertion?

```ts
expect(
  await locator.textContent()
).toBe('Success');
```

It reads the text once and immediately compares it.

Prefer:

```ts
await expect(locator).toHaveText('Success');
```

The Playwright assertion retries until the expected state is reached.

---

## 14. `toHaveText()` vs `toContainText()`?

Exact expected text:

```ts
await expect(locator)
  .toHaveText('Welcome John');
```

Contains expected text:

```ts
await expect(locator)
  .toContainText('Welcome');
```

---

## 15. What is a soft assertion?

A soft assertion records a failure but allows the test to continue.

```ts
await expect.soft(
  page.getByTestId('status')
).toHaveText('Success');

await expect.soft(
  page.getByTestId('total')
).toHaveText('$100');
```

The test still fails overall if a soft assertion failed.

---

# Fixtures and Hooks

## 16. What are fixtures?

Fixtures prepare dependencies and test environments.

Common built-in fixtures include:

```ts
page
context
browser
request
```

Example:

```ts
test('checkout', async ({ page }) => {
  // page is provided as a fixture
});
```

Fixtures can also manage setup and teardown.

---

## 17. Why use fixtures instead of only `beforeEach()`?

Fixtures provide:

- Reusability across files
- Automatic setup and teardown
- Dependency injection
- Test and worker scopes
- Lazy initialization
- Better framework scalability

---

## 18. How do you create a custom fixture?

```ts
import { test as base } from '@playwright/test';

export const test = base.extend({
  loggedInPage: async ({ page }, use) => {

    await page.goto('/login');

    // login steps

    await use(page);

    // optional cleanup
  }
});
```

Code before `use()` performs setup.

Code after `use()` performs teardown.

---

## 19. Test-scoped vs worker-scoped fixture?

### Test-scoped

Created separately for every test.

### Worker-scoped

Created once per worker and reused by tests executed in that worker.

Worker-scoped fixtures are useful for expensive setup such as creating a test user.

---

## 20. What hooks are available?

```ts
test.beforeAll()
test.afterAll()

test.beforeEach()
test.afterEach()
```

- `beforeEach` and `afterEach` run around every applicable test.
- `beforeAll` and `afterAll` run around a group or file lifecycle within a worker.

---

# Test Isolation and Authentication

## 21. How does Playwright provide test isolation?

Each test normally receives a fresh BrowserContext.

That isolates browser state such as:

- Cookies
- Local storage
- Session storage
- Browser session state

This prevents one test from contaminating another.

---

## 22. Why should tests not depend on one another?

Tests should ideally be independently executable.

Bad design:

```text
Test 1 creates customer
Test 2 edits customer
Test 3 deletes customer
```

If Test 1 fails, Tests 2 and 3 become unreliable.

A better approach is to create the required state independently for each test, often through fixtures or APIs.

---

## 23. How do you avoid logging in before every test?

Authenticate once, save the browser state, and reuse it.

```ts
await page.context().storageState({
  path: 'playwright/.auth/user.json'
});
```

Configuration:

```ts
use: {
  storageState: 'playwright/.auth/user.json'
}
```

---

## 24. What is `storageState`?

`storageState` represents browser storage information that can be loaded into a new BrowserContext.

It is commonly used to reuse authenticated sessions across tests.

---

## 25. Is it always safe for parallel tests to share one authenticated account?

No.

If tests modify server-side state for the same account, they can interfere with each other.

For stateful tests, consider:

- Separate accounts per worker
- Separate accounts per test
- Unique test data
- API-based setup and cleanup

---

# Page Object Model

## 26. What is Page Object Model?

Page Object Model encapsulates page-specific locators and actions inside classes.

```ts
class LoginPage {
  constructor(private page: Page) {}

  username = this.page.getByLabel('Username');

  password = this.page.getByLabel('Password');

  loginButton = this.page.getByRole('button', {
    name: 'Login'
  });

  async login(
    username: string,
    password: string
  ) {
    await this.username.fill(username);
    await this.password.fill(password);
    await this.loginButton.click();
  }
}
```

Usage:

```ts
const login = new LoginPage(page);

await login.login(
  'john',
  'password123'
);
```

---

## 27. What are disadvantages of POM?

POM can become harmful when page classes grow too large or contain unnecessary abstraction.

Good page objects should be:

- Focused
- Domain-oriented
- Reusable
- Easy to understand

For component-heavy applications, component objects may be better than one giant page class.

---

# Network and API Testing

## 28. How do you intercept or mock an API request?

```ts
await page.route(
  '**/api/users',
  async route => {
    await route.fulfill({
      status: 200,
      json: [
        { id: 1, name: 'John' }
      ]
    });
  }
);
```

Playwright can inspect, modify, abort, or mock network traffic.

---

## 29. Why mock APIs?

API mocking is useful for testing UI behavior independently from backend availability.

Examples:

- API returns `500`
- API returns an empty list
- API returns malformed data
- API is slow
- Specific edge-case data is needed

Do not mock everything. Critical end-to-end flows should still validate real integrations.

---

## 30. How do you wait for an API response?

Create the wait before triggering the action.

```ts
const responsePromise =
  page.waitForResponse(
    response =>
      response.url().includes('/orders') &&
      response.status() === 200
  );

await page
  .getByRole('button', { name: 'Submit' })
  .click();

const response = await responsePromise;
```

---

## 31. How would you test a `500` response?

```ts
await page.route(
  '**/api/orders',
  route =>
    route.fulfill({
      status: 500,
      body: 'Server error'
    })
);

await page.goto('/orders');

await expect(
  page.getByText('Something went wrong')
).toBeVisible();
```

The purpose is to verify frontend error handling.

---

# Tabs, Popups, Frames, Dialogs, and Downloads

## 32. How do you handle a new tab?

```ts
const pagePromise =
  context.waitForEvent('page');

await page
  .getByText('Open new tab')
  .click();

const newPage =
  await pagePromise;

await newPage.waitForLoadState();
```

Start listening before the action that opens the new page.

---

## 33. How do you handle a popup?

```ts
const popupPromise =
  page.waitForEvent('popup');

await page
  .getByText('Open popup')
  .click();

const popup =
  await popupPromise;
```

A popup is another `Page` object.

---

## 34. How do you handle an iframe?

```ts
const frame =
  page.frameLocator('#payment-frame');

await frame
  .getByLabel('Card number')
  .fill('4111111111111111');
```

Use `frameLocator()` to interact with elements inside an iframe.

---

## 35. How do you handle an alert or browser dialog?

```ts
page.on(
  'dialog',
  async dialog => {
    console.log(dialog.message());
    await dialog.accept();
  }
);

await page
  .getByRole('button')
  .click();
```

If a dialog handler is registered, make sure it accepts or dismisses the dialog.

---

## 36. How do you test a download?

```ts
const downloadPromise =
  page.waitForEvent('download');

await page
  .getByText('Download')
  .click();

const download =
  await downloadPromise;

console.log(
  download.suggestedFilename()
);
```

Start waiting before triggering the download.

---

# Parallelism and Retries

## 37. Does Playwright run tests in parallel?

Yes.

Playwright uses worker processes.

By default:

- Test files can run in parallel across workers.
- Tests inside one file normally run in declaration order.
- Full parallelism can be enabled explicitly.

---

## 38. What is a Playwright worker?

A worker is an operating-system process used by Playwright Test to execute tests.

The `workers` configuration controls maximum concurrency.

---

## 39. Why might tests fail only in parallel?

Common causes include:

- Same authenticated account
- Same database record
- Shared test data
- Shared files
- External service collisions
- Test-order dependencies
- Global mutable state

Example:

```text
Test A updates customer 123
Test B deletes customer 123
```

Both may pass alone but fail unpredictably together.

---

## 40. How do you prevent parallel test collisions?

Generate isolated data.

Example:

```ts
const username =
  `user-${testInfo.workerIndex}`;
```

Or:

```ts
const email =
  `user-${crypto.randomUUID()}@example.com`;
```

Other solutions include:

- Per-worker accounts
- Per-test records
- API setup and cleanup
- Worker-scoped fixtures

---

## 41. What are retries?

Retries rerun failed tests up to a configured number of times.

```ts
export default defineConfig({
  retries: 2
});
```

Retries can help identify flaky tests, but they should not hide instability.

---

## 42. Would you fix a flaky test by increasing retries?

No.

First identify the cause.

Typical causes include:

- Hard waits
- Weak selectors
- Race conditions
- Shared test data
- Network dependencies
- Missing `await`
- Animations
- Overlays

Retries are a safety net, not a root-cause fix.

---

# Configuration and Projects

## 43. What belongs in `playwright.config.ts`?

Typical settings include:

```ts
export default defineConfig({

  testDir: './tests',

  timeout: 30_000,

  retries: 2,

  workers: 4,

  use: {
    baseURL: 'https://example.com',

    trace: 'on-first-retry',

    screenshot: 'only-on-failure'
  }
});
```

Other common configuration includes:

- Projects
- Reporters
- Authentication state
- Devices
- Browser settings
- CI-specific behavior

---

## 44. What are Playwright projects?

Projects allow the same test suite to run under different configurations.

```ts
projects: [
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome']
    }
  },
  {
    name: 'firefox',
    use: {
      ...devices['Desktop Firefox']
    }
  },
  {
    name: 'webkit',
    use: {
      ...devices['Desktop Safari']
    }
  }
]
```

Projects can also represent:

- Different environments
- Mobile devices
- Authentication roles
- Test groups
- Browser combinations

---

# Debugging and Flaky Tests

## 45. A test fails only in CI. What do you investigate?

Do not immediately increase the timeout.

Check:

1. Playwright trace
2. Screenshots
3. Video if enabled
4. Network requests
5. Browser console errors
6. Locator timing
7. Environment differences
8. Test data collisions
9. CI concurrency
10. Resource constraints

Try to reproduce the failure under CI-like conditions.

---

## 46. What is Trace Viewer?

Trace Viewer helps inspect a test execution after failure.

A trace can provide information about:

- Actions
- DOM snapshots
- Network activity
- Console messages
- Screenshots
- Timing

Typical configuration:

```ts
use: {
  trace: 'on-first-retry'
}
```

---

## 47. What commonly causes flaky Playwright tests?

Common causes:

- `waitForTimeout()`
- Fragile selectors
- Shared test data
- Test dependencies
- Race conditions
- Missing `await`
- Unstable APIs
- Animations
- Incorrect synchronization
- Environment differences

Playwright reduces timing problems, but poor test or application design can still cause flakiness.

---

## 48. Improve this flaky test

Problematic:

```ts
await page.click('#submit');

await page.waitForTimeout(3000);

expect(
  await page.locator('.message')
    .textContent()
).toBe('Success');
```

Improved:

```ts
await page
  .getByRole('button', { name: 'Submit' })
  .click();

await expect(
  page.getByText('Success')
).toBeVisible();
```

Improvements:

- Better locator
- No hard wait
- Retrying assertion

---

# JavaScript and TypeScript Fundamentals

## 49. Why is `async/await` important in Playwright?

Most browser operations are asynchronous.

```ts
await page.goto('/');

await page
  .getByRole('button')
  .click();
```

Without `await`, later code may execute before the previous operation finishes.

---

## 50. What is a Promise?

A Promise represents the eventual result of an asynchronous operation.

A Promise can be:

```text
pending
fulfilled
rejected
```

`await` pauses the current async function until the Promise settles.

---

## 51. What is wrong with this code?

```ts
page.getByRole('button').click();

expect(page.url())
  .toContain('/dashboard');
```

The click is asynchronous and is not awaited.

Better:

```ts
await page
  .getByRole('button')
  .click();

await expect(page)
  .toHaveURL(/dashboard/);
```

---

## 52. When would you use `Promise.all()`?

Use it when independent asynchronous operations should happen concurrently.

For Playwright browser events, a common pattern is to create the event promise before the triggering action:

```ts
const popupPromise =
  page.waitForEvent('popup');

await page
  .getByText('Open')
  .click();

const popup =
  await popupPromise;
```

The important concept is avoiding race conditions.

---

# Test Design

## 53. What tests would you automate first?

Prioritize based on:

- Business criticality
- Regression risk
- Execution frequency
- Manual effort
- Stability
- Repeatability
- Customer impact

Common candidates include:

- Login
- Checkout
- Account management
- Core business workflows

---

## 54. What tests would you avoid automating?

Examples:

- One-time tests
- Very unstable experimental features
- Low-value scenarios
- Tests whose maintenance cost exceeds their value
- Tests requiring subjective human judgment

---

## 55. What is the test pyramid?

Conceptually:

```text
        E2E
       /   \
 Integration
 /         \
Unit Tests
```

The idea is to have:

- Many fast unit tests
- Fewer integration tests
- A smaller number of slower end-to-end tests

Playwright is commonly used at the browser/E2E layer and can also perform API testing.

---

# Practical Coding Questions

## 56. Write a login test

```ts
test('user can login', async ({ page }) => {

  await page.goto('/login');

  await page
    .getByLabel('Email')
    .fill('john@example.com');

  await page
    .getByLabel('Password')
    .fill('secret');

  await page
    .getByRole(
      'button',
      { name: 'Login' }
    )
    .click();

  await expect(page)
    .toHaveURL(/dashboard/);

  await expect(
    page.getByRole(
      'heading',
      { name: 'Dashboard' }
    )
  ).toBeVisible();
});
```

---

## 57. How would you test search results?

```ts
test('search products', async ({ page }) => {

  await page.goto('/');

  await page
    .getByPlaceholder('Search')
    .fill('iPhone');

  await page
    .getByRole(
      'button',
      { name: 'Search' }
    )
    .click();

  const products =
    page.getByTestId('product');

  await expect(products)
    .toHaveCount(5);
});
```

---

## 58. How would you test a loading spinner?

Avoid:

```ts
await page.waitForTimeout(5000);
```

Prefer:

```ts
const spinner =
  page.getByTestId('spinner');

await expect(spinner)
  .toBeVisible();

await expect(spinner)
  .toBeHidden();

await expect(
  page.getByTestId('results')
).toBeVisible();
```

Wait for state changes, not arbitrary time.

---

# Senior-Level Questions

## 59. How would you design a Playwright framework from scratch?

Example structure:

```text
tests/
  login/
  checkout/
  account/

pages/
  LoginPage.ts
  CheckoutPage.ts

components/
  Header.ts
  Modal.ts

fixtures/
  testFixtures.ts

utils/
  api.ts
  dataFactory.ts

playwright.config.ts

playwright/.auth/
```

Key design decisions:

- Locator conventions
- Fixture strategy
- Authentication strategy
- Test-data strategy
- API helpers
- Page/component object strategy
- Environment configuration
- Reporting
- Tracing
- CI execution
- Parallelization
- Cleanup

---

## 60. UI vs API: when would you use each?

Use the UI when validating user behavior.

Use APIs for setup or cleanup when the setup itself is not under test.

Example:

```text
API -> create customer

UI -> verify customer appears

UI -> update customer

API -> cleanup customer
```

This keeps tests faster and focused on the behavior being validated.

---

## 61. How would you reduce execution time for 2,000 Playwright tests?

Investigate:

- More parallel workers
- CI sharding
- Repeated login
- Unnecessary UI setup
- API-based test setup
- Hard waits
- Duplicate tests
- Browser/project coverage
- Slow backend dependencies
- Test-data bottlenecks

Increase concurrency only after ensuring tests are properly isolated.

---

## 62. Why Playwright over Selenium?

A balanced answer is best.

Playwright provides built-in features such as:

- Auto-waiting
- Browser-context isolation
- Chromium, Firefox, and WebKit support
- Network interception
- Tracing
- Native fixtures
- Test runner
- Multi-page handling

Selenium has:

- A larger ecosystem
- Broad language support
- Long-standing enterprise adoption
- Extensive browser/grid tooling

The best choice depends on project and organization requirements.

---

## 63. What is the difference between `fill()` and keyboard input?

Use `fill()` when you simply want to set the value of an input:

```ts
await input.fill('John');
```

Use keyboard actions when the application specifically depends on keyboard behavior:

```ts
await input.press('Enter');
```

Do not simulate keystrokes unless the test actually needs to validate keyboard interaction.

---

## 64. When would you use `force: true`?

Rarely.

```ts
await button.click({
  force: true
});
```

`force` bypasses some normal actionability checks.

If an element cannot normally receive a click because an overlay covers it, forcing the click may hide a real application bug.

Investigate why the element is not actionable before using `force`.

---

## 65. What is your approach when a test fails?

A strong debugging process is:

1. Determine whether the failure is caused by the application, test code, environment, test data, or timing.
2. Inspect the error message and stack trace.
3. Open the Playwright trace.
4. Check DOM snapshots and screenshots.
5. Inspect browser console errors.
6. Inspect relevant network requests.
7. Check locator quality.
8. Check shared data and parallel execution.
9. Fix the root cause.
10. Avoid hiding failures with arbitrary waits or retries.

---

# Scenario-Based Interview Questions

These questions test how you apply Playwright in real projects. A strong answer should explain the likely cause, how you would investigate it, the preferred solution, and what you would avoid.

## 66. A test passes locally but fails in CI. What do you do?

Use a structured debugging process:

1. Open the trace from the CI run.
2. Check screenshots, video, console errors, and network activity.
3. Compare Node, Playwright, browser, locale, timezone, and environment configuration.
4. Look for missing synchronization or race conditions.
5. Check whether parallel tests share users or test data.
6. Reproduce locally with CI-like settings if possible.

Do not immediately increase timeouts or retries. First determine whether the failure comes from the application, test code, environment, timing, or shared state.

---

## 67. A button click is flaky. How would you investigate it?

Check:

- Whether the locator uniquely identifies the intended element
- Whether an overlay, animation, or loading state blocks the button
- Whether the element is being re-rendered
- Whether the test is missing an `await`
- Whether an upstream API call has completed
- Whether the test uses a hard wait instead of waiting for state

Prefer Playwright's actionability checks and semantic locators. Avoid solving the issue with `force: true` until you understand why the normal click fails.

---

## 68. Two tests pass individually but fail when run together. What is the likely cause?

The first suspicion should be shared state.

Examples:

- Same user account
- Same database record
- Same order or customer
- Same uploaded file
- Global mutable variables
- Test-order dependency

Fix the problem by isolating data per test or worker.

```ts
const email =
  `user-${testInfo.workerIndex}-${crypto.randomUUID()}@example.com`;
```

---

## 69. You need to test an error state that is difficult to reproduce from the backend. What do you do?

Mock the backend response.

```ts
await page.route('**/api/orders', route =>
  route.fulfill({
    status: 500,
    json: { error: 'Internal server error' }
  })
);

await page.goto('/orders');

await expect(
  page.getByText('Something went wrong')
).toBeVisible();
```

This makes the test deterministic and lets you validate rare error conditions.

---

## 70. The page can take anywhere from 1 to 10 seconds to load results. How do you synchronize the test?

Do not use:

```ts
await page.waitForTimeout(10000);
```

Wait for the actual state:

```ts
await expect(
  page.getByTestId('results')
).toBeVisible();
```

Or synchronize with the relevant API response:

```ts
const responsePromise =
  page.waitForResponse(res =>
    res.url().includes('/search') &&
    res.ok()
  );

await page
  .getByRole('button', { name: 'Search' })
  .click();

await responsePromise;
```

---

## 71. Your selectors keep breaking after frontend refactors. What would you change?

Create a locator strategy.

Prefer, roughly:

1. `getByRole()`
2. `getByLabel()`
3. Other user-facing locators
4. `getByTestId()` where semantic locators are insufficient
5. Stable CSS as a last resort

Avoid selectors tied to generated IDs, styling classes, or deep DOM structure.

---

## 72. You have hundreds of tests and UI login takes several seconds per test. How would you optimize it?

Reuse authenticated state with `storageState`.

For suites where tests mutate account-specific server data, do not let every parallel test share the same account. Use multiple users, such as one account per worker.

Another option is to authenticate through an API and inject the resulting state when appropriate.

---

## 73. A test sometimes misses a newly opened tab. Why?

The listener may be registered after the event occurs.

Correct pattern:

```ts
const newPagePromise =
  context.waitForEvent('page');

await page
  .getByRole('link', { name: 'Open report' })
  .click();

const newPage =
  await newPagePromise;
```

Register the wait before triggering the action.

---

## 74. A framework uses `force: true` for most clicks. Is that a good design?

No.

`force: true` bypasses normal actionability behavior and can hide real issues such as:

- Overlays
- Disabled elements
- Incorrect application state
- Wrong locators
- UI defects

Use it only when bypassing normal user interaction is intentional and justified.

---

## 75. Text appears shortly after an API response and the test fails intermittently. What would you change?

Use a retrying Playwright assertion.

Avoid:

```ts
expect(
  await locator.textContent()
).toBe('Success');
```

Prefer:

```ts
await expect(locator)
  .toHaveText('Success');
```

The assertion waits for the UI to reach the expected state.

---

## 76. You need a customer to exist before testing the customer-details page. Would you create the customer through the UI?

Usually no, unless customer creation itself is part of the behavior under test.

A better pattern is:

```text
API -> create customer
UI  -> validate customer details
API -> cleanup
```

This reduces runtime and isolates the UI behavior you actually want to test.

---

## 77. A checkout test takes several minutes. How would you reduce its runtime?

Investigate:

- Repeated login
- Hard waits
- UI-based test-data creation
- Duplicate navigation
- Slow external dependencies
- Unnecessary cross-browser coverage for every test
- Safe opportunities for parallelism

Use APIs for setup, reuse authentication where safe, and keep the E2E path focused on business-critical UI behavior.

---

## 78. A request is triggered after clicking Submit. How do you verify the request completes successfully?

Start waiting before clicking.

```ts
const responsePromise =
  page.waitForResponse(res =>
    res.url().includes('/orders') &&
    res.status() === 201
  );

await page
  .getByRole('button', { name: 'Submit' })
  .click();

const response =
  await responsePromise;

expect(response.ok()).toBeTruthy();
```

---

## 79. Your team wants every interaction inside a Page Object. Would you agree?

Not automatically.

Use Page Objects for reusable page or component behavior. Use fixtures for state and dependencies. Simple test-specific interactions do not always need another abstraction layer.

Overusing POM can create large classes that become harder to maintain than the tests themselves.

---

## 80. An optional modal appears only for some users. How would you handle it?

First determine whether the modal is legitimately optional.

If it is optional, handle it based on observable state without introducing arbitrary sleeps.

If it is expected under a defined condition, assert the condition and verify the modal explicitly rather than silently ignoring it.

Avoid broad exception handling that hides unexpected UI changes.

---

## 81. A test passes headed but fails headless. What would you investigate?

Check:

- Timing and race conditions
- Viewport differences
- Responsive layout
- Animations
- Browser-specific rendering
- Resource loading
- Fonts or assets
- Environment variables
- Hidden assumptions in the test

The goal is to find why behavior differs, not to simply force headed mode in CI.

---

## 82. A test works in Chromium but fails in Firefox. What do you do?

Treat the failure as useful information.

Determine whether it is:

- A real browser compatibility issue
- A browser-specific application bug
- A test relying on Chromium-only behavior
- A locator/timing issue

Use browser-specific traces, network logs, and console output to compare behavior.

---

## 83. Your CI suite uses many retries and usually becomes green. Is that acceptable?

Not as a long-term strategy.

Retries can absorb genuine transient failures, but frequent retries indicate that the suite is unreliable.

Track tests that require retries, identify their root causes, and fix or temporarily quarantine known flaky tests rather than silently accepting them.

---

## 84. A developer suggests adding a five-second sleep to fix a test. What would you do?

Wait for the real condition instead.

For example:

```ts
await expect(
  page.getByRole('heading', { name: 'Dashboard' })
).toBeVisible();
```

or wait for the relevant API response.

A fixed delay is both slower than necessary when the app is fast and still insufficient when the app takes longer.

---

## 85. How would you test that a regular user cannot access an admin page?

Use a non-admin authenticated state and verify the expected authorization behavior.

Possible expected results include:

- Redirect to login
- Redirect to a safe page
- `403` response
- Access-denied message
- Admin controls not being present

Test authorization behavior, not just whether a menu link is hidden.

---

## 86. How would you test multiple user roles?

Use separate authentication states or fixtures.

For example:

```text
admin.json
editor.json
viewer.json
customer.json
```

For real-time multi-user workflows, create separate BrowserContexts inside the same test so each role has isolated cookies and storage.

---

## 87. A test deletes shared environment data. What would you change?

Do not let automated tests modify uncontrolled shared records.

Prefer:

- Dedicated test environments
- Unique generated records
- Per-test or per-worker data
- Isolated tenants/accounts
- Explicit cleanup
- API-based teardown

Tests should own the data they mutate.

---

## 88. `selectOption()` does not work on a dropdown. Why?

The UI may not be a native `<select>` element.

Many modern applications implement custom dropdowns using regular DOM elements.

For a custom component, interact with it like a user:

```ts
await page
  .getByRole('combobox', { name: 'Country' })
  .click();

await page
  .getByRole('option', { name: 'Canada' })
  .click();
```

Use `selectOption()` only for native select controls.

---

## 89. How would you test a file upload?

For a file input:

```ts
await page
  .getByLabel('Upload document')
  .setInputFiles('fixtures/report.pdf');
```

Then verify the outcome:

```ts
await expect(
  page.getByText('report.pdf')
).toBeVisible();
```

Do not stop at setting the file. Verify what the user or backend is expected to observe.

---

## 90. How would you handle CAPTCHA in automated tests?

Do not automate solving production CAPTCHA as part of normal E2E testing.

In a controlled test environment, use one of these strategies:

- Disable CAPTCHA
- Use provider-supported test keys
- Add a test-only bypass
- Validate the integration separately

The test should validate your application logic without fighting an anti-automation mechanism.

---

## 91. A workflow depends on receiving an email. How would you automate it?

Prefer a controlled test mailbox or email API.

A robust design might be:

```text
UI -> request password reset
API/test mailbox -> retrieve message
Extract reset URL/token
UI -> complete reset
```

Avoid depending on a personal inbox or unpredictable external mail delivery.

---

## 92. A page makes dozens of network requests. How do you wait for the correct one?

Use a precise predicate.

```ts
const response =
  await page.waitForResponse(res =>
    res.url().includes('/api/orders/123') &&
    res.request().method() === 'GET' &&
    res.status() === 200
  );
```

Do not wait for an arbitrary request or rely on broad timing assumptions.

---

## 93. Would you use `networkidle` after every navigation?

No.

Modern applications may use polling, analytics, WebSockets, or long-lived connections.

Prefer waiting for a business-relevant UI state, URL, or specific response.

---

## 94. A locator matches two elements. Would you solve it with `.first()`?

Only if "the first one" is actually part of the requirement.

Usually a better solution is to narrow the locator:

```ts
const dialog =
  page.getByRole('dialog', {
    name: 'Delete account'
  });

await dialog
  .getByRole('button', { name: 'Confirm' })
  .click();
```

Using `.first()` without understanding why multiple elements match can hide a locator problem.

---

## 95. How would you handle a dynamic table?

Locate the row by meaningful content and scope actions to that row.

```ts
const row = page
  .getByRole('row')
  .filter({ hasText: 'INV-1024' });

await expect(row)
  .toContainText('Pending');

await row
  .getByRole('button', { name: 'Pay' })
  .click();
```

Avoid relying on fixed row indexes because sorting, filtering, and server data can change the order.

---

## 96. How do you handle an element that is currently hidden?

Do not click a hidden element just because you can locate it.

Wait for or trigger the state in which a real user can interact with it.

```ts
const button =
  page.getByRole('button', { name: 'Continue' });

await expect(button)
  .toBeVisible();

await button.click();
```

If it should remain hidden, assert that instead.

---

# Additional 2026 Interview Topics

These questions cover useful areas that are often missing from short cheat sheets.

## 97. How do you initialize a new Playwright Test project?

A common starting point is:

```bash
npm init playwright@latest
```

For an existing project, install the test runner and browser binaries:

```bash
npm install -D @playwright/test
npx playwright install
```

---

## 98. What is the difference between headed and headless mode?

**Headless mode** runs without a visible browser window and is commonly used in CI.

**Headed mode** displays the browser and is useful when debugging locally.

```bash
npx playwright test --headed
```

---

## 99. What is the difference between a selector and a Locator?

A selector is the query used to identify an element.

A `Locator` is Playwright's higher-level abstraction around element lookup. It resolves lazily, re-queries the current DOM, works with actionability checks, and integrates with retrying assertions.

That is why Locators are preferred over manually storing element handles.

---

## 100. `Locator` vs `ElementHandle`: what is the difference?

An `ElementHandle` points to a particular DOM node.

A `Locator` describes how to find the element and resolves it when an action or assertion occurs.

If a React component re-renders and replaces the node, a Locator can resolve the new node, while a previously captured handle may no longer represent the current UI.

Prefer Locators for normal test automation.

---

## 101. How do you work with multiple matching elements?

Useful Locator APIs include:

```ts
const items =
  page.getByRole('listitem');

const count =
  await items.count();

await items.first().click();
await items.last().click();
await items.nth(2).click();
```

Use positional selection only when position is meaningful. Otherwise narrow the locator by content or parent context.

---

## 102. What is `APIRequestContext`?

`APIRequestContext` is Playwright's built-in HTTP client.

It can call APIs without launching a browser.

```ts
test('health endpoint', async ({ request }) => {
  const response =
    await request.get('/api/health');

  expect(response.status())
    .toBe(200);
});
```

It is useful for API tests, authentication, setup, cleanup, and hybrid UI/API scenarios.

---

## 103. How do you combine API and UI testing effectively?

Use the API for fast setup and the UI for the behavior being validated.

```ts
test('order appears in history',
  async ({ request, page }) => {

  const response =
    await request.post('/api/orders', {
      data: {
        sku: 'ABC',
        quantity: 1
      }
    });

  const order =
    await response.json();

  await page.goto('/orders');

  await expect(
    page.getByText(order.id)
  ).toBeVisible();
});
```

This avoids spending UI time on setup that is unrelated to the test's purpose.

---

## 104. How can you modify or abort network requests?

Use `page.route()`.

Abort:

```ts
await page.route(
  '**/analytics/**',
  route => route.abort()
);
```

Modify headers:

```ts
await page.route(
  '**/api/**',
  route => {
    const headers = {
      ...route.request().headers(),
      'x-test-mode': 'true'
    };

    return route.continue({ headers });
  }
);
```

This is useful for dependency control, error scenarios, and testing unusual network conditions.

---

## 105. How do you generate and open the HTML report?

Run tests:

```bash
npx playwright test
```

Open the most recent HTML report:

```bash
npx playwright show-report
```

Playwright can also produce reporters such as JUnit output for CI integration.

---

## 106. How do you configure screenshots, video, and traces?

Example:

```ts
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
  trace: 'on-first-retry'
}
```

This keeps successful CI runs lighter while preserving useful debugging evidence for failures.

---

## 107. Inspector vs UI Mode vs `page.pause()`?

### Inspector / debug mode

```bash
npx playwright test --debug
```

Useful for stepping through a specific test and inspecting locators.

### UI Mode

```bash
npx playwright test --ui
```

Useful for visually browsing, running, and debugging the test suite.

### `page.pause()`

```ts
await page.pause();
```

Stops a running test at a specific point and opens interactive debugging tools.

---

## 108. How do you run Playwright in CI?

A typical CI job:

1. Check out the code.
2. Install dependencies with `npm ci`.
3. Install Playwright browsers and required OS dependencies.
4. Run the tests.
5. Upload reports/traces as artifacts.

Example command:

```bash
npx playwright install --with-deps
npx playwright test
```

Configure CI retries modestly and preserve failure artifacts for debugging.

---

## 109. What is test sharding?

Sharding splits a suite across multiple machines or CI jobs.

Example:

```bash
npx playwright test --shard=1/4
npx playwright test --shard=2/4
npx playwright test --shard=3/4
npx playwright test --shard=4/4
```

Use sharding when one machine is already using concurrency effectively and the suite still takes too long.

---

## 110. What is the difference between workers and shards?

**Workers** parallelize tests on one machine.

**Shards** split the suite across multiple independent jobs or machines.

A large CI suite may use both.

---

## 111. How would you deal with unusually slow tests?

Do not raise the timeout for the entire suite just because a few tests are slow.

Options include:

```ts
test('large export', async ({ page }) => {
  test.slow();

  // ...
});
```

or a targeted timeout:

```ts
test.setTimeout(120_000);
```

You can also tag slow tests and run them separately from the fast PR suite.

---

## 112. When do retries help and when do they hide bugs?

Retries can help with genuine transient issues such as a temporary external-service failure.

They become harmful when they normalize:

- Race conditions
- Shared-state collisions
- Weak locators
- Missing waits
- Application defects

A test that consistently needs retries should be investigated.

---

## 113. How do you test multiple browser configurations?

Use projects.

```ts
projects: [
  {
    name: 'chromium',
    use: {
      ...devices['Desktop Chrome']
    }
  },
  {
    name: 'firefox',
    use: {
      ...devices['Desktop Firefox']
    }
  },
  {
    name: 'webkit',
    use: {
      ...devices['Desktop Safari']
    }
  }
]
```

Projects can also model roles, environments, devices, and test suites.

---

## 114. How do you set up a multi-user workflow?

Use separate BrowserContexts.

```ts
const customerContext =
  await browser.newContext({
    storageState: 'customer.json'
  });

const adminContext =
  await browser.newContext({
    storageState: 'admin.json'
  });

const customer =
  await customerContext.newPage();

const admin =
  await adminContext.newPage();
```

Each user has independent cookies and browser storage while interacting with the same application backend.

---

## 115. How can token-based authentication speed up tests?

If the application supports API authentication, obtain the token without using the login UI and initialize the browser state appropriately.

This can reduce repeated UI setup, but the implementation depends on how the application stores and transmits authentication.

Keep at least dedicated login-flow tests so the actual login UI remains covered.

---

## 116. What is Playwright MCP?

Playwright MCP is an integration that exposes browser-control capabilities through the Model Context Protocol so compatible AI agents can interact with a Playwright-controlled browser.

For interview purposes, understand the high-level distinction:

- Traditional tests are deterministic code authored by engineers.
- MCP can let AI tooling inspect and interact with browser state through structured tools.
- Generated or AI-assisted tests still need review for correctness, stability, and maintainability.

---

## 117. What Playwright CLI commands should you know?

Common commands:

```bash
npx playwright test
npx playwright test login.spec.ts
npx playwright test --headed
npx playwright test --debug
npx playwright test --ui
npx playwright codegen https://example.com
npx playwright show-report
npx playwright install
```

You do not need to memorize every flag, but you should know how to run, debug, generate, install, and inspect tests.

---

# Sources Used for Additional Interview Coverage

The additional questions above were synthesized and paraphrased from common Playwright interview themes, including:

- TestDino, *Playwright Interview Questions and Answers for QAs and SDETs (2026)*
- NareshIT, *Top Playwright Automation Interview Questions Answers*
- Playwright concepts already covered by the companion cheat sheet

Duplicate topics were intentionally omitted or expanded only when the added material introduced a distinct interview angle.

# Top 15 Questions to Master

If you are short on preparation time, make sure you can explain these without hesitation:

1. Browser vs BrowserContext vs Page
2. What is a Locator?
3. Why prefer `getByRole()`?
4. What is auto-waiting?
5. What are web-first assertions?
6. Why avoid `waitForTimeout()`?
7. What are fixtures?
8. Test-scoped vs worker-scoped fixtures
9. How does `storageState` work?
10. How does Playwright isolate tests?
11. How does Playwright run tests in parallel?
12. How do you intercept or mock an API?
13. What are the pros and cons of Page Object Model?
14. How do you debug flaky tests?
15. Why are `async`, `await`, and Promises important?

---

# Interview Tip

Do not memorize these answers word for word.

For each topic, make sure you can:

- Explain the concept in your own words
- Give one practical example
- Explain when you would use it
- Explain one common mistake
- Write a small code example without looking it up

That is usually much more valuable in an interview than memorizing API syntax.
