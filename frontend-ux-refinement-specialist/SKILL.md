---
name: frontend-ux-refinement-specialist
description: Improve product interface quality in existing frontend applications by refining visual hierarchy, layout clarity, navigation, microcopy, empty states, feedback states, interaction flow, and overall usability. Use when the request is to polish a product UI, review UX/UI quality, reduce cognitive load, clarify screens, improve dashboards/forms/settings pages, strengthen user guidance, or make an interface feel more coherent and easier to use without changing the core product scope.
---

# Frontend UX Refinement Specialist

## Overview

Refine existing product interfaces so they become clearer, easier to scan, and easier to act in.
Prioritize hierarchy, comprehension, state design, and task flow over decorative redesign.

## Operating Mode

Treat this skill as a product-interface refinement workflow, not a marketing-site redesign workflow.
Preserve the product's established design system, component model, and interaction patterns unless they are the source of the problem.

Prefer improving what already exists:

- clarify what the page is for
- surface the most important actions earlier
- reduce redundant text and visual noise
- improve labels, helper text, and status feedback
- strengthen empty, loading, error, and success states
- simplify navigation and decision paths

## Audit First

Before editing, identify:

1. The primary user job on the screen.
2. The main decision points and actions.
3. The current blockers to understanding or action.
4. Which issues are structural versus cosmetic.

Inspect the interface through these lenses:

- hierarchy: Is it obvious what matters first, second, and last?
- clarity: Are labels, headings, and controls immediately understandable?
- density: Is the user seeing too much at once?
- navigation: Is it clear where to go next and how to return?
- microcopy: Do buttons, helper text, and notices explain intent precisely?
- states: Are empty/loading/error/success states useful and reassuring?
- feedback: Does the UI confirm what changed, what is happening, and what failed?
- consistency: Do spacing, tone, labels, and interaction patterns match the rest of the app?

## Prioritization Rules

Prioritize fixes in this order:

1. Remove confusion that blocks task completion.
2. Improve information hierarchy and call-to-action placement.
3. Fix misleading or vague copy.
4. Improve system feedback and edge states.
5. Clean up spacing, grouping, and polish details.

Do not spend time on surface styling if the structure is still weak.

## Refinement Workflow

### 1. Define the target experience

State in one or two sentences what the user should understand and accomplish on the screen.

Example:

- "The user should immediately see current performance, know what needs attention, and act without hunting through secondary panels."
- "The user should understand plan limits, current status, and next billing action without reading dense explanatory text."

### 2. Restructure before restyling

Reorder sections, headings, summaries, and actions before changing visual treatment.

Prefer patterns such as:

- one strong page heading with a clear supporting sentence
- one primary action per section or decision cluster
- summary cards or status rows before dense detail tables
- advanced or low-frequency actions pushed downward or behind progressive disclosure

### 3. Tighten microcopy

Rewrite copy to be concrete, short, and operational.

Prefer:

- specific labels over generic ones
- action-oriented button text
- helper text that answers "what happens next?"
- notices that explain consequence, not just status

Avoid:

- vague labels like "Manage", "Configure", or "Continue" without context
- long explanatory paragraphs above obvious controls
- technical/internal terminology when a user-facing term exists

### 4. Strengthen states

For every important screen, verify:

- empty state explains why the page is empty and what to do next
- loading state preserves layout and expectation
- error state tells the user what failed and what they can retry
- success feedback confirms the result without requiring guesswork
- disabled controls explain why they are unavailable when needed

### 5. Preserve implementation realism

Keep refinements proportional to the existing codebase.
Favor changes that can be implemented cleanly with the current component system and routing model.

If the request includes code changes, implement the smallest coherent change set that materially improves the experience.

## Output Expectations

When responding with analysis only, provide:

1. the main usability problems
2. the root cause behind each problem
3. the highest-value refinements
4. a recommended execution order

When responding with implementation, make the UI changes directly and ensure the final state reflects the analysis.

## Review Standard

Hold the interface to this bar:

- the page purpose is obvious within a few seconds
- the next action is clear
- the most important information appears first
- edge states feel intentional, not leftover
- the copy sounds like the product, not like placeholders

If a screen still feels noisy, hesitant, or ambiguous, continue refining until the structure is defensible.
