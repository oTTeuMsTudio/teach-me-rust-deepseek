# Building a full-stack Rust web app like a seasoned engineer.

Let’s take the next step and **add end-to-end browser testing** using **Playwright**, so every change you make is automatically tested in real browsers.

This will:
- ✅ Test frontend interactivity (e.g., adding/deleting todos)
- ✅ Ensure login/logout works
- ✅ Run on CI (GitHub Actions) after every push
- ✅ Help prevent regressions

---

# 🧪 Step 1: Install Playwright

We’ll use JavaScript-based Playwright tests because it's well-supported, even though our app is built in Rust. The frontend is still HTML/JS/WASM, so Playwright can test it just fine.

### a. Add Playwright to your project:

```bash
npm init playwright@latest
```

Follow the prompts:
- Yes to installing Playwright
- Yes to installing Playwright Test CLI
- Yes to installing browser binaries

This creates:
- `playwright.config.ts`
- `playwright/` folder with global setup info
- `tests/` and `tests-examples/` folders

You can delete the example tests.

---

# 📁 Step 2: Write Your First End-to-End Test

Create a new file: `tests/todo.spec.ts`

```ts
import { test, expect } from '@playwright/test';

test('has title', async ({ page }) => {
  await page.goto('http://localhost:3000');

  // Expect a heading "To-Do List"
  await expect(page.getByRole('heading', { name: 'To-Do List' })).toBeVisible();
});

test('can add a todo', async ({ page }) => {
  await page.goto('http://localhost:3000');

  await page.fill('input[type=text]', 'Buy groceries');
  await page.click('button[type=submit]');

  await expect(page.getByText('Buy groceries')).toBeVisible();
});

test('can delete all todos', async ({ page }) => {
  await page.goto('http://localhost:3000');

  await page.click('button:has-text("Delete All")');

  await expect(page.getByText('Buy groceries')).not.toBeVisible();
});
```

> These assume your dev server runs on port 3000.

---

# 🛠️ Step 3: Update GitHub Action to Run Tests

Update your `.github/workflows/ci.yml` or create a new one:

### Add this job at the bottom of your existing CI workflow:

```yaml
  e2e-tests:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install-deps && npx playwright install

      - name: Start Dev Server
        run: |
          cd my-todo-app
          trunk serve --port 3000 &
          sleep 10 # Wait for server to start
        working-directory: ${{ github.workspace }}

      - name: Run E2E Tests
        run: npx playwright test
```

> Replace `my-todo-app` with the actual folder name if different.

---

# 🧪 Step 4: Test It Locally

Run your dev server:

```bash
trunk serve --port 3000
```

Then run tests:

```bash
npx playwright test
```

You can also open the UI:

```bash
npx playwright show-trace
```

---

# 🚀 Done! Now Every Push Runs Browser Tests

✅ Playwright tests added  
✅ Run in GitHub Actions  
✅ Verify real user flows  
✅ Prevent breaking changes

---

## 🎯 Next:
1. Add **custom domain & SSL**
2. Set up **logging + error tracking**
3. Add **analytics dashboard**
4. Set up **Docker support**
