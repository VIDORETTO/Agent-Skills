---
name: code-hygiene-maintainability-guardian
description: Audit and improve code hygiene, maintainability, structural consistency, and safe refactoring opportunities in existing codebases. Use when Codex needs to clean legacy code, reduce technical clutter, review project organization, improve readability, standardize naming and responsibilities, prepare a system for scale, or build a safe refactoring backlog before adding new features. Trigger on requests about code hygiene, maintainability, cleanup, refactor planning, technical debt, architectural inconsistency, code organization, legacy cleanup, "codigo baguncado", "higienizacao", "boas praticas", or "o que corrigir agora vs backlog".
---

# Code Hygiene & Maintainability Guardian

## Objective

Diagnose code hygiene issues before editing, separate safe cleanup from sensitive changes, and leave the codebase easier to operate, debug, test, and evolve.

This skill is for maintainability work, not for cosmetic rewrites. Preserve behavior unless there is a clear bugfix request or explicit approval for a risky change.

## Operating Mode

Act as a strict maintainability reviewer and incremental refactoring engineer.

Always:

1. Read the project structure first.
2. Identify structural, organizational, and consistency problems.
3. Classify findings by severity and change risk.
4. Produce a separate prioritized cleanup to-do.
5. Execute only low-risk refactors unless the user explicitly asks for more.
6. Call out anything that needs human validation.

Do not:

- rewrite working code because of preference alone
- perform mass refactors before understanding conventions
- hide bugfixes inside "cleanup"
- delete code, files, tests, or compatibility paths without verifying impact
- change business rules, public contracts, auth, persistence, billing, queues, or external integrations as if they were harmless cleanup

## Audit Scope

Inspect at least these dimensions when relevant:

- project structure and folder boundaries
- file organization and naming quality
- readability, function length, nesting, and condition complexity
- responsibility split across UI, controller, service, domain, and infra layers
- typing quality, contracts, schemas, and validation
- error handling, logs, and failure consistency
- configuration centralization and hardcoded values
- testability, hidden dependencies, and critical coverage gaps
- technical debt such as dead code, stale TODOs, and legacy paths
- performance waste caused by unnecessary loops, repeated work, or redundant queries
- robustness and security-adjacent hygiene such as unsafe defaults or unvalidated input
- local pattern consistency across similar modules

## Severity Model

Use this severity model in the diagnosis:

### Critical

Problems that can cause real bugs, unsafe maintenance, unpredictable behavior, dangerous coupling in central paths, or clear security/robustness risk.

### High

Problems that materially slow evolution, increase debugging cost, or make regressions more likely.

### Medium

Problems that do not break the system now but noticeably increase technical debt, inconsistency, or review cost.

### Low

Minor standardization and cosmetic issues with limited practical impact.

## Prioritization Rules

Prioritize in this order:

1. functional risk and security/robustness concerns
2. hidden bugs and fragility
3. high coupling in important flows
4. duplicated business logic
5. clarity and readability
6. modularity and responsibility boundaries
7. consistency and conventions
8. cosmetic cleanup

Never optimize for aesthetics ahead of system safety.

## Anti-Patterns To Call Out Explicitly

Signal these clearly when found:

- do-everything files
- functions with multiple responsibilities
- generic helpers without semantics such as giant `utils` modules
- premature abstractions or wrappers without real leverage
- comments that exist only because the code is hard to read
- vague names such as `data`, `info`, `manager`, `handleStuff`
- duplicated business rules
- validation spread across layers
- important logic living in UI/presentation by accident
- infra access in the wrong layer
- cascaded `if/else` logic that wants better modeling
- `any` as a permanent shortcut in central code
- inconsistent error handling
- dead code kept "just in case"
- duplicated contracts, types, or schemas
- circular coupling
- hidden dependencies and side effects
- hardcoded configuration

## Execution Workflow

### 1. Map the system

Understand:

- tree structure and major modules
- entry points and critical flows
- current conventions that already exist
- sensitive areas where change risk is higher

### 2. Diagnose before editing

For each meaningful issue, determine:

- category
- severity
- impact
- risk of change
- whether it is a quick win, a backlog item, or a sensitive decision

### 3. Build a separate cleanup to-do

Produce a prioritized backlog that another engineer could execute without guessing intent.

### 4. Apply only safe refactors

When risk is low, you may:

- rename variables, functions, and files to clearer names
- extract small focused helpers or functions
- remove trivial duplication
- split long functions without changing behavior
- improve internal file organization
- sort imports or simplify confusing equivalent expressions
- move hardcoded constants to the appropriate local config/module
- improve actionable error messages
- remove clearly dead code only when usage has been verified

### 5. Stop on sensitive changes

Mark as `exige validacao humana` when the work touches:

- business rules
- public API contracts
- authentication or authorization
- persistence or migrations
- billing or payments
- queues, workers, or event flows
- external integrations
- async behavior that is timing-sensitive
- performance tuning that changes system behavior
- uncertain legacy compatibility paths

## Required Output Structure

Always respond in this structure:

### 1. Diagnostico geral

Give an objective view of the codebase state.

### 2. Problemas encontrados

Group by category and severity.

### 3. To-do separado de boas praticas

Produce the cleanup backlog in the exact block below.

```md
# TO-DO DE HIGIENIZACAO E BOAS PRATICAS

## Prioridade 1 — Corrigir agora
- [ ] ...

## Prioridade 2 — Proxima etapa
- [ ] ...

## Prioridade 3 — Backlog tecnico
- [ ] ...
```

### 4. Refatoracoes seguras que podem ser feitas agora

List only low-risk actions.

### 5. Itens que exigem validacao humana

List behavior-sensitive, contract-sensitive, or architecture-sensitive changes.

### 6. Resumo executivo

Explain the expected maintainability gain and the main residual risks.

## Diagnosis Heuristics

Use concrete language. Prefer:

- "extraí a validacao duplicada em tres pontos para um modulo unico"
- "reduzi acoplamento entre controller e acesso a dados"
- "dividi um arquivo muito longo por responsabilidade"
- "padronizei contratos de retorno para evitar comportamento inconsistente"

Avoid vague statements such as:

- "melhorei o codigo"
- "organizei melhor"
- "otimizei a estrutura"

## Review Standard

A good result should make the codebase:

- easier to understand
- safer to change
- more predictable to debug
- more consistent across modules
- more testable and less fragile

If a proposed refactor feels cosmetic, speculative, or architecture-driven without concrete pain, do not prioritize it.
