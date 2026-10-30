<!--
Sync Impact Report
- Version change: unratified template -> 1.0.0
- Modified principles: initial adoption; five principles defined from template placeholders.
- Added sections: Technology and Architecture; Development Workflow and Quality Gates.
- Removed sections: none.
- Templates: .specify/templates/tasks-template.md updated; plan and spec templates checked and aligned.
- Spec Kit guidance: .github/agents/speckit.tasks.agent.md updated to require tests for behavior changes.
- Follow-up TODOs: none.
-->

# Metrotom Constitution

## Core Principles

### I. Product-First Scope
The metronome and timer MUST remain the primary product experience. Portfolio content
MUST support the personal presentation of the project without obscuring or delaying
those tools. New features MUST have a clear benefit to the intended user or the
portfolio presentation; speculative features and unrelated services are out of scope.

### II. Vue-First, Portable Architecture
The application MUST use Vue 3, TypeScript, and single-file components, following
existing project conventions. Domain behavior for the metronome and timer MUST remain
separate from browser routing and build-tool details where practical. The current
runtime is Vite with Vue Router; a possible future move to Nuxt MUST NOT justify
premature abstractions or Nuxt dependencies before that migration is approved.

### III. Reliable Timekeeping and Explicit Controls
Timer elapsed time MUST be derived from a clock rather than accumulated interval
callbacks. Metronome ticks MUST use timing suitable for audio playback rather than
assuming UI update intervals are precise. Start, pause, and reset actions MUST have
clear, consistent effects, and the interface MUST communicate when playback or timing
cannot continue as requested.

### IV. Accessible, Consistent Interface
Interactive features MUST be usable by keyboard, expose accessible names and state,
and provide visible focus. Text and controls MUST remain legible across supported
viewport sizes. Tailwind CSS and shadcn-vue components are the standard styling and UI
foundation; Lucide Vue icons MUST complement, not replace, accessible control labels.

### V. Verified, Focused Changes
Changes to metronome or timer behavior MUST include or update automated tests for the
affected behavior, including timing and control-state transitions. Other behavior
changes MUST receive appropriate automated coverage. Changes MUST pass the project
build and type checks before completion. Implementations MUST prefer the simplest
approach consistent with these principles and avoid unrelated dependencies or
abstractions.

## Technology and Architecture

- The current application stack is Vue 3, TypeScript, Vite, and Vue Router.
- Tailwind CSS provides styling; shadcn-vue provides reusable interface components;
  Lucide Vue provides icons.
- Keep the client-side application compatible with its current Vite runtime. Treat
  Nuxt as a possible future migration, not a current requirement.
- Preserve the existing `src/` organization and project conventions unless a concrete
  feature need justifies a change.

## Development Workflow and Quality Gates

- Feature specifications and plans MUST state how they satisfy the relevant
  constitutional principles. Plans MUST resolve any conflict before implementation.
- Tasks MUST include automated tests for metronome and timer behavior, and relevant
  verification for accessibility and routing changes.
- Run the repository's build and type-check command (`npm run build` at adoption)
  after implementation. If behavioral tests cannot be run because the project lacks
  a test runner, add the smallest suitable test setup as part of the behavior change.
- Changes MUST preserve existing behavior unless a specification explicitly approves
  a behavior change.

## Governance

This constitution governs project specifications, plans, tasks, implementation, and
reviews. Amendments MUST be approved by the project owner, documented in this file,
and accompanied by an updated version and amendment date. Use semantic versioning:
MAJOR for incompatible governance or principle changes, MINOR for new principles or
material expansions, and PATCH for clarifications and non-semantic edits.

Every feature plan and code review MUST check applicable principles and quality gates.
Any exception MUST be recorded with its rationale and approved before implementation.
Templates and guidance that encode changed requirements MUST be updated with the
amendment.

**Version**: 1.0.0 | **Ratified**: 2026-10-30 | **Last Amended**: 2026-10-30
