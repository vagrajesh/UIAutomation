# UIAutomation

# Playwright **Services/Agents** Framework — One‑Click Project Generator (with Optional UI Runner)

> **Purpose**  
> This README is a **single, copy‑pasteable file** you can drop into an empty repo and use with **GitHub Copilot** or **Claude** to *generate the entire project*: a scalable Playwright **Services/Agents** automation framework, example tests (including **multi‑user flows**), **isolated/contract** tests, and an optional **React dashboard** to list & run tests and visualize **Agent steps**.

---

## Why this framework?

- **Services** = structure & selectors (context‑scoped locators per page/section)
- **Agents** = flows & business steps (`test.step(...)` for readable reports)
- **Context‑scoped** locators avoid cross‑page collisions and flakiness
- **Multi‑page & multi‑user friendly** (separate browser contexts, shared RunContext)
- **Fast inner loop** via **isolated DOM** + **contract tests**
- **Optional UI Runner** to:
  - set **Base URL**
  - **discover** tests
  - **run** selected tests (or grep)
  - see **steps** emitted by Agents

> You’ll end up with a maintainable test suite that matches how your app is organized (auth, products, users, approvals…), with clean orchestration for end‑to‑end scenarios.

---

## What gets generated

.
├─ package.json
├─ playwright.config.ts
├─ tsconfig.json
├─ .gitignore
├─ README.md  ← (this file)
├─ src/
│  ├─ types/
│  │  └─ run-context.ts
│  ├─ helpers/
│  │  └─ within.ts
│  ├─ services/
│  │  ├─ auth.login.page.ts
│  │  ├─ products.page.ts
│  │  ├─ users.admin.page.ts
│  │  └─ approvals.page.ts
│  ├─ agents/
│  │  ├─ base.agent.ts
│  │  ├─ auth.agent.ts
│  │  ├─ product.agent.ts
│  │  ├─ user-admin.agent.ts
│  │  ├─ approval.agent.ts
│  │  └─ orchestrator.ts
│  └─ fixtures/
│     └─ agents.fixture.ts
├─ tests/
│  ├─ advances.spec.ts                         # public demo-style smoke (anchors on stable labels)
│  ├─ advances.agent.isolated.spec.ts          # isolated agent test via page.setContent()
│  ├─ services.rate-sheet.contract.spec.ts     # contract test for a component service
│  └─ e2e-create-product-user-approval.spec.ts # multi-user flow (admin + approver)
└─ ui-runner/                                   # OPTIONAL dashboard
├─ server/
│  ├─ src/
│  │  ├─ index.ts
│  │  ├─ routes.ts
│  │  ├─ memoryStore.ts
│  │  ├─ playwrightService.ts
│  │  └─ customReporter.ts
│  ├─ package.json
│  └─ tsconfig.json
└─ web/
├─ src/
│  ├─ api.ts
│  ├─ main.tsx
│  ├─ App.tsx
│  └─ components/
│     ├─ BaseUrlForm.tsx
│     ├─ TestList.tsx
│     ├─ RunnerPanel.tsx
│     └─ ResultsView.tsx
├─ package.json
├─ tsconfig.json
├─ vite.config.ts
└─ index.html

---

## How to use this README with **Copilot** or **Claude**

You’ll feed the prompts below step‑by‑step. Each prompt asks the assistant to generate a set of files.  
**Copy the produced code blocks into your repo** with the same paths and filenames.

> **Tip**: You can also ask the assistant to “create the files in place” if you’re using an IDE plugin that supports direct file writes.

---

# 🧠 Prompts to Generate the Project

> Run these in order. Paste *each* prompt into GitHub Copilot Chat or Claude, then copy the resulting code into your repo.

---

### Prompt 1 — Initialize Node + Playwright + TS

```text
Create the following in the current folder for a Playwright TypeScript project:

- package.json with scripts:
  - "test": "playwright test"
  - "test:headed": "playwright test --headed"
  - "codegen": "playwright codegen"
- devDeps: @playwright/test (latest), typescript ^5, ts-node ^10
- tsconfig.json targeted to ES2021, module ESNext, strict true, include src, tests, playwright.config.ts
- .gitignore for node_modules, test-results, playwright-report, blob-report, trace.zip, dist, .DS_Store
- playwright.config.ts that:
  - sets use.baseURL = process.env.BASE_URL || "https://www.fhlbdm.com"
  - sets use: { trace: 'on-first-retry', video: 'retain-on-failure', screenshot: 'only-on-failure' }
  - config: timeout 90_000, expect.timeout 10_000, retries 1, reporter html

Do not install dependencies; only output the files' contents in separate code blocks.


Prompt 2 — Core Types, Helpers, Services
Generate these files under src/:

- types/run-context.ts: defines RunContext { product?: { id?: string; name?: string }; users?: { requester?: string; approver?: string } }
- helpers/within.ts: small helper that returns byRole, byText, byTestId, css from a Locator root.
- services/auth.login.page.ts: login page object with goto('/login'), username(), password(), submit(), expectLoaded()
- services/products.page.ts: products module with goto('/products'), newProductButton(), newProductRoot(), withinNew(), rowByName(name), expectListVisible()
- services/users.admin.page.ts: admin users module with goto('/admin/users'), newUserBtn(), createUser(email, role)
- services/approvals.page.ts: approvals inbox with goto('/inbox'), requestRowByProduct(name), approveProduct(name)

Use Playwright's getByRole/getByLabel/getByText and avoid brittle CSS. Return code in separate TS code blocks.
``

Prompt 3 — Agents + Orchestrator
Create these agent classes under src/agents:

- base.agent.ts: BaseAgent(page, runContext). step(title, fn) wraps test.step.
- auth.agent.ts: login(username, password) using LoginPage service.
- product.agent.ts: createProduct({ name, sku? }) using ProductsPage; save rc.product.name.
- user-admin.agent.ts: createRequesterAndApprover({ requester, approver }) using UsersAdminPage; save rc.users.
- approval.agent.ts: approveProductBySecondUser() using ApprovalsPage; reads rc.product.name.
- orchestrator.ts: class Orchestrator(browser, runContext) with withAdmin(fn) and withApprover(fn), each opening a newContext (optionally using storageState paths), instantiating relevant agents, closing after.

One file per class, TypeScript code blocks only.

Prompt 4 — Playwright Fixture
Create src/fixtures/agents.fixture.ts exporting:
- test: a Playwright test extended with an "agents" fixture that constructs Orchestrator(browser, runContext)
- expect: re-export of test.expect

We will import { test, expect } from this fixture in specs that want the orchestrator injected.
Return code in a TS block.

Prompt 5 — Tests (public example + multi-user E2E + isolated + contract)
Create these tests under /tests:

1) advances.spec.ts — navigate to `${process.env.BASE_URL || 'https://www.fhlbdm.com'}/products-services/advances/`, assert:
   - heading "Advances" visible
   - presence of a "Market Rates" section (structure-based)
   - presence of "Download Product Rate Sheet" links (Excel/PDF/Print)
   Use role/text selectors only and avoid asserting dynamic numbers.

2) e2e-create-product-user-approval.spec.ts — Uses Orchestrator to:
   - Admin context: (optionally login) create requester & approver users, create a product with unique name.
   - Approver context: (optionally login) approve the product.
   Use timestamp-based names to avoid collisions.

3) advances.agent.isolated.spec.ts — Isolated DOM with page.setContent() including minimal HTML that has:
   - <h1>Advances</h1>, a "Market Rates" section, and a "Download Product Rate Sheet" group with Excel/PDF/Print links that open a popup.
   Instantiate the AdvancesAgent and run validations and popup test without real navigation.

4) services.rate-sheet.contract.spec.ts — Contract test for a RateSheetActions-like service against a minimal HTML snippet validating it exposes Excel/PDF/Print links within a scoped container.

Return separate .ts code blocks.

Prompt 6 — UI Runner (Server)
Under ui-runner/server create:
- package.json with scripts: dev (ts-node-esm src/index.ts), build (tsc -p .), start (node dist/src/index.js)
- tsconfig.json for ES2021, ESNext modules
- src/memoryStore.ts: in-memory store for baseURL, tests list, runs; RunInfo/TestResult types
- src/customReporter.ts: Playwright reporter that collects per-test steps (from test.step) and writes JSON to REPORT_OUT
- src/playwrightService.ts: functions listTests() and runTests(run), spawning npx playwright. It must use env BASE_URL from MemoryStore, `--reporter=json` plus the custom reporter (path), then merge JSON results with step data.
- src/routes.ts: Express router with endpoints:
    GET /api/base-url, POST /api/base-url
    GET /api/tests
    POST /api/runs  (starts a run async)
    GET /api/runs/:id
    GET /api/runs/:id/stream  (SSE for live logs + completion)
- src/index.ts: Express app wiring, CORS, JSON, /api routes, and serve static ui-runner/web/dist in production.

Return code files in separate TS or JSON blocks as appropriate. Do not install packages.
``
Prompt 7 — UI Runner (Web)
Under ui-runner/web create:
- package.json with scripts dev/build/preview using Vite and React 18
- tsconfig.json for React JSX
- vite.config.ts that proxies /api to http://localhost:5050
- index.html with a #root div
- src/api.ts: fetch helpers for /api endpoints
- src/components/: BaseUrlForm.tsx, TestList.tsx, RunnerPanel.tsx, ResultsView.tsx
- src/App.tsx: layout combining components
- src/main.tsx: React root

Keep styles inline/minimal. Output code files in separate blocks.

Prompt 8 — Root Workspace Scripts (Optional)

Add a ui-runner/package.json at the ui-runner root with:
- workspaces or simple npm scripts to run server and web in dev concurrently (using 'concurrently')
- scripts: dev (server+web), build (web then server), start (server serves built web)
Also add developer instructions in a short README section.

Return the JSON file content.


Install & Run

# 1) Install Playwright deps at repo root
npm i
npx playwright install

# 2) (Optional) UI Runner deps
cd ui-runner/server && npm i && cd ../web && npm i && cd ..

# 3) Run tests headless/headed
npm test
npm run test:headed

# 4) (Optional) Run the UI Runner (dev)
npm run dev --prefix ui-runner
# API: http://localhost:5050
# Web: http://localhost:5173  (proxied to API)

# 5) (Optional) Production serve of UI Runner
npm run build --prefix ui-runner
npm run start --prefix ui-runner
# Open http://localhost:5050


Best Practices & Tips

Selectors: Prefer getByRole, getByLabel, getByText. If your app exposes data-testid, set it once in playwright.config.ts (use.testIdAttribute) and use getByTestId.
Scoping: Always derive locators from a root (Page or Locator). Compose: page ➜ section ➜ component ➜ element.
Waiting: Avoid waitForTimeout. Use assertions that encode state: modal visible/hidden, toast appears, row exists, etc.
Multi‑user: Use separate contexts (or projects) and storageState per role for fast, stable auth.
Cross‑context: Share identity (name/ID), not DOM objects; re‑locate in the other context using Services.
Data: Make resources unique (timestamp/uuid). Clean up if the app supports it; otherwise run in isolated tenants.
Observability: Use test.step(...) inside agents. Keep trace: 'on-first-retry' and video: 'retain-on-failure' while stabilizing.


Troubleshooting


UI Runner can’t list tests
Ensure npx playwright test --list works at repo root. If server is run from another cwd, adjust cwd in playwrightService.ts.


Steps don’t show in Results
Confirm the custom reporter is passed alongside JSON reporter and that REPORT_OUT is set for the run. Agents must use test.step(...).


Flaky flows
Add preconditions inside agent steps (assert page state) and prefer role/label selectors. Avoid indexing (.nth()) unless the list is anchored.


Need to switch environments
Set BASE_URL at runtime (BASE_URL=https://env.example.com npm test) or via the UI Runner Base URL form.
