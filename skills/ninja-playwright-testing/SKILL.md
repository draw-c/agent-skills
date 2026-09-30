---
name: ninja-playwright-testing
description: Elite ninja Playwright testing craft — locators, auto-waiting, fixtures, auth, network mocking, mobile emulation, visual regression, API testing, and CI. Pairs with ninja-playwright for cleanup.
---

# 🥷 Ninja Playwright Testing Skill

> *"Auto-wait is the way. Sleeps are the enemy."*

## Core Principles (The Ninja Code)

- **Never use `waitForTimeout`.** Hard waits lower performance and introduce flakiness. Playwright's auto-waiting handles actionability — visible, stable, enabled, receives events, single element.
- **Web-first assertions (`toBe*`) auto-retry.** `expect(locator).toBeVisible()` waits; a manual poll does not.
- **Locators over selectors.** `getByRole`, `getByLabel`, `getByText`, `getByTestId` — semantic, resilient, user-first.
- **One source of truth for auth.** Capture `storageState` once, reuse across tests; never log in per-test.
- **Mock at the network boundary.** `page.route()` intercepts, fulfills, or modifies — no backend dependency for UI tests.
- **Mobile emulation ≠ native app testing.** Playwright emulates mobile *browsers* (viewport, touch, UA). Native iOS/Android is Appium/Maestro territory.

## Locators (The Strike)

Priority order — use the highest that fits:

| Locator | Use when | Example |
|---|---|---|
| `getByRole` | Elements with implicit ARIA role | `page.getByRole('button', { name: 'Login' })` |
| `getByLabel` | Form inputs with `<label>` | `page.getByLabel('Order Number').fill('ORD-001')` |
| `getByPlaceholder` | Inputs with placeholder | `page.getByPlaceholder('Search').fill('ORD-001')` |
| `getByText` | Non-interactive content | `page.getByText('Northwind Traders').click()` |
| `getByTestId` | Last resort; `data-testid` | `page.getByTestId('create-blank-canvas-app').click()` |
| `page.locator('css')` | CSS fallback — document why | `page.locator('textarea[data-id="notes"]')` |

**Patterns:**

```javascript
// Exact match to avoid partial hits
await page.getByText('ORD-001', { exact: true }).click();

// Regex match
await page.getByText(/Order \d+/).first().click();

// Chaining — scope child to parent
page.getByRole('dialog').getByRole('button', { name: 'Delete' });

// Parameterized template
page.locator(`[role="rowheader"] a:has-text("${appName}")`);
```

## Auto-Waiting & Assertions (The Silent Kill)

**Actions auto-wait.** `click()`, `fill()`, `selectOption()` wait for actionability — no explicit waits needed.

**Web-first assertions auto-retry:**

```javascript
// Waits for visible, stable, enabled, receives events
await expect(page.getByRole('button', { name: 'Login' })).toBeVisible();
await expect(page.getByRole('button', { name: 'Login' })).toBeDisabled();
await expect(page.getByRole('button', { name: 'Login' })).toBeHidden();

// Text content — retries until match
await expect(page.getByRole('heading')).toHaveText('Dashboard');
```

**Anti-patterns:**
```javascript
// ❌ Hard wait — flaky, slow
await page.waitForTimeout(2000);
await page.click('#login');

// ❌ Manual poll — reinvents web-first assertions
while (await page.locator('.spinner').isVisible()) { await sleep(100); }

// ✅ Auto-waiting + web-first assertion
await expect(page.locator('.spinner')).toBeHidden();
await page.getByRole('button', { name: 'Login' }).click();
```

## Authentication (Storage State)

Capture once in `globalSetup`, reuse across all tests:

```javascript
// global-setup.ts
import { chromium, FullConfig } from '@playwright/test';

async function globalSetup(config: FullConfig) {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  await page.goto('/login');
  await page.getByLabel('Email').fill('user@contoso.com');
  await page.getByLabel('Password').fill(process.env.TEST_PASSWORD);
  await page.getByRole('button', { name: 'Sign in' }).click();
  await page.waitForURL('/dashboard');
  await page.context().storageState({ path: 'auth.json' });
  await browser.close();
}
export default globalSetup;
```

```javascript
// playwright.config.ts
export default defineConfig({
  globalSetup: './global-setup.ts',
  use: { storageState: 'auth.json' },
});
```

**Multiple users** — separate state files per role:
```javascript
// In a specific test file
test.use({ storageState: 'auth-admin.json' });
```

**Security:**
- `.playwright-ms-auth/` and `auth.json` go in `.gitignore`
- Never commit tokens; use env vars or secret manager
- Dedicated test accounts with least privilege

## Network Mocking (The Shadow)

Intercept, fulfill, modify, or abort:

**Mock — no API call:**
```javascript
await page.route('*/**/api/v1/fruits', async route => {
  await route.fulfill({ json: [{ name: 'Strawberry', id: 21 }] });
});
```

**Modify — call API, patch response:**
```javascript
await page.route('*/**/api/v1/fruits', async route => {
  const response = await route.fetch();
  const json = await response.json();
  json.push({ name: 'Loquat', id: 100 });
  await route.fulfill({ response, json });
});
```

**Block — abort requests:**
```javascript
await page.route('**/*.{png,jpg,jpeg}', route => route.abort());
await page.route('**/*.css', route => route.abort());
```

**Modify request headers:**
```javascript
await page.route('**/*', async route => {
  const headers = route.request().headers();
  delete headers['X-Secret'];
  await route.continue({ headers });
});
```

**Wait for response:**
```javascript
const responsePromise = page.waitForResponse('**/api/fetch_data');
await page.getByText('Update').click();
const response = await responsePromise;
```

**HAR files** — record once, replay in CI:
```javascript
await page.routeFromHAR('hars/fruits.har', { url: '*/**/api/v1/fruits' });
```

## Mobile Emulation

Playwright emulates mobile *browsers* — viewport, touch, device scale, UA:

```python
# Python — pytest fixture
import pytest
from playwright.sync_api import sync_playwright

@pytest.fixture
def mobile_page(playwright):
    iphone = playwright.devices['iPhone 13']
    browser = playwright.chromium.launch()
    context = browser.new_context(**iphone)
    page = context.new_page()
    yield page
    context.close()
    browser.close()
```

**Touch interactions:**
```python
def test_touch_swipe(mobile_page):
    mobile_page.goto("https://example.com/gallery")
    mobile_page.locator(".gallery").swipe_left()
    mobile_page.tap(".image-thumbnail")
```

**Orientation:**
```python
def test_orientation(mobile_page):
    mobile_page.set_viewport_size({"width": 844, "height": 390})
```

**Scope boundary:** This is Chromium/Firefox/WebKit with mobile viewport and touch — same as Chrome DevTools device toolbar. Real mobile OS (Android/iOS) requires Appium or Maestro.

## Visual Regression

First run captures baseline; subsequent runs compare pixel-by-pixel:

```javascript
// tests/visual.spec.ts
test('design system matches baseline', async ({ page }) => {
  await page.goto('/design-system');
  // Stability gate — wait for heading before screenshot
  await expect(page.getByRole('heading', { level: 1 })).toBeVisible();
  await expect(page).toHaveScreenshot('design-system.png', { fullPage: true });
});
```

**Config — eliminate false positives:**
```javascript
// playwright.config.ts
expect: {
  toHaveScreenshot: {
    animations: 'disabled',      // freeze CSS animations
    caret: 'hide',               // hide blinking cursor
    scale: 'css',                // normalize Retina vs CI
    maxDiffPixelRatio: 0.01,     // allow 1% diff
  },
},
```

**Mask dynamic content:**
```javascript
await expect(grid).toHaveScreenshot('orders-grid.png', {
  mask: [page.locator('[col-id="modifiedon"]')],  // mask timestamps
});
```

**Update baselines intentionally:**
```bash
npx playwright test --update-snapshots
```

## API Testing (The Double Strike)

Validate API contracts alongside UI — same Playwright test, `request` fixture:

```javascript
test('create and verify order via API', async ({ request, page }) => {
  // API — create
  const created = await request.post('/api/orders', {
    data: { customerId: 1, items: [{ sku: 'A1', qty: 2 }] },
  });
  expect(created.status()).toBe(201);
  const { id } = await created.json();

  // API — verify contract (not just status)
  const fetched = await request.get(`/api/orders/${id}`);
  const body = await fetched.json();
  expect(body).toMatchObject({
    id,
    status: 'pending',
    total_cents: expect.any(Number),
  });

  // UI — verify rendered
  await page.goto(`/orders/${id}`);
  await expect(page.getByText(`Order #${id}`)).toBeVisible();
});
```

**Pitfalls:**
- Don't assert only `status === 200` — check response body shape
- Don't hardcode tokens — fetch fresh in `beforeAll`
- Don't treat `page.route()` mocks as API coverage — they're isolation, not validation
- Fail closed on schema drift — non-zero exit, not a warning

## CI (The Gate)

```yaml
# .github/workflows/playwright.yml
name: Playwright
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npx playwright install --with-deps
      - run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-trace
          path: test-results/
```

**Config for CI:**
```javascript
export default defineConfig({
  retries: 2,                    // 2 max — 3+ means the test is flaky
  trace: 'on-first-retry',       // full timeline on flaky failure
  reporter: [['html'], ['junit', { outputFile: 'results.xml' }]],
});
```

## Review Checklist (The Strike Test)

- [ ] No `waitForTimeout` — auto-waiting actions and web-first assertions only
- [ ] Locators use `getByRole`/`getByLabel`/`getByTestId` — CSS only with documented reason
- [ ] `storageState` captured once in `globalSetup`, reused across tests
- [ ] `auth.json` in `.gitignore`; no tokens committed
- [ ] Network mocks use `page.route()` — fulfill, modify, or abort
- [ ] Mobile tests use `playwright.devices` presets; native app testing is out of scope
- [ ] Visual regression has stability gate (`toBeVisible` before screenshot)
- [ ] `toHaveScreenshot` config: `animations: 'disabled'`, `caret: 'hide'`, `scale: 'css'`
- [ ] API assertions check body shape, not just status code
- [ ] `retries: 2` max — more means the test is flaky and needs fixing
- [ ] `trace: 'on-first-retry'` set for debugging
- [ ] CI uploads trace artifacts on failure