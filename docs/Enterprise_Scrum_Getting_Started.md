# Getting started: Enterprise Scrum × Scrum@Scale for the EntropyLab network

A starting point for organising the loose network of people (and now
agents) who work on EntropyLab, using the parts of Mike Beedle's
**Enterprise Scrum** and Jeff Sutherland's **Scrum@Scale** that fit a
volunteer, asynchronous, security-critical open-source project, and
dropping the parts that do not. It ends with a playbook for onboarding an
AI agent as a teammate, adapted from Sutherland's own write-up of doing
exactly that in Jira, translated to GitHub.

This document describes a *proposal to run an experiment*, not a mandate.
Everything in it is meant to be inspected and adapted at the first
retrospective. Nothing here changes `CONTRIBUTING.md`; that file remains
the rulebook for what is mergeable.

---

## 0. Why bother, and why now

The lab is already operating at a scale most companies would call "a
program". Measured on this clone of the upstream history at the time of
writing:

| Signal                                       | Value        |
| -------------------------------------------- | ------------ |
| Distinct commit authors, last 90 days        | 26           |
| Non-bot commits, last 30 days                | 326          |
| Pull requests merged into `rock`, best week  | 88           |
| Test files in `test/`                        | 99           |

That is throughput. The thing a loose network does *not* get for free is
coordination: knowing what is in flight, who is waiting on whom, which
review is blocking a release, and how a newcomer (human or agent) finds
work they can finish. Scrum's whole purpose is to make that state visible
at a fixed cadence with the least possible ceremony. Scrum@Scale's own
design constraint is the right one here: **Minimum Viable Bureaucracy**,
"the least amount of governing bodies and processes needed ... without
impeding the delivery of customer value."

The second reason is agents. Sutherland's current message is that
disciplined Scrum becomes *more* important, not less, once agents move
from copilots to participants in the flow of work. His team onboarded an
agent onto a real Jira board in mid-2026 and wrote up what broke. This
fork is doing the same thing on GitHub, and the ticket style already used
in issues #1 and #3 here is a good match for his pattern. Section 6 makes
that explicit.

---

## 1. The two frameworks in one page each

### 1.1 Enterprise Scrum (Beedle): Scrum as a parameterised template

Mike Beedle, a co-author of the Agile Manifesto, generalised Scrum so it
could run *any* business process, not just software delivery. His own
summary is "abstraction, generalization, and parameterization". The
vocabulary is deliberately neutral:

| Scrum term       | Enterprise Scrum term | Point of the rename                              |
| ---------------- | --------------------- | ------------------------------------------------ |
| Product Owner    | **Business Owner**    | The thing owned may be a release, an audit, a doc set, a budget |
| Scrum Master     | **Coach**             | Serves the process, does not manage the people   |
| Product Backlog  | **Value List**        | Ordered by value; items need not be features     |
| Sprint           | **Cycle**             | Length is a parameter, chosen per team and per level |
| —                | **Canvas**            | A one-page description of a team's configuration |

The idea that matters for us is **parameterisation**: each team fills in
its own canvas (what we deliver, to whom, how long a cycle is, what "done"
means, which metric we watch) instead of inheriting one company-wide
setting. That is exactly what a network of independent contributors needs.
The upstream maintainers, a translation pipeline, a fork where an agent is
being trained, and a security-audit effort do not share a cycle length or
a definition of value, and should not be forced to.

Beedle died in 2018 and the framework has not evolved since. Treat it as a
lens, not a product.

### 1.2 Scrum@Scale (Sutherland): the coordination skeleton

Scrum@Scale keeps ordinary Scrum inside each team and adds the fewest
structures needed to coordinate many teams. Its shape:

- **Two cycles.** The *Scrum Master cycle* ("how": impediments, cross-team
  coordination, delivery, process feedback) and the *Product Owner cycle*
  ("what": vision, backlog decomposition and refinement, release planning,
  product feedback). They touch at three shared components: team-level
  process, product & release feedback, and metrics & transparency.
- **Scrum of Scrums (SoS).** Four or five teams that must integrate their
  work meet as one "team of teams". Its daily event is the **Scaled Daily
  Scrum**; its facilitator is the **Scrum of Scrums Master**.
- **MetaScrum.** The Product Owners of those teams, led by a **Chief
  Product Owner**, keep one aligned backlog across the group.
- **Executive Action Team (EAT).** Owns the *how* at the top: removes the
  impediments that no team can remove itself, and owns the transformation
  itself.
- **Executive MetaScrum (EMS).** Owns the *what* at the top: sets vision
  and strategic priorities, arbitrates between backlogs.

The guide says something that makes the framework usable for us at all:
"In very small organizations or implementations, the Executive Action Team
members and the attendees of the Executive MetaScrum event may consist
entirely of the same people." We are that case.

### 1.3 What each contributes to the mashup

| Need in the lab                                       | Take from                |
| ----------------------------------------------------- | ------------------------ |
| Skeleton for coordinating several efforts              | Scrum@Scale              |
| Each effort chooses its own cadence, metric, "done"    | Enterprise Scrum         |
| Backlog that holds non-code work (docs, audits, i18n)  | Enterprise Scrum (Value List) |
| One place where blocking impediments go                | Scrum@Scale (EAT)        |
| Explicit permission to collapse layers when small      | Scrum@Scale (MVB)        |
| Agents as teammates with tickets, not as chatbots      | Sutherland's 2026 work   |

What we drop: story points and velocity as a planning currency (volunteer
capacity is not forecastable, and it invites gaming), fixed sprint lengths
across all teams, and any role that exists only to attend meetings.

---

## 2. The lab's shape

### 2.1 Teams (Enterprise Scrum "canvases")

A *team* here is any group of people, one person, or one person plus an
agent, that owns a Value List and delivers integrated increments. Candidate
canvases for the current network, each to be confirmed by the people
actually doing the work:

1. **Core / upstream.** The maintainers who merge to `rock` on
   `OogaBoogaX/entropylab`. Delivers the release artefact
   (`entropylab.html`) and owns `CONTRIBUTING.md`.
2. **Feature squads, ad hoc.** Whoever is carrying a multi-PR feature (the
   recent vanity grinder, PSBT editor, BIP-352, i18n consolidation are all
   this shape). They form for the feature and dissolve on merge.
3. **Translation pipeline.** Mostly automated (see
   `docs/Translation_Automation_Setup.md`); its human work is operator
   setup and gate policy.
4. **Verification & security.** Reproducible-build checks, attestation,
   fuzzing (`fuzzing/`), and advisory handling under `SECURITY.md`.
5. **Fork sandboxes.** A personal fork where an agent is onboarded (this
   repository). Its Value List is training tasks and hardening of the
   agent workflow, and it never targets upstream without a human.

Each canvas answers, in a pinned issue or a short markdown file:

```
Team:           <name>
Business Owner: <GitHub handle>            # orders the Value List
Coach:          <GitHub handle or "none">  # facilitates, removes impediments
Delivers:       <what an increment is>     # e.g. "merged PRs on rock"
Cycle:          <1 week | 2 weeks | per release | continuous>
Done means:     <link to DoD, see §5>
Metric:         <one number, see §4>
Sync:           <where the async daily lives, see §3>
```

### 2.2 Scaled roles, mapped to people who already exist

| Scrum@Scale role         | Enterprise Scrum name | In the lab                                                  |
| ------------------------ | --------------------- | ----------------------------------------------------------- |
| Product Owner (per team) | Business Owner        | Whoever orders that team's Value List                       |
| Chief Product Owner      | Business Owner of the lab | The upstream maintainer who decides what ships in `rock` |
| Scrum Master (per team)  | Coach                 | Optional per team; one person may coach several            |
| Scrum of Scrums Master   | Lab Coach             | One volunteer who runs the sync and the retro               |
| Executive MetaScrum      | Lab Council ("what")  | Chief Product Owner + team Business Owners                  |
| Executive Action Team    | Lab Council ("how")   | The same people, with a different agenda item              |

The **Lab Council** is one meeting (or one async thread) with two halves,
because the guide explicitly permits EAT and EMS to be the same people.
Split them only when the "what" and "how" conversations start crowding
each other out.

An agent never holds a role. It is a **team member** assigned tickets,
and only inside a team whose Business Owner has accepted it. That mirrors
Sutherland's note in the third article of his series: Scrum leaves task
distribution to the Developers, and an agent that assigns work is
executing a Product Owner's explicit instruction, not running the team.

---

## 3. Cadence and events (all asynchronous by default)

The network spans time zones and nobody is paid to attend meetings, so
every event has an async form that is the default, and a synchronous form
that is optional.

| Scrum@Scale event      | Lab form                                                                                              | Cadence (default parameter) |
| ---------------------- | ------------------------------------------------------------------------------------------------------ | --------------------------- |
| Sprint / Cycle         | A milestone on GitHub with a one-sentence goal                                                          | 1 week for core; teams choose |
| Daily Scrum            | A comment on the team's open **Cycle sync** issue: done / next / blocked                               | When you have something; at least once per cycle |
| Scaled Daily Scrum     | The Lab Coach reads all team syncs and posts one roll-up on the lab-wide sync issue, listing cross-team blockers | Twice a week  |
| Sprint Review          | The release, or for non-release cycles, the merged PR list on the milestone                            | Per cycle                   |
| Retrospective          | A **Retro** issue with three headings: keep / change / one experiment                                   | Per cycle for each team     |
| Scaled Retrospective   | The Lab Coach merges team retros into one list of experiments; the Council picks at most one lab-wide | Monthly                     |
| MetaScrum / Council    | One issue per month: priorities for `rock`, impediments nobody could clear, decisions to record         | Monthly                     |

The `.github/ISSUE_TEMPLATE/` directory now contains forms for **Backlog
item**, **Agent task**, and **Cycle sync / retro** so each of these is one
click. Any tool that lives outside GitHub (a chat room, a call) is fine,
but the record of decisions lives in the repo, where the agent can read it.

Time-boxes: a synchronous Scaled Daily Scrum is 15 minutes, a Council call
is 45. Anything longer is a sign that a decision should have been an
issue.

---

## 4. Metrics and transparency

Scrum@Scale asks for metrics that "provide the appropriate context with
which to make data-driven decisions". For volunteers, pick numbers that
are observable from GitHub without anyone filling in a timesheet, and that
point at *flow*, not individual output:

| Metric                                 | Why it matters                                             | Where to read it                 |
| -------------------------------------- | ---------------------------------------------------------- | -------------------------------- |
| Cycle time: issue opened → PR merged   | The one number that captures coordination cost              | Issue and PR timestamps          |
| Open PRs older than 7 days             | Review, not coding, is usually the bottleneck               | PR list sorted by age            |
| CI green rate on `rock`                | Red base branch stalls everyone                             | Actions history                  |
| Merged PRs per cycle                   | Throughput, without pretending it is a forecast             | Milestone                        |
| Impediments open > 1 cycle             | The Council's own to-do list                                | `impediment` label               |
| Team happiness, 1–5, once per cycle    | Sutherland's leading indicator; for volunteers it predicts churn | The retro issue              |

Do not track velocity in story points. Do not compare individuals.
Publish the numbers in the monthly Council issue so the whole network can
see them; transparency is the mechanism, not the report.

---

## 5. Definition of Done

`CONTRIBUTING.md` already ends with a six-question sanity check. It *is*
the lab's Definition of Done, and it applies to every team and every
teammate, human or agent:

1. Does this keep EntropyLab a calculator that never invents entropy?
2. Does the app stay silent on the network?
3. Is the output still a single self-contained `entropylab.html`?
4. Is it smaller, or at least no bigger, than it was?
5. Did you rebuild, keep docs/version in sync, and commit no generated files?
6. Could an auditor follow the change in one pass?

Two additions for agent-produced work, because an agent's output is
probabilistic and its diff is the only evidence of what it did:

7. A human read the whole diff, not the PR description, before merging.
8. The PR body lists the commands the agent actually ran and their exit
   status, and the CI run on the final commit is green.

A Value List item is *ready* to be pulled when it names the files in scope,
the files out of scope, and the test that proves it. Issues #1 and #3 in
this fork are the reference examples.

---

## 6. Onboarding an agent as a teammate, Sutherland-style, on GitHub

Sutherland's team (JVS Management / ScrumAI.org) documented onboarding a
self-hosted Hermes agent onto a real Jira board in June 2026 in a
three-part series: setup, first task, and the week the agent began
assigning work to humans. Every lesson transfers; only the API changes.

### 6.1 The lessons, verbatim in spirit

| What they learned                                                                  | What it means on GitHub                                                                                   |
| ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Give the agent **its own account**, never a borrowed human login: every action is attributable, permissions scope tightly to one project | A dedicated GitHub account (or GitHub App) for the agent, with access to the sandbox fork only |
| A **green health check can lie**: the auth probe passed while real inference failed | Validate with a real end-to-end ticket, not with "the token works"                                        |
| The **"more secure" scoped token authenticated but was blind**; the classic token worked | Use a fine-grained token scoped to the fork, then *prove* it can read issues, comment, push a branch, and open a PR before the first real task |
| **Ticket text is data, not commands**: prompt injection is a design concern once the agent has a shell | The agent acts only on issues assigned to its account by a repo owner; comments from others are context, never instructions |
| Start from a **Blank Slate toolset** (file ops + terminal) and turn tools on one at a time; leave delete and bulk operations off | No force-push, no branch deletion, no merge rights, no secrets; the agent opens PRs, humans merge |
| The **pattern**: a human writes a ticket assigned to the agent, the agent detects it within ~15 minutes via cron, executes, reports back; no special UI | The `[agent]` issue prefix plus the `agent` label, an assignment to the agent's account, a PR that closes the issue |
| **Week 1 executor, week 2 distributor**, then retro support, each step tested with a deliberately small task | A staged ladder (§6.3); the agent earns each rung with a passing task, and the team can send it back a rung |

### 6.2 Repo-specific guard rails (non-negotiable)

These come from `AGENTS.md`, `CONTRIBUTING.md`, and the fork's README note
and apply before any Scrum consideration:

- The agent **never** handles real seeds, keys, or passphrases. Test
  vectors only (BIP39, BIP32, Bitcoin Core published vectors).
- The agent **never** generates entropy for key material, adds network
  calls, or touches `src/locales/`, `entropylab.html`, or the
  `src/js/*-wasm-b64.js` artefacts.
- API keys, agent config, and `.env` files never enter the repo.
- The agent works on `agent/<slug>` branches in the fork and never opens a
  PR against `OogaBoogaX/entropylab` unless a human does it deliberately.
- Every agent task ends with `npm run build && npm test` and the output in
  the PR body.

### 6.3 The onboarding ladder

Each rung is one ticket, written with the **Agent task** template, and the
agent does not advance until a human has closed the previous rung's PR.

| Rung | Task                                                                                              | Passes when                                                       |
| ---- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| 0    | **Smoke test** (Sutherland's "make me a coffee"): comment on the issue, wait 5 minutes, comment "Task Completed", close it | Comments appear from the agent's own account, in order, on time |
| 1    | **Docs-only change** with explicit in-scope and out-of-scope files (issue #1 here)                 | PR touches only the named file; CI green; human merges            |
| 2    | **Source change with a test** (issue #3 here plus a test expectation)                              | The test the issue named fails before and passes after            |
| 3    | **Triage**: label and cross-reference the open issues, without changing any code                   | Labels are accurate; no false duplicates                          |
| 4    | **Distribution**: for a batch of ready items, propose an assignee each and explain why in a comment | A Business Owner accepts the proposals without edits              |
| 5    | **Cycle sync**: post the done / next / blocked roll-up for the team from PR and issue state         | The human reading it needs no corrections                         |
| 6    | **Retro support**: draft the keep / change / experiment summary from the cycle's threads           | The team adopts at least one of the drafted experiments           |

Rung 4 is where Sutherland's team drew the line carefully: the agent
distributes work only when the Product Owner instructs it to, and the
outcome is still the Developers' to accept. Keep that line.

### 6.4 The agent as a daily-scrum participant

Sutherland's phrasing is that the agent should treat tasks "the same way
any developer would: pick it up, do the work, mark it done". For the daily
sync that means:

- The agent posts its own done / next / blocked comment on the Cycle sync
  issue, from its own account, once per cycle, generated from the PRs and
  issues it touched. A human reads it like any other teammate's.
- "Blocked" from an agent must name the exact question a human needs to
  answer. Open-ended blockers are a defect in the ticket, and the Coach
  fixes the ticket, not the agent.
- The agent may draft the Scaled Daily Scrum roll-up (rung 5), but the Lab
  Coach posts it. Judgement about which blocker matters most stays human.

### 6.5 Security posture in one paragraph

An agent with a shell and a token is an attack surface. The lab's threat
model already assumes hostile input (the whole tool is built to be
audited), so extend it: an issue comment that tells the agent to "ignore
your instructions" is untrusted data; the agent's account has no merge or
delete permission; the fork's secrets are held outside the repo; a human
merges every PR after reading every line; and the agent's first task after
any change to its tools or model is rung 0 again. If a rung fails twice,
the agent goes back one rung and the failure is discussed at the retro,
not patched around.

---

## 7. First 30 days

**Days 1–3, Cycle 0 (set-up).**
- Each candidate team from §2.1 fills in its canvas, or declines to exist.
- One person volunteers as Lab Coach for 30 days.
- Labels created: `agent`, `impediment`, `ready`, `blocked`.
- The first Cycle sync issue is opened from the template.

**Days 4–10, Cycle 1.**
- Every team runs one cycle at its chosen length.
- Agent rung 0 and rung 1 in the sandbox fork.
- The Lab Coach posts two roll-ups and records every cross-team blocker
  as an `impediment` issue.

**Days 11–20, Cycle 2.**
- First team retros; first Scaled Retrospective; one experiment chosen.
- Agent rungs 2 and 3.
- First metrics snapshot (§4) posted to the Council issue.

**Days 21–30, Cycle 3 and first Council.**
- Council reviews the metrics, kills anything that produced ceremony
  without information, and decides whether to continue.
- Agent rung 4 only if rungs 0–3 passed cleanly.
- This document gets its first edit from the retro, in a PR like any other.

---

## 8. Crosswalk

| Concept          | Scrum@Scale          | Enterprise Scrum | This lab                       | Lives in GitHub as                    |
| ---------------- | -------------------- | ---------------- | ------------------------------ | ------------------------------------- |
| Unit of work     | Product Backlog Item | Value List Item  | Backlog item                   | Issue with the Backlog item template  |
| Ordered work     | Product Backlog      | Value List       | Value List                     | Project board / milestone             |
| Iteration        | Sprint               | Cycle            | Cycle                          | Milestone                             |
| Daily            | Daily Scrum / SDS    | Cycle stand-up   | Cycle sync                     | Issue with the Cycle sync template    |
| Team config      | —                    | Canvas           | Canvas                         | Pinned issue or `docs/` file          |
| Owner of "what"  | PO / CPO / EMS       | Business Owner   | Business Owner / Council       | Milestone owner                       |
| Owner of "how"   | SM / SoSM / EAT      | Coach            | Coach / Lab Coach / Council    | `impediment` label owner              |
| Done             | Definition of Done   | Definition of Done | The six questions + two agent lines | `CONTRIBUTING.md` §"A final sanity check" |
| Agent            | —                    | —                | Team member on a ladder        | Own account, `agent` label, `[agent]` issues |

---

## 9. Sources

- Scrum@Scale Guide (online):
  https://www.scrumatscale.com/scrum-at-scale-guide-online/
- Mike Beedle, *Enterprise Scrum: Scaling Scrum to the Executive Level*
  (paper) and the Enterprise Scrum introduction; summary at
  https://ame3.ai/enterprise-scrum/
- Jeff Sutherland / JVS Management, agent onboarding series (2026):
  part one, *Onboarding an AI Agent as a Real Teammate*,
  https://jvsmanagement.com/onboarding-an-ai-agent/ ;
  part two, *AI Agent in Jira: My First Day on the Job*,
  https://jvsmanagement.com/ai-agent-in-jira/ ;
  part three, *AI Assigning Work to Humans*,
  https://jvsmanagement.com/ai-assigning-work-to-humans/
- Scrum Day Madison 2026, Sutherland session abstract, *When the Machine
  Can Think: Scrum for AI Agent Teams*:
  https://www.scrumday.org/jeff-sutherland
- This repository: `CONTRIBUTING.md`, `AGENTS.md`, `SECURITY.md`, issues
  #1 and #3.
