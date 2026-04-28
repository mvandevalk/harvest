<!--
  SYNC IMPACT REPORT
  ==================
  Version change: 1.0.0 → 1.1.0
  Modified principles: None renamed
  Added sections:
    - Core Principles: VI. Code Quality & Type Safety (new principle)
    - Core Principles: VII. Performance as a Requirement (new principle)
    - Core Principles: V expanded with Design System requirements
    - Quality Standards: explicit CI-enforced Quality Gates checklist
    - Quality Standards: Definition of Done checklist
    - Development Workflow: Code Review Requirements
    - Development Workflow: Performance budgets with concrete targets
  Removed sections: None
  Templates requiring updates:
    - .specify/templates/plan-template.md  — Constitution Check should include Quality Gates ✅ compatible
    - .specify/templates/spec-template.md  — Success Criteria should reference perf budgets ✅ compatible
    - .specify/templates/tasks-template.md — Definition of Done applies to task completion ✅ compatible
  Follow-up TODOs:
    - None. All placeholders resolved.
-->

# Harvest Constitution

## Core Principles

### I. Component Reusability (NON-NEGOTIABLE)

Every UI element, hook, service, and utility MUST be built as a reusable, self-contained
unit before being wired into a feature. Components with shared responsibilities MUST be
extracted into a shared library or common module — duplication is not permitted when a
reusable abstraction already exists or can be cleanly introduced.

- Components MUST accept props/configuration rather than hard-coding context-specific values.
- Shared logic (data fetching, formatting, validation) MUST live in dedicated hooks or
  service modules, never duplicated inline across features.
- A component MUST have a single, clearly named responsibility. Compound responsibilities
  MUST be split before merging.
- Before building a new component, MUST search for an existing one that can be extended.
- Shared components MUST live in a dedicated shared/common package — not scattered across
  feature folders.

**Rationale**: Meal planning UIs are inherently repetitive (ingredient cards, meal slots,
calendar grids). Without a reusability mandate, maintenance cost scales with feature count
rather than staying flat.

### II. Test-First Development (NON-NEGOTIABLE)

Tests MUST be written and confirmed to FAIL before implementation begins. The Red-Green-Refactor
cycle is strictly enforced.

- Unit tests MUST cover all reusable components, hooks, and service functions.
- Integration tests MUST cover every user journey defined in the spec's User Scenarios section.
- Every user story MUST have at least one independent acceptance test that validates it in isolation.
- A task is NOT complete until its corresponding tests pass and CI is green.
- Test coverage for new code MUST NOT fall below 80% line coverage.

**Rationale**: Meal planner logic (scheduling, substitutions, shopping list generation) is
prone to subtle regressions. Test-first discipline catches breakage before it reaches users.

### III. UX Simplicity — Three-Click Rule

Every primary user action (plan a meal, add an ingredient, generate a shopping list) MUST be
completable within three interactions from the application home screen.

- New features MUST NOT increase the interaction depth of existing primary flows.
- Loading states, empty states, and error states MUST be explicitly designed and implemented
  for every user-facing screen — they are not optional polish.
- Default values and smart suggestions MUST be provided wherever user input is required,
  reducing cognitive load.
- Interaction patterns (navigation, feedback, error states) MUST be consistent across all
  screens and flows.

**Rationale**: Harvest is a daily-use tool. Friction in primary workflows leads to abandonment.
Simplicity is a functional requirement, not a design preference.

### IV. Integration Testing for User Journeys

Every user story defined in a spec MUST have at least one end-to-end integration test that
exercises the full journey from user action to persisted/rendered result.

- Integration tests MUST use realistic data (representative meal plans, ingredient lists).
- Contract tests MUST be written for any API boundary, internal service interface, or
  storage schema change.
- Integration tests MUST run in CI on every pull request — they are not optional.

**Rationale**: Component-level unit tests do not catch wiring failures between the meal
scheduler, ingredient store, and calendar view. Integration tests do.

### V. Accessibility & Responsiveness (NON-NEGOTIABLE)

All UI features MUST meet WCAG 2.1 AA accessibility standards and MUST render correctly
on mobile (≥ 375px), tablet (≥ 768px), and desktop (≥ 1280px) viewports.

- Interactive elements MUST have visible focus indicators and ARIA labels where native
  semantics are insufficient.
- Color MUST NOT be the sole means of conveying information (e.g., meal status, allergen flags).
- Color contrast MUST meet WCAG 2.1 AA: 4.5:1 for normal text, 3:1 for large text.
- Keyboard navigation MUST work for all interactive flows.
- All visual tokens (color, spacing, typography) MUST be defined centrally via design system
  variables — no hard-coded values in component styles.
- Responsive layout MUST be validated before a feature is considered complete.

**Rationale**: Harvest is used in the kitchen, often on mobile. A feature that is only
desktop-usable is a partially shipped feature.

### VI. Code Quality & Type Safety (NON-NEGOTIABLE)

All code MUST be readable, self-documenting, and free of unnecessary complexity.

- All public APIs, component props, and service interfaces MUST have explicit types; implicit
  `any` or untyped returns are prohibited.
- Linting and formatting rules MUST be enforced via CI — no exceptions and no bypasses.
- Duplication MUST be avoided — extract shared logic into reusable utilities or components.
- New dependencies MUST be justified in the PR description against their size and maintenance cost.

**Rationale**: Inconsistent quality and missing types create compounding maintenance debt.
A high, enforced baseline keeps the codebase navigable and onboarding friction low.

### VII. Performance as a Requirement

Performance budgets MUST be defined per feature before implementation begins (recorded in plan.md).
Performance regressions detected in CI MUST block merging.

- **Initial page load**: JavaScript bundle MUST be < 200 KB gzipped for the critical path.
- **Core Web Vitals**: LCP < 2.5s, FID/INP < 100ms, CLS < 0.1 on representative hardware.
- **Time to Interactive**: < 3s on simulated mid-range mobile (4G).
- **API response time**: p95 < 300ms for read endpoints, p95 < 500ms for write endpoints.
- Lazy loading and code splitting MUST be used for non-critical paths.
- Bundle size MUST be monitored; new dependencies MUST be justified against their size cost.

**Rationale**: Harvest is a kitchen tool — users interact on mobile, often while cooking.
Performance is a form of respect for users' time. Budgets enforced from the start prevent
costly rewrites later.

## Quality Standards

### CI-Enforced Quality Gates (non-bypassable)

All of the following MUST pass before any PR can be merged:

1. **Lint + Format**: Zero lint errors; formatter check passes.
2. **Type check**: No type errors in strict mode.
3. **Unit tests**: All pass; coverage MUST NOT regress below 80% baseline.
4. **Integration tests**: All contract and integration tests pass.
5. **Bundle analysis**: Bundle size diff reported; regression blocks merge if over budget.
6. **Accessibility audit**: Automated a11y checks pass (zero critical violations).

### Definition of Done

A task or feature is DONE only when ALL of the following are true:

- All acceptance scenarios from the spec pass.
- All CI quality gates pass.
- Code review approved by at least one other contributor.
- Error boundaries implemented at the feature level.
- Performance budget verified and documented in plan.md.

### Additional Standards

- Pull requests MUST NOT reduce overall test coverage below 80%.
- Error boundaries MUST be implemented at the feature level to prevent a single widget failure
  from crashing the entire application.
- New design patterns introduced in a feature MUST be promoted to the shared design system if
  they appear in more than one place.

## Development Workflow

### Branching & PR Requirements

- Feature branches MUST be named `###-short-description` matching the spec folder.
- Every PR MUST reference its spec (`specs/###-feature-name/`) and include a link to the
  relevant user stories.
- The Constitution Check gate in `plan.md` MUST be completed and passing before Phase 0
  research is considered done.
- Complexity violations (e.g., departing from the three-click rule) MUST be documented in
  the Complexity Tracking table in `plan.md` before implementation begins.
- Each user story MUST be committed and demo-able independently before the next priority
  story begins.

### Code Review Requirements

- Every PR MUST be reviewed by at least one other contributor before merge.
- The reviewer MUST verify Constitution Check compliance (see plan-template.md Constitution
  Check section).
- Complexity additions MUST be documented in the Complexity Tracking table.
- New components MUST be accompanied by usage documentation.

## Governance

This constitution supersedes all other development practices for the Harvest project.
Amendments require:

1. A written rationale explaining why the change is needed.
2. An updated Sync Impact Report (as an HTML comment at the top of this file).
3. Propagation of the change to any affected spec, plan, or tasks templates.
4. Version bump following semantic versioning:
   - **MAJOR**: Principle removed or redefined in a backward-incompatible way.
   - **MINOR**: New principle or section added, or material guidance expanded.
   - **PATCH**: Clarification, wording fix, or non-semantic refinement.

All PRs MUST include a Constitution Check confirming compliance. Violations MUST be
documented and justified — silent non-compliance is not permitted.

**Version**: 1.1.0 | **Ratified**: 2026-04-28 | **Last Amended**: 2026-04-28
