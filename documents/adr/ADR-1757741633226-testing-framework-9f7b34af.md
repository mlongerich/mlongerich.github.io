# Architectural Decision Record: I’ll start by gathering a couple of high-level requirements to understand the business goals behind a testing framework for this site. 1) What is the primary objective you want to achieve with a testing framework for mlongerich.github.io? For example, are you aiming to:
- catch broken links and regressions
- ensure accessibility and performance
- validate content correctness and SEO-related checks
- automate visual regression tests
- enable faster, more reliable deployments
- something else

2) Who are the primary users of this testing framework, and how will they interact with it? For instance, are they:
- site maintainers and content editors
- developers/QA engineers
- external contributors
What are their needs in terms of visibility, reporting, and run frequency (e.g., on push, nightly, or on demand)?

Optional context (if helpful for you to share later): Do you already have any CI/CD tools or workflows in place (e.g., GitHub Actions), and are there constraints or preferences for tooling or languages?

**Date:** 2025-09-13
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This analysis proposes a structured, CI-driven testing framework for mlongerich.github.io that encompasses link integrity, accessibility, performance/SEO, content checks, and visual regression. The recommended approach leverages GitHub Actions, Lighthouse CI, Pa11y/axe-core, and Playwright-based visual diff tooling to deliver actionable, scalable, and maintainable quality gates. Phased implementation ensures a solid foundation (Phase 1) before expanding to performance/SEO checks (Phase 2), visual regression (Phase 3), and comprehensive reporting (Phase 4). Key risks include flaky tests and longer CI times, mitigated by test stabilization, parallelization, and staged rollouts. If you provide more details about the repository’s exact stack (Jekyll vs. other static site tooling) and hosting preferences, I can tailor the plan, CI scripts, and tooling choices with concrete configurations and sample workflow files.

## Architecture Decisions

### Adopt a multi-layer testing framework anchored on GitHub Actions to validate link integrity, accessibility, performance, content correctness, SEO, and visual regression for a GitHub Pages static site.

**Rationale:** The site is a static, likely GitHub Pages/Jekyll-based deployment. A CI-driven testing framework ensures regressions are caught early, maintains content quality, and supports efficient deployments. Centralizing tests in CI provides visibility to site maintainers, contributors, and developers, and enables scheduled checks (nightly) in addition to on-push validation.

**Impact:** Requires initial investment to wire multiple test suites (link checks, a11y, SEO/content checks, visual regression) and to configure CI dashboards. Long-term payoff in reliability and faster deployments.

**Risks:** Flaky tests due to dynamic content, CI time increases, and maintenance overhead for multiple tooling ecosystems. Mitigation: stabilize test data, use robust selectors, implement retries with sensible backoff, and modularize tests.

## Implementation Plan

### Overview
Establish a phased, CI-driven testing ecosystem for the static site, starting with baseline checks (lint, link, a11y, basic performance) and evolving to visual regression and reporting. Align with GitHub Actions for automation and GitHub Pages deployment if applicable.

### Critical Path
Baseline CI setup and site build (Phase 1) must succeed before advanced tests (Phase 2–3). LHCI/server availability and a reachable test URL are prerequisites for meaningful Lighthouse runs. Visual regression requires stable baseline images and deterministic rendering across environments.

## Technology Recommendations

### CI/Automation
**Recommendation:** GitHub Actions with a matrix workflow for lint, link checks, a11y, and Lighthouse CI; optional nightly matrix.
**Rationale:** Tightly integrated with GitHub-hosted repository, straightforward reporting, and native artifact/dashboards.

### Accessibility Testing
**Recommendation:** Pa11y/axe-core for automated accessibility checks; Pa11y CI for CI integration.
**Rationale:** Widely adopted, good coverage for WCAG checks, easy to integrate with CI.

### Performance & SEO
**Recommendation:** Lighthouse CI (lhci) or self-hosted LHCI server for dashboards and performance budgets.
**Rationale:** Industry-standard for performance, accessibility, and SEO checks; supports thresholds and dashboards.

### Visual Regression
**Recommendation:** Playwright with pixel-diff or a self-hosted visual regression tool; seed with baseline screenshots.
**Rationale:** Reliable cross-browser rendering, good ecosystem, and integrates with CI.

### Code Quality & Content
**Recommendation:** HTMLHint, stylelint, and pa11y for coverage; optionally custom content validators.
**Rationale:** Improves maintainability and catches common issues early.

## Technical Risks

### Flaky tests due to dynamic content or network flakiness in CI.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Stabilize test data, add retries with backoff, use deterministic selectors, isolate environment-specific tests, and cache assets where possible.

### Increased CI runtime leading to longer feedback loops.
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Parallelize tests by area (lint/link/a11y separately), cache dependencies, and run heavy tests (visual regressions) on a nightly schedule or on-demand.

### Maintenance burden from multiple tooling ecosystems (Ruby, Node, etc.).
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Standardize on a single CI container image that contains all needed runtimes, pin versions, and provide clear contributor onboarding.

### Access and privacy concerns with external testing services for visual diffs.
**Probability:** Low
**Impact:** Medium
**Mitigation:** Prefer in-repo or self-hosted visual regression tooling when possible; avoid leaking content or secrets in diff results.

## Performance Considerations

**Scalability:** Static site typically scales well with CDN-backed hosting; CI-based tests scale with parallelization. Ensure test runners are containerized and can run in parallel across pages.

**Performance Targets:** Page load under 3s on mobile, Lighthouse performance score >= 90, CLS < 0.1, LCP < 2.5s where feasible.

**Optimization Strategies:** Cache dependencies, run Lighthouse in a controlled environment with prebuilt assets, mock external resources where possible.,Prioritize critical pages for initial visual regression baselines.,Use a staging URL or a dedicated test domain for Lighthouse runs.

## Security Considerations

**Threat Model:** Static site exposure with no server-side logic; main surface is content and dependencies.,Possibility of leaking CI credentials or tokens via logs if not careful.,Third-party tooling dependencies may introduce vulnerabilities.

**Security Controls:** Use GitHub Actions Secrets for tokens; avoid printing secrets in logs.,Lock dependency versions and pin transitive dependencies.,Run tests in isolated containers and restrict network access where feasible.

**Compliance:** No special regulatory constraints implied for a personal/portfolio site; ensure accessibility compliance (WCAG 2.1/2.2) is addressed.

## Development Strategy

**Methodology:** Agile-driven, with PR-based quality gates and scheduled nightly validations; incrementally add test coverage per sprint.

**Testing Strategy:** Unit-like checks for content (title/description presence), integration-like checks for page-level QA, end-to-end checks via Lighthouse for user-perceived performance, accessibility, and SEO.,Regression-sensitive tests guarded by thresholds; failures block PRs or nightly deployments.

**Deployment Strategy:** If using GH Pages, deploy artifacts after successful test suite; otherwise deploy to a staging domain or environment.,Use gh-pages or a dedicated deployment workflow with proper branch protections.

## Alternatives Considered

Alternative approaches not documented

## Decision

This architectural approach is recommended based on the technical analysis above.

## Consequences

### Positive
- Technical implementation aligned with business requirements
- Scalable and maintainable solution

### Negative
- Technical complexity and implementation effort
- Potential technical debt if not properly maintained

---