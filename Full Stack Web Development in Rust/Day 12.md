You're building a **full-stack Rust app** with **Leptos**, and you've already added:
- ✅ PostgreSQL for persistence
- ✅ JWT-based authentication
- ✅ Real-time updates with WebSockets
- ✅ AI-powered features (OpenAI)
- ✅ Mobile-first responsive UI with Tailwind
- ✅ Unit & integration tests
- ✅ Structured logging with tracing
- ✅ User profiles and settings
- ✅ Dashboard with analytics

Now let’s add **E2E (end-to-end) testing** using **Playwright** — the modern browser automation tool that supports multiple browsers and has great TypeScript/JS support.

---

# 🧪 Add E2E Testing to Your Leptos App with Playwright

## ✅ What We’ll Do:

1. Install **Playwright**  
2. Write test scenarios like:
   - ✅ Register user
   - ✅ Login
   - ✅ Create todo
   - ✅ Logout
3. Run tests in **headless mode**
4. Use **auth fixtures** and **test database setup**
5. Optional: Add GitHub Actions workflow

Let’s go!

---

## 1. 🛠️ Setup Playwright

Make sure your app is running locally on `http://localhost:3000`:

```bash
trunk serve
```

Then install Playwright:

```bash
npm init playwright@latest
```

Follow prompts:
- Yes to installing Playwright
- Choose all browsers
- Yes to adding VSCode extension
- No to open example test

This creates:
- `playwright.config.ts`
- `tests/` and `tests-examples/` folders

---

## 2. 📦 Configure Playwright

Update `playwright.config.ts`:

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  timeout: 30 * 1000,
  expect: {
    timeout: 5000
  },
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
  ],
});
```

---

## 3. 🧪 Write First E2E Test

Create file: `tests/app.spec.ts`

```ts
import { test, expect } from '@playwright/test';

test.describe.serial('Full App E2E', () => {
  test('has title', async ({ page }) => {
    await page.goto('/');
    await expect(page).toHaveTitle(/Rust Todo App/);
  });

  test('can navigate to login', async ({ page }) => {
    await page.click('text=Login');
    await expect(page).toHaveURL('/login');
  });

  test('can register new user', async ({ page }) => {
    await page.goto('/register');
    await page.fill('input[name=email]', 'testuser@example.com');
    await page.fill('input[name=password]', 'password123');
    await page.click('button[type=submit]');
    
    // Wait for redirect or success message
    await expect(page.locator('text=Logged in!')).toBeVisible();
  });

  test('can create todo', async ({ page }) => {
    await page.goto('/todos');

    await page.fill('input[name=title]', 'Buy groceries');
    await page.click('button[type=submit]');

    await expect(page.locator('text=Buy groceries')).toBeVisible();
  });

  test('can toggle todo', async ({ page }) => {
    await page.check('input[type=checkbox]');
    const checkbox = page.locator('input[type=checkbox]').first();
    await expect(checkbox).toBeChecked();
  });

  test('can logout', async ({ page }) => {
    await page.click('button:text("Logout")');
    await expect(page).toHaveURL('/');
  });
});
```

> Adjust selectors based on your actual HTML structure (use dev tools)

---

## 4. 🧱 Optional: Setup Test Database

To avoid polluting your dev DB during testing:

### Option A: Use Docker + Temp DB

Use `sqlx`'s test helpers or run Postgres in Docker:

```bash
docker run -e POSTGRES_USER=test -e POSTGRES_PASSWORD=test -e POSTGRES_DB=testdb -p 5435:5432 postgres
```

Then set:

```env
DATABASE_URL=postgres://test:test@localhost:5435/testdb?sslmode=disable
```

In GitHub Actions, use:

```yaml
services:
  postgres:
    image: postgres
    env:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb
    ports: ["5432:5432"]
```

---

## 5. 🚀 Run Tests

Start your app in one terminal:

```bash
trunk serve
```

In another:

```bash
npx playwright install-deps
npx playwright test
```

To see browser:

```bash
npx playwright show-trace ./test-results/app-spec@chromium/
```

---

## 6. 🧩 Add Fixtures (Optional Advanced)

Create reusable fixtures like logged-in user:

```ts
// fixtures.ts
import { fixtures } from '@playwright/test';

fixtures.user = async ({ page }, use) => {
  const email = 'testuser@example.com';
  const password = 'password123';

  await page.goto('/register');
  await page.fill('input[name=email]', email);
  await page.fill('input[name=password]', password);
  await page.click('button[type=submit]');

  await use({ email, password });
};
```

Then use it in test:

```ts
test('logged in user can see dashboard', async ({ user, page }) => {
  await page.goto('/dashboard');
  await expect(page.locator('text=Dashboard')).toBeVisible();
});
```

---

## ✅ Summary

✅ Set up Playwright for E2E testing  
✅ Wrote test suite covering full user flow  
✅ Used serial tests to simulate real usage  
✅ Optional: Added fixtures and test DB setup  

---

## 🚀 Next:

1. 📦 Adding **file uploads** (e.g., profile picture or CSV export)  
2. 🧑‍💼 Adding **roles/permissions** (admin/user)  
3. 📅 Adding **calendar/todo scheduling** feature  
4. 🧵 Writing a **custom Leptos component library**
