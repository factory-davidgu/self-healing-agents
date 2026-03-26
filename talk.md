### Opening image

Background image: opening-bg.jpg

---

### At Factory, we make Droid

A multi-model coding agent that lives on your terminal, in your IDE, in your Slack, in your remote machines.

---

Let your agents
move fast and break things
then let them fix it themselves

- First shows: move fast and break things
- Then "Let your agents" and "then let them fix it themselves" fade in

---

### Self-healing agents

How Droid closes its own feedback loop

David Gu - Member of Technical Staff @ Factory

---

And as every piece of software out there...
we have bugs

---

### Bug fix runbook

1. User files bug report
2. Eng triages logs, metrics, stacktraces
3. Reproduce the bug
4. Define a regression test
5. Fix the bug and submit a PR

- The goal of a good engineering system is to **reduce every bottleneck**

**Key insight:** Most of this loop is *mechanical*. Query logs, correlate, reproduce, patch.

---

### How can we navigate through this loop faster?

- Agents should use *the same systems your engineers use*
- Agents write code, but that's *half of the problem*
- We will focus on these steps: *observability, reproduction, and testing*

---

### Quick primer on Skills

---

### Skills = procedural memory for agents

- A `SKILL.md` markdown file with frontmatter metadata (name, description, etc)
- Plus scripts that the skill may invoke
- Composable: `/bug-report` -> `/axiom-query` -> `/droid-control` -> ...
- Not just a prompt template. It's a **runbook** the agent follows.

```yaml
---
name: bug-report
description: Analyze CLI bug reports from S3.
  Query Axiom for logs, metrics.
  Parses session logs and groups errors by root cause.
---
# Daily CLI Bug Report Analysis
## Workflow
1. Download and extract reports from S3     + script
2. Query Axiom for logs and metrics         -> /axiom-query
3. Parse session logs, extract errors       + schemas + scripts
4. Reproduce in sandboxed TUI               -> /droid-control
5. Write an e2e regression test             -> /droid-control
```

---

### The self-healing toolkit

Bug Reports (S3) -> `/bug-report` -> bug report id
Queryable Logs   -> `/axiom-query` -> axiom cli/mcp
Sandboxed TUI    -> `/droid-control` -> tuistory
E2E Tests        -> `/cli-e2e-testing`
PR Pipeline      -> `/create-pr` + `/follow-up-on-pr`

~40 skills in our repo

---

### Step 1: bug reporting

- Make it **really easy** for users to report issues directly from the product

`/bug should be able to fuzzy search models in the /models menu`

- Upload bug bundle with relevant metrics
  - droid version
  - terminal emulator
  - platform + OS
  - session logs
- Give them a **UUID** to track and follow up if needed

---

### Step 1: bug reporting (demo)

[VIDEO: bug-report.mp4]

---

### Step 2: triage

`/bug-report` -> `/axiom-query` + code navigation

- `/bug-report` downloads from S3, extracts errors
- `/axiom-query` gathers metrics and logs

**Key technique:** The agent runs the *same queries your oncall engineers run*. It needs access to your observability platform's query language.

---

### Step 2: triage (demo)

[VIDEO: axiom.mp4]

---

### Step 3: reproduction (live demo or video)

`/droid-control` - tuistory

- **tuistory**: a Playwright-like framework for terminal UIs. Launch, type, press keys, wait, snapshot.
- Droid launches an instance of itself in a sandboxed session, triggers the bug, captures evidence

```bash
tuistory launch "droid" -s repro --cols 120 --rows 40
tuistory -s repro wait ">"
tuistory -s repro type "/model"
tuistory -s repro press enter
tuistory -s repro snapshot

tuistory -s repro type "opus"
tuistory -s repro snapshot

tuistory -s repro type "4"
tuistory -s repro snapshot
```

### Step 3: reproduction (demo)

[VIDEO: tuistory.mp4]

---

### Step 4: e2e regression test

`/cli-e2e-testing`

- Framework: `@microsoft/tui-test`, a Playwright-like framework for terminal UIs.
- Each test gets an **isolated workspace** (project dir + simulated user home).
- TUI traces are automatically recorded; Droid can inspect terminal state at any point in time.

```typescript
test('fuzzy matches "opus4.6" to "Opus 4.6"', async ({ terminal }) => {
  await expect(
    terminal.getByText('>', { full: false, strict: false })
  ).toBeVisible({ timeout: 15000 });

  // Open model selector via /model command
  terminal.write('/model\r');

  // Wait for model selector to appear
  await expect(
    terminal.getByText('Factory Provided Models', {
      full: false, strict: false,
    })
  ).toBeVisible({ timeout: 10000 });

  // Type fuzzy query without space
  terminal.write('opus4.6');

  // Should show Opus 4.6 as a match (fuzzy matching)
  await expect(
    terminal.getByText('Opus 4.6', { full: false, strict: false })
  ).toBeVisible({ timeout: 3000 });
});
```

---

### Step 5: fix + ship

`/create-pr` - `/follow-up-on-pr`

- `/create-pr`: lint + commit + branch + pr
- `/follow-up-on-pr`: rebases, addresses feedback, fixes CI

---

### Full demo: bug to PR

[VIDEO: bug-to-pr.mp4]

---

[SCREENSHOT: screenshot-bug-to-pr.png]

---

### Extra notes

1. **Skills are often not picked up automatically.** Be explicit if you can (for now)
2. **MCPs work too.** Just be careful of context bloating
3. **Start building skills locally** and then share them with your team via a plugin marketplace or by committing them to a specific repo

---

### Beyond bug fixes: other specialized skills

The same pattern applies to other engineering workflows:

- `/memory-leak-analysis`: Collects V8 heap snapshots over time, classifies objects by growth pattern (only grows = leak, oscillates = healthy GC)
- `/oncall-alert-triage`: Groups and deduplicates alerts, runs the same investigation queries an oncall engineer would, classifies severity, detects false positives
- `/automated-qa`: Browser-driven smoke tests against production: OTP login via AgentMail API, new user signup, voucher redemption, cloud session creation

---

### How you can do this

1. **Write your runbooks as machine-readable skills.** Start with your oncall playbook. If a human follows steps, an agent can too.
2. **Give the agent read-only query access to your observability stack.** Axiom, Datadog, Grafana. Show some example queries too.
3. **Build a sandboxed reproduction environment.** Docker, tuistory, Playwright. The agent needs to run your product and observe it fail.
4. **Make regression tests the success criterion.** Not "does the code look right" but "does the test that previously failed now pass."
5. **Instrument your code as much as possible.** With logs and metrics, and log them into queryable stores.

---

### Safety first

Be careful of how you inject external dynamic content

- **Clinejection (2026):** a malicious GitHub issue title prompt-injected an AI triage bot, compromising ~4,000 developer machines
- **Sanitize external inputs.** Issue titles, PR descriptions, user comments. All untrusted content that agents ingest
- **Principle of least privilege.** Read-only where possible, limit scope
- **Sandboxed environments with fine-grained controls.** Restricted network access and filesystem

---

### What's next?

Automating the process.
High fidelity simulations.
Becoming a **Dark Factory**

---

### Q&A

We are hiring. factory.ai/careers

David Gu - Member of Technical Staff @ Factory

@agent_wrap
