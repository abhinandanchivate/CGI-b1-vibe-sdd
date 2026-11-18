# Antigravity CLI + Spec Kit (SDD) Setup Guide for Windows — Enterprise Account

Nov 18, 2026 · @Abhi

## Overview

**Important change:** Google has replaced Gemini CLI with **Antigravity CLI** (command: `agy`). Since 18 June 2026, Gemini CLI no longer serves free, Google AI Pro or Ultra accounts; it still works only with a Gemini Code Assist Standard/Enterprise license or a paid API key. This guide therefore uses Antigravity CLI. If your company already pays for Gemini Code Assist and wants to stay on Gemini CLI, see the note at the end of Step 3.

By the end of this guide, `agy` will run on Windows under your company account, and GitHub Spec Kit will add SDD skills such as `/speckit-specify` and `/speckit-plan` inside it. Setup takes about 30 minutes, plus any wait for IT.

The one decision to make first is how your company account pays for usage. Google Workspace subscriptions don't include Antigravity, so a work account needs one of these options (from Google's official Workspace page):

| Option | How it's paid | What IT must do |
| --- | --- | --- |
| **A: Gemini Enterprise license** | Seat license with included credits; overage at Agent Platform rates | Buy Gemini Enterprise, turn on AI developer tools, assign you a license |
| **B: Google Cloud project (pay as you go)** | Per token, billed to the company's Cloud Billing account | Enable the Agent Platform API, grant you the Agent Platform User role |
| Personal Google account | Free, or Google AI Pro/Ultra | Nothing; not suitable for company code unless IT approves |

If you sign in with your work email through plain "Continue with Google", you only get a small starter quota, or an "account is ineligible" message if your admin has blocked individual access.

All commands below are for **Windows PowerShell** (Start menu → search "PowerShell"). Where a step says "reopen PowerShell", close every PowerShell window and open a new one so it picks up changes.

## Before you start

Collect these from your IT or cloud admin first; most enterprise sign-in failures come from a missing license, role or setting on the admin side.

- [ ] Which option is approved: **A** (Gemini Enterprise license) or **B** (Google Cloud project, pay as you go)
- [ ] The Google Cloud **project ID** (looks like `my-company-ai-123456`, not the display name)
- [ ] For Option A: confirmation that a Gemini Enterprise Standard or Plus license is assigned to your work email, that **AI developer tools** is turned on (Gemini Enterprise → Settings → AI developer tools), and that you have the **Gemini Enterprise User** role (`roles/discoveryengine.agentspaceUser`)
- [ ] For Option B: confirmation that billing is active, the **Agent Platform API** (`aiplatform.googleapis.com`, formerly Vertex AI) is enabled, and you have the **Agent Platform User** role (`roles/aiplatform.user`)
- [ ] The region to use: `global`, `us` or `eu`
- [ ] Permission to install software on your laptop
- [ ] If your network uses a proxy or firewall allow-list, the proxy address (see Troubleshooting)

## Step 1: Install the prerequisites

You need three tools for Spec Kit: Python 3.11+, Git and uv. Node.js is **not** needed for Antigravity CLI, which installs as a standalone program. The fastest way on Windows 10/11 is `winget`, which is built in.

1. Open **PowerShell** and run each line:

   ```powershell
   winget install --id Python.Python.3.12 -e
   winget install --id Git.Git -e
   ```
2. Install **uv** (the Python tool manager Spec Kit uses):

   ```powershell
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```
3. Allow PowerShell to run installed scripts for your user:

   ```powershell
   Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
   ```

   Type `Y` when asked. If company policy blocks this, ask IT, or use the Command Prompt (cmd) install route in Step 2.
4. **Reopen PowerShell**, then verify:

   ```powershell
   python --version    # 3.11 or higher
   git --version
   uv --version
   ```

If `winget` is unavailable or blocked, download the installers from python.org (tick **Add python.exe to PATH**) and git-scm.com. The Google Cloud CLI (`gcloud`) is only needed for the optional headless sign-in described in Step 3.

## Step 2: Install Antigravity CLI (`agy`)

Google's official installer for Windows runs in PowerShell:

```powershell
irm https://antigravity.google/cli/install.ps1 | iex
```

If PowerShell scripts are blocked, use **Command Prompt (cmd)** instead:

```
curl -fsSL https://antigravity.google/cli/install.cmd -o install.cmd && install.cmd && del install.cmd
```

**Reopen PowerShell** and confirm the command is found:

```powershell
agy --version
```

Enterprise sign-in with a Google Cloud project needs Antigravity CLI **v1.4.11 or later**. If you used Gemini CLI before, the installer detects your old Gemini CLI settings and offers to copy them over. Don't sign in yet; Step 3 covers which sign-in option to pick.

## Step 3: Sign in with your enterprise account

Antigravity CLI stores your sign-in in the Windows credential store, so you sign in once inside `agy`; unlike the old Gemini CLI, no `GOOGLE_CLOUD_PROJECT` environment variables are needed for interactive use. Follow **either** Option A or Option B. Screen names below follow Google's docs and may differ slightly between versions.

### Option A: Gemini Enterprise license

1. Open PowerShell in any folder and run `agy`. If it is already signed in with another account, type `/logout` first, then run `agy` again.
2. In **Select login method**, choose **Use a Google Cloud project** (the business-account route).
3. Choose **Continue with Google Cloud** and sign in with your **work email** in the browser.
4. In **Select billing model**, choose **Gemini Enterprise** and pick your license (for example Gemini Enterprise Plus).
5. Enter the project ID and region from IT, then send a test prompt such as `hello`.

If you see "Contact your administrator to set up Gemini Enterprise for this project" or "No Gemini Enterprise license was found", the admin still has to turn on **AI developer tools**, assign your license to that project and region, or grant the Gemini Enterprise User role.

### Option B: Google Cloud project (pay as you go)

1. Run `agy` (type `/logout` first if already signed in).
2. In **Select login method**, choose **Use a Google Cloud project**.
3. Choose **Continue with Google Cloud** and sign in with your **work email** in the browser.
4. In **Select billing model**, choose **Pay As You Go**.
5. Enter your project ID, choose the region (`global`, `us` or `eu`), then send a test prompt.

Usage is billed per token to the project at Agent Platform pricing, and Antigravity collects no telemetry under Google Cloud terms. To switch project or region later, `/logout` and sign in again.

### Optional: headless or automated use

For build servers or scripts without a browser, Antigravity can use Google Cloud Application Default Credentials. Install the Google Cloud CLI (`winget install --id Google.CloudSDK -e`), run `gcloud auth application-default login`, and follow the ADC section of [Antigravity in Gemini Enterprise](https://antigravity.google/docs/enterprise/). The CLI can also use a Gemini API key from Google AI Studio, but that bills to a personal or AI Studio account, so only use it if IT approves.

### If your company wants to stay on Gemini CLI

Gemini CLI still works with a **Gemini Code Assist Standard or Enterprise** license or a paid API key. In that case: install Node.js 20+ (`winget install --id OpenJS.NodeJS.LTS -e`), run `npm install -g @google/gemini-cli`, run `setx GOOGLE_CLOUD_PROJECT "your-project-id"`, reopen PowerShell, run `gemini` and choose **Login with Google**. In Step 5, use `--integration gemini` instead of `agy`; the commands are then written with dots (`/speckit.specify`) instead of hyphens. New setups should prefer Antigravity, since Google has moved its terminal tooling there.

## Step 4: Install Spec Kit (the `specify` CLI)

Spec Kit is GitHub's open-source SDD toolkit. Install its CLI once with uv, pinned to a stable release. Open the [Spec Kit releases page](https://github.com/github/spec-kit/releases), copy the tag marked **Latest** (1.0.x at the time of writing), and put it in place of `vX.Y.Z`:

```powershell
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

**Reopen PowerShell**, then check it:

```powershell
specify version
specify check
```

`specify check` should find Git and `agy`. To upgrade later, rerun the install command with `--force` and the newer tag.

## Step 5: Set up your project for Antigravity

In Spec Kit, Antigravity is its own integration with the key **`agy`** (not `gemini`). It installs the SDD commands as Antigravity **skills**.

**New project:**

```powershell
cd C:\Projects
specify init my-project --integration agy --script ps
cd my-project
```

**Existing code folder** (run inside it):

```powershell
cd C:\Projects\existing-app
specify init --here --integration agy --script ps
```

**Project already set up with Spec Kit for another agent** (for example the old Gemini CLI): add Antigravity with `specify integration install agy`.

`--script ps` gives you the PowerShell versions of the helper scripts, which is what you want on Windows. After it finishes, your project contains:

| Folder / file | What it holds |
| --- | --- |
| `.agents/skills/speckit-<command>/SKILL.md` | The Spec Kit skills Antigravity picks up (`speckit-specify`, `speckit-plan`, …) |
| `.specify/memory/constitution.md` | Project-wide rules applied to every feature |
| `.specify/scripts/powershell/` | Helper scripts the skills call |
| `.specify/templates/` | Templates for specs, plans and tasks |
| `specs/` | Created later: one folder per feature with its spec, plan and tasks |

Commit these folders to Git so your team shares the same skills and rules.

## Step 6: Run the SDD workflow

Open PowerShell in your project folder and start Antigravity there, so it picks up the skills in `.agents/skills/`:

```powershell
cd C:\Projects\my-project
agy
```

Type `/speckit-` to see the skills. With the Antigravity integration they use **hyphens, not dots**, and Antigravity's own `/plan` command is different from Spec Kit's `/speckit-plan`. Run them in this order, reviewing the generated file after each one before moving on:

1. **`/speckit-constitution`**: set project rules once (coding standards, testing, security, allowed tech). Example: `/speckit-constitution Use TypeScript strict mode, every feature needs unit tests, no secrets in code, follow our REST naming conventions.`
2. **`/speckit-specify`**: describe **what** to build and **why**, with no tech choices. A feature branch and `specs/001-.../spec.md` are created. Example: `/speckit-specify Build an employee leave tracker. Employees request leave with dates and type; managers approve or reject; employees see their remaining balance.`
3. **`/speckit-clarify`** (recommended): Antigravity asks about gaps in the spec; answer them so the spec is complete.
4. **`/speckit-plan`**: describe **how**: stack, architecture, integrations. Produces `plan.md` and supporting design files. Example: `/speckit-plan React + Vite frontend, Node.js Express API, PostgreSQL, Google SSO login.`
5. **`/speckit-tasks`**: breaks the plan into an ordered `tasks.md` checklist.
6. **`/speckit-analyze`** (optional): checks spec, plan and tasks for gaps and contradictions.
7. **`/speckit-implement`**: Antigravity writes the code task by task. It asks permission before editing files or running commands; review each change, run tests and commit.

For each new feature, start again at step 2; the constitution stays in place. The review between steps is the core of SDD: fixing a wrong assumption in `spec.md` costs minutes, fixing it in code costs hours.

## Troubleshooting on Windows

| Problem | Fix |
| --- | --- |
| `agy`, `specify` or `uv` "is not recognized" | Reopen PowerShell. Still failing: check `%USERPROFILE%\.local\bin` is in your user PATH (Start → "Edit environment variables for your account"), or rerun the installer. |
| "running scripts is disabled on this system" | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, or install `agy` with the Command Prompt (cmd) command in Step 2. |
| "Sorry, this account is ineligible to use Antigravity" | Your admin has blocked personal-style access for work accounts. Use `/logout`, then sign in through **Use a Google Cloud project** (Option A or B). |
| Only a small starter quota on your work account | You signed in through plain "Continue with Google". `/logout` and sign in through **Use a Google Cloud project**. |
| "Contact your administrator to set up Gemini Enterprise" or "No Gemini Enterprise license was found" | Admin must turn on **AI developer tools** (Gemini Enterprise → Settings), distribute a license to that project and region, and grant you the Gemini Enterprise User role. If your company uses Option B, pick **Pay As You Go** instead. |
| Permission denied with Option B | Check that billing is active, the Agent Platform API is enabled and you hold the Agent Platform User role on that project. |
| `/speckit-` skills don't appear | Start `agy` from the project root (the folder containing `.agents/`). Check `.agents/skills/` exists; if not, rerun `specify integration install agy`. Update `agy` to the latest version. |
| `/speckit.specify` does nothing | That's the Gemini CLI spelling. In Antigravity, use hyphens: `/speckit-specify`. |
| Old `gemini` command now fails or shows free-tier limits | Expected since 18 June 2026 unless your company has a Gemini Code Assist license. Move to `agy` (Step 2). |
| Network or certificate errors behind a corporate proxy | Set `setx HTTPS_PROXY "http://proxy:port"`, reopen PowerShell, and ask IT to allow the Google endpoints listed in the network section of [Antigravity in Gemini Enterprise](https://antigravity.google/docs/enterprise/). |

## Quick reference checklist

- [ ] Got the approved option (A or B), project ID, region and roles from IT
- [ ] Installed Python, Git and uv; set execution policy to RemoteSigned
- [ ] Installed Antigravity CLI: `irm https://antigravity.google/cli/install.ps1 | iex`
- [ ] Signed in via **Use a Google Cloud project → Continue with Google Cloud** with your work email and got a reply to a test prompt
- [ ] Installed Spec Kit at the latest tag and `specify check` passes
- [ ] Ran `specify init --here --integration agy --script ps` in the project
- [ ] Started `agy` in the project root and created the constitution with `/speckit-constitution`
- [ ] First feature: specify → clarify → plan → tasks → analyze → implement

## Sources

Checked on 18 Nov 2026. Google changes this tooling often, so recheck these pages if a screen looks different.

- [Antigravity for Google Workspace accounts](https://antigravity.google/docs/workspace/): sign-in options and Option B steps
- [Antigravity in Gemini Enterprise](https://antigravity.google/docs/enterprise/): Option A, admin setup, network endpoints
- [Antigravity CLI on GitHub](https://github.com/google-antigravity/antigravity-cli): Windows install commands and sign-in
- [Transitioning Gemini CLI to Antigravity CLI](https://github.com/google-gemini/gemini-cli/discussions/27274): Google's announcement and timeline
- [Spec Kit Antigravity integration](https://github.github.io/spec-kit/integrations/agy.html): `agy` key and `/speckit-` skills
- [Spec Kit releases](https://github.com/github/spec-kit/releases): current version tag
