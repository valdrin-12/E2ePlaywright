# Learning Log: Building a Playwright + TypeScript Framework

A step-by-step record of how this framework was built and why each decision was made.

---

## Step 1: Initialize the project

**What we did and why**
- Ran `git init` so the folder is a git repository. `.gitignore` and the CI workflow only make sense inside a repo.
- Ran `npm init playwright@latest`, the official scaffolder. It created `package.json`, installed `@playwright/test`, generated a starter config, an example test and a GitHub Actions workflow, and downloaded the browsers.
- Ran the example test once to confirm the install works before changing anything.

**Key commands**
```bash
git init
npm init playwright@latest -- --quiet --lang=ts --gha --browser=chromium --browser=firefox
npx playwright test --reporter=list
```
Run interactively, the command asks: TypeScript or JavaScript → TypeScript, tests folder → `tests`, add GitHub Actions workflow → yes, install browsers → yes.

**Generated files**
| File | Purpose |
|---|---|
| `package.json` | Project manifest. Lists `@playwright/test` and `@types/node` as **devDependencies** (test tooling never ships to production). |
| `package-lock.json` | Locks exact dependency versions so every machine (and CI's `npm ci`) installs the same thing. Commit it. |
| `node_modules/` | Installed packages. Never committed (it's in `.gitignore`). |
| `playwright.config.ts` | The central configuration: where tests live, parallelism, retries, reporters, browsers ("projects"). |
| `tests/example.spec.ts` | A sample test against playwright.dev. We delete it later. |
| `.github/workflows/playwright.yml` | A GitHub Actions pipeline that runs the tests on every push or pull request. |
| `.gitignore` | Keeps generated output (`test-results/`, `playwright-report/`) and `node_modules/` out of git. |

Browsers are downloaded to a shared cache (`~/Library/Caches/ms-playwright` on macOS), not into the project.

**Key concepts**
- **@playwright/test**: the test runner with assertions built in. It is a different package from the bare `playwright` library.
- **Projects**: a project is a named set of settings (for example "chromium" or "firefox"). The same tests run once per project.
- **Workers**: separate processes that run tests in parallel. That's why 4 tests ran at the same time.
- **devDependencies vs dependencies**: test tools are devDependencies because the shipped app doesn't need them.

**Interview Q&A**
- **Q: How do you set up a new Playwright project?**
  A: `npm init playwright@latest`. It installs `@playwright/test`, creates `playwright.config.ts`, an example spec and optionally a GitHub Actions workflow, and downloads browser binaries. After that, `npx playwright test` runs everything.
- **Q: Why is `@playwright/test` a devDependency?**
  A: It's only needed to develop and test the app, not to run it in production, so it shouldn't end up in the production bundle or install.
- **Q: Your tests pass, but you want to see what the browser is doing. What are your options?**
  A: `--headed` to show the browser, `--debug` to step through with the Inspector, `--ui` for UI mode, or `--trace on` and replay the trace in the trace viewer. On CI I rely on traces because CI has no screen.

**Side note: seeing the browser (headless vs headed)**
- Playwright runs **headless** by default, so no window appears. That's faster and works on CI servers, which have no screen.
- To watch a run: `npx playwright test --project=chromium --headed --workers=1`
  - `--project=chromium` runs only that project, `--headed` shows the window, `--workers=1` gives you one window at a time.
- To step through actions one at a time: `npx playwright test --debug` (Playwright Inspector).
- **Chromium vs Google Chrome:** the `chromium` project uses Playwright's bundled Chromium. To use real installed Chrome, add `channel: 'chrome'` to the project's `use`.
- **Best practice:** keep `headless` as the default in the config and pass `--headed` only when you want to watch. Don't set `headless: false` in the config, because it breaks CI.
