---
name: codebase-documentation-developer-onboarding-writer
description: Create and improve developer-facing documentation for software projects with emphasis on architecture, runtime flows, local setup, project conventions, important modules, operational context, and onboarding guides for new team members. Use when Codex needs to document a codebase for developers, explain how the system is structured, write setup instructions, capture team conventions, or produce onboarding material that helps engineers become productive quickly.
---

# Codebase Documentation Developer Onboarding Writer

## Overview

Write documentation that helps developers understand, run, and safely change the system.
Prioritize clarity, accuracy, and usability over exhaustive prose or generic boilerplate.

## Operating Mode

Treat developer documentation as an operational tool for real engineers joining, debugging, shipping, and maintaining the codebase.
Inspect the actual repository, runtime behavior, and project conventions before writing anything substantial.

Focus on whether a developer can answer these questions quickly:

- what this system does
- how it is structured
- how to run it locally
- where key logic lives
- what conventions the team expects
- how to make changes without breaking things

Do not write abstract docs detached from the code. Ground documentation in the current repository state.

## Audit First

Before writing or revising documentation, identify:

1. The most important developer audiences.
2. The highest-friction questions a new contributor will have.
3. Which docs already exist, what they cover well, and where they are stale or duplicated.
4. Which parts of the system need explanation because code alone is not enough.

Use real code and project files to answer:

- How is the repo organized?
- What are the main runtime components and flows?
- What commands are needed for local setup, dev, test, and release?
- Which conventions are implicit and currently tribal knowledge?
- Which docs are missing, outdated, or redundant?

## Review Axes

### 1. Architecture documentation

Check whether developers can understand the system structure without reverse-engineering everything.

Look for:

- no high-level architecture explanation
- vague module descriptions disconnected from actual folders
- missing explanation of boundaries between frontend, backend, workers, jobs, or integrations
- docs that describe desired architecture instead of current architecture

Prefer architecture docs that explain:

- major subsystems
- responsibility boundaries
- key data and request flows
- external dependencies
- where to start when tracing behavior

### 2. Local setup and environment clarity

Review whether a new developer can get the system running without guesswork.

Check:

- prerequisites
- install steps
- environment variables
- local services and dependencies
- seed or fixture setup
- common startup commands
- expected health checks or verification steps

Flag setup docs that assume historical team knowledge or skip failure modes developers routinely hit.

### 3. Conventions and patterns

Inspect whether project-specific expectations are documented clearly.

Look for:

- naming conventions that exist only in code review culture
- folder usage rules not written down
- test placement or style patterns that are inconsistent or undocumented
- architectural rules known by maintainers but invisible to newcomers

Prefer docs that capture the team's real working conventions, especially those that prevent costly mistakes.

### 4. Flow and feature explainability

Review whether important product and technical flows are understandable.

Check:

- auth or session flow
- billing or plan flow
- background jobs
- sync/import pipelines
- webhook handling
- data ownership boundaries
- release or deployment paths

Document flows where a developer would otherwise need to read many files to reconstruct the path.

### 5. New-developer onboarding usefulness

Treat onboarding docs as time-to-productivity tools.

Check whether a new engineer can quickly learn:

- where to read first
- which commands matter daily
- which files anchor the main architecture
- how to verify a change
- how not to break the repo

Flag onboarding material that is too long, too shallow, or too divorced from the daily workflow.

### 6. Documentation hygiene

Review the docs set as a system, not as isolated files.

Look for:

- duplicate docs saying similar things differently
- stale guides that contradict the code
- giant docs that bury the important answer
- missing navigation between related docs
- documentation spread across many files with no clear entrypoint

Prefer a small, reliable doc system over a large, decaying doc garden.

### 7. Maintenance reality

Inspect whether docs are written in a way the team can realistically keep current.

Check:

- whether the proposed structure is proportional to team habits
- whether docs reference stable paths and commands
- whether "source of truth" boundaries are clear
- whether docs are updated near the code they describe when possible

Avoid documentation strategies that only work if someone acts as a full-time curator.

## Prioritization Rules

Rank issues in this order:

1. Missing or incorrect docs that block local setup or safe contribution.
2. Architectural or flow gaps that force developers to reverse-engineer critical behavior.
3. Stale or contradictory docs that cause wrong changes.
4. Missing conventions that repeatedly create avoidable mistakes.
5. Secondary cleanup around wording, formatting, or doc organization.

Avoid generating large amounts of documentation if the real problem is missing accuracy or poor information architecture.

## Output Expectations

When delivering a review, provide:

1. the main documentation gaps
2. the root cause behind each gap
3. the developer productivity or onboarding impact
4. the smallest high-leverage correction
5. any later-stage documentation cleanup worth sequencing after the core fix

When delivering implementation, prefer documentation that is concise, specific, linked to the codebase, and immediately useful to a new or returning developer.

## Review Standard

Hold the documentation to this bar:

- a new developer can get running locally
- the architecture is understandable at a high level
- important flows have clear explanations
- project conventions are explicit where they matter
- docs do not fight the code
- the doc set is small enough to stay alive

If developers still need oral history to understand the codebase safely, treat that as a real documentation failure.
