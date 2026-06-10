## Project Management Rules (Delivery Manager)

> **Scope:** This section governs how AI agents (Claude Code, GitHub Copilot, and any other automated tool) must behave with respect to project lifecycle artifacts, validation gates, and change management in the `novatech-assistant` repository.
> These rules are **binding for all agents**. When in doubt, stop and surface the ambiguity to a human before proceeding.

---

### 1. Spec Lifecycle — Core Rules

#### 1.1 Artifact Locations

- Spec files MUST live exclusively under `/docs/specs/<module-id>-<module-name>/`.
- Each module directory MUST contain exactly three files: `requirements.md`, `plan.md`, `tasks.md`.
- Architectural decision records MUST live under `/docs/adr/` and follow the naming pattern `ADR-NNNN-<slug>.md`.
- DO NOT create spec files outside the paths above.
- DO NOT create versioned copies such as `requirements-v2.md` or `tasks-final.md`. Version control is handled exclusively by Git history.

#### 1.2 Module Index

| Module ID | Module Name | Spec Path |
|---|---|---|
| `01` | pipeline-ingestao | `/docs/specs/01-pipeline-ingestao/` |
| `02` | api-busca | `/docs/specs/02-api-busca/` |
| `03` | api-feedback | `/docs/specs/03-api-feedback/` |
| `04` | bot-teams | `/docs/specs/04-bot-teams/` |
| `05` | painel-web | `/docs/specs/05-painel-web/` |

#### 1.3 Artifact Ownership

- `requirements.md` is owned by the **Product Specialist**. Agents MUST NOT modify this file without an approved RFC referencing it.
- `plan.md` is owned by the **Tech Lead**. Agents MUST NOT generate or overwrite `plan.md` unless an approved `requirements.md` exists for the same module and Gate 1 has been recorded as passed.
- `tasks.md` is owned by the **Dev** (with Copilot assistance). Agents MUST NOT populate `tasks.md` unless an approved `plan.md` exists for the same module and Gate 2 has been recorded as passed.

---

### 2. Validation Gates — Enforcement Rules

Agents MUST NOT advance work past a gate boundary without a human approval record in the commit history or Azure DevOps. The gate record is the **only** authorization signal agents should trust.

#### Gate 1 — `requirements.md` → `plan.md`

```
Trigger   : requirements.md is present and marked approved.
Approvers : Product Specialist (primary) + Tech Lead (co-sign).
Signal    : Git commit message matching pattern [spec][NN-*] approve: * gate-1 *
```

- MUST check for Gate 1 approval signal before generating or editing `plan.md`.
- DO NOT generate `plan.md` if `requirements.md` is in status `draft` or `in-review`.
- SHOULD surface a warning if `requirements.md` references any of the 12 documents flagged as pending Compliance resolution without explicit handling instructions.

#### Gate 2 — `tasks.md` → Implementation

```
Trigger   : plan.md is present and marked approved.
Approver  : Tech Lead.
Signal    : Git commit message matching pattern [spec][NN-*] approve: * gate-2 *
```

- MUST check for Gate 2 approval signal before writing any implementation code for a module.
- DO NOT create source files under `src/`, `functions/`, `bot/`, or `web/` for a module whose `tasks.md` lacks a Gate 2 approval signal.
- MUST read the full `tasks.md` at the start of every implementation session. DO NOT rely on context from a previous session that predates the last commit to `tasks.md`.
- Each task in `tasks.md` MUST specify: target language (`TypeScript`, `React`, or `Bicep`), the acceptance criterion, and the Azure service it touches (if any).

#### Gate 3 — Implementation → Merge

```
Trigger   : A pull request or merge request is opened.
Approver  : Tech Lead (mandatory) + 1 peer Dev (mandatory).
Signal    : PR approval recorded in repository history before merge.
```

- MUST NOT merge code that has not received at least one human approval.
- Before flagging code as ready for review, agents MUST self-verify the following checklist:

```
[ ] System prompt template in this module does not exceed 4,000 tokens.
[ ] Azure AI Search query returns at most 5 chunks (top_n <= 5).
[ ] Conversation history window is capped at 3 turns in session management code.
[ ] Contradictory-document resolution logic is present: results ordered by
    the `vigencia` metadata field before context assembly.
[ ] No direct reference to documents flagged `status: obsoleto`.
[ ] All new code is in TypeScript (backend/bot), React (frontend),
    or Bicep (infrastructure). No other languages or frameworks.
[ ] No import of a library not already present in package.json or
    not listed in the approved Azure SDK set.
```

- If any item above fails, the agent MUST stop, annotate the failure with the expected value vs. the found value, and request human intervention. DO NOT auto-fix token budget violations by silently truncating the system prompt.

#### Gate 4 — Tests → Deploy

```
Trigger   : Implementation complete; test suite is present.
Approvers : QA (quality sign-off) + Tech Lead (technical ratification).
Signal    : Gate 4 approval recorded in Azure DevOps linked to the deploy item.
```

- MUST NOT trigger or approve a deploy sequence without Gate 4 sign-off.
- When generating test files, agents MUST include at minimum:
  - One test covering the contradictory-document scenario (two docs with the same topic and different `vigencia` values; assert the response references the more recent one).
  - One boundary test sending a context payload exceeding the token budget; assert graceful handling (explicit error or controlled truncation).
  - One integration test asserting that the Azure AI Search call returns no more than 5 chunks.
- DO NOT use any of the 12 documents with pending Compliance contradictions as test oracles (ground-truth expected responses). Their resolved content is not guaranteed.
- Bicep templates MUST be validated against a staging environment, not only linted. DO NOT mark infrastructure tasks as complete based on `az bicep build` success alone.

---

### 3. Change Management — RFC Protocol

#### 3.1 When an RFC is Required

An RFC (Request for Change) is required whenever a spec file that has already passed its gate needs to be modified. This applies to **all agents**: if an agent identifies a necessary change to an approved spec, it MUST open an RFC item in Azure DevOps and MUST NOT modify the spec file directly.

```
RFC is required when:
  - requirements.md is modified after Gate 1 has been recorded.
  - plan.md is modified after Gate 2 has been recorded.
  - tasks.md is modified after implementation has started (any source file exists for the module).

RFC is NOT required when:
  - Fixing typos or prose clarity with zero impact on acceptance criteria or
    technical parameters (commit message: [spec] fix: <description>).
  - Adding a new module spec from scratch (no gate has been passed yet).
```

#### 3.2 RFC Identifier

- RFC identifiers follow the pattern `RFC-NNN` (zero-padded to 3 digits).
- The RFC number MUST be included in every commit message that applies the change: `[spec][NN-module] rfc: RFC-NNN <summary>`.

#### 3.3 Agent Behavior During an Open RFC

- MUST NOT continue generating code for tasks classified as `INVALIDATED` by the RFC.
- MUST treat `tasks.md` as read-only until the RFC is approved and the file is updated with the new task set.
- SHOULD surface a warning to the developer if the current working file references a task marked `[CANCELADA - RFC-NNN]`.

#### 3.4 Task Reset After RFC Approval

When an RFC is approved and `tasks.md` is updated, agents MUST follow this reset sequence:

```
1. STOP all in-progress work on impacted tasks.
2. A human MUST commit the revert of invalidated code before the agent resumes:
     git commit -m "revert: remove code from tasks invalidated by RFC-NNN"
3. READ the updated tasks.md in full before starting any new generation.
4. DO NOT reuse context or code snippets from the pre-RFC session.
5. Treat the updated tasks.md as the sole source of truth for this module.
```

---

### 4. Commit Message Standards for Agents

All commits authored or co-authored by an AI agent MUST follow these patterns:

```
Spec lifecycle:
  [spec][NN-module] create: <artifact> initial version
  [spec][NN-module] update: <artifact> <summary> (requires RFC if post-gate)
  [spec][NN-module] approve: <artifact> gate-N passed
  [spec][NN-module] rfc: RFC-NNN applied — <summary>
  [spec] fix: <typo or clarity fix with no semantic change>

Implementation:
  feat(module): <what was added>
  fix(module): <what was corrected>
  revert: <what was removed and why>
  test(module): <what scenarios were covered>
  infra(module): <Bicep change description>
```

- DO NOT use generic messages like `update`, `changes`, `wip`, or `fix stuff`.
- MUST include `Co-Authored-By: <agent-name>` trailer in every commit where AI generated the majority of the content.

---

### 5. Prohibited Actions (Hard Limits)

The following actions are unconditionally prohibited for all agents operating in this repository:

```
DO NOT modify requirements.md, plan.md, or tasks.md of an approved module
         without a corresponding RFC-NNN reference in the commit message.

DO NOT generate implementation code for a module whose tasks.md has not
         received a Gate 2 approval signal.

DO NOT merge any branch without at least one human approval recorded.

DO NOT deploy or trigger any deployment pipeline without Gate 4 sign-off
         recorded in Azure DevOps.

DO NOT hardcode Azure connection strings, API keys, or secrets in any file.
         All secrets MUST be resolved via Azure Key Vault references or
         environment variables defined in the project's .env.example.

DO NOT introduce any dependency (npm package, Azure service, or external API)
         not present in the approved stack: TypeScript/Node, React, Bicep,
         Azure OpenAI, Azure AI Search, Azure Functions, Bot Framework SDK.

DO NOT use the 12 documents flagged with pending Compliance contradictions
         as authoritative content in prompts, tests, or seeded data.
```

---

*Section maintained by: Delivery Manager*
*Last updated: 2026-06-10*
*Applies to agents: Claude Code, GitHub Copilot, and any future automated tool operating in `novatech-assistant`.*
