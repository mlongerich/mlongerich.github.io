# Architectural Decision Record: 1) What parts of the codebase should have unit tests? For example, core JavaScript utilities, build scripts, or any React/Vue components if present? Are there any existing tests I should align with or a module you want to prioritize?

2) Do you have a preferred testing framework and CI setup? Options include Jest, Vitest, or Mocha, and whether tests should run in Node or a browser-like environment. Also, would you like GitHub Actions configured for running tests and generating coverage reports?

**Date:** 2025-09-11
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This analysis recommends continuing with a GitHub Pages-based static site driven by a Jekyll-compatible CI workflow, complemented by performance and SEO enhancements and a CMS-driven content workflow if desired. Phase 1 stabilizes builds and resolves path-related issues, Phase 2 optimizes performance/SEO, and Phase 3 introduces a CMS and content governance. Key risks center on build plugin compatibility, CMS integration, and performance regressions, with mitigations centered on CI-driven builds, conservative plugin usage, and measurable performance checks.

## Architecture Decisions

### Maintain GitHub Pages deployment with a static site (Jekyll-oriented workflow) and automate builds via GitHub Actions

**Rationale:** GitHub Pages provides a low-maintenance, cost-effective hosting option for a static site like mlongerich.github.io. A Jekyll-based workflow aligns with GitHub Pages defaults, minimizes external dependencies, and enables reproducible builds when combined with CI. Automation via GitHub Actions mitigates issues like the observed path-related build errors by providing a consistent build environment and a repeatable deployment process.

**Impact:** Improved reliability and repeatable deployments; potential plugin limitations on GitHub Pages require careful selection of supported plugins or a build-on-CI step that outputs static files to gh-pages.

**Risks:** GitHub Pages plugin restrictions may limit certain Jekyll plugins; mitigations include using only whitelisted plugins or moving the build to CI and publishing static output. Path/base URL issues in _config.yml/front matter could reoccur if not validated in the CI environment.

## Implementation Plan

### Overview
A 3-phase plan to stabilize the site, optimize performance and SEO, and enable a content editing workflow, with 2-3 deliverables per phase.

### Critical Path
Stabilizing the CI/CD pipeline and resolving build-time path/front matter issues is the critical path. Once builds are deterministic, performance/SEO enhancements and CMS integration can proceed in parallel. The main dependency is ensuring the chosen deployment approach (gh-pages vs. Netlify/Vercel) aligns with the hosting constraints of GitHub Pages.

## Technology Recommendations

### Static site framework / hosting
**Recommendation:** Continue with Jekyll-based workflow on GitHub Pages; use GitHub Actions to build and deploy the static site to gh-pages
**Rationale:** Low operational overhead, native GitHub Pages support, reproducible builds, and aligns with the existing repository type

### CI/CD & automation
**Recommendation:** GitHub Actions to build and publish static site artifacts to gh-pages
**Rationale:** Automates builds, ensures consistency across environments, reduces path-related issues

### Content editing / CMS
**Recommendation:** Netlify CMS (or similar Git-based CMS) configured to work with GitHub Pages hosting
**Rationale:** Empowers non-technical editors, integrates with a static-hosting workflow, supports previews

### Performance & assets
**Recommendation:** Implement image optimization (compression, responsive sizes, WebP) and optional CSS optimization
**Rationale:** Improves Core Web Vitals, reduces paint times, better mobile experience

### SEO & accessibility
**Recommendation:** Add sitemap.xml, robots.txt, canonical links; ensure semantic HTML and ARIA where appropriate
**Rationale:** Improves discoverability and accessibility with minimal ongoing effort

## Technical Risks

### Build failures due to Jekyll/GitHub Pages plugin incompatibilities or misconfigured baseurl/path leading to routing issues
**Probability:** Medium
**Impact:** High
**Mitigation:** Limit to GitHub Pages-supported plugins; implement CI-driven build that outputs static files to gh-pages; validate baseurl/path in CI by building a sample page; keep _config.yml and front matter consistent across pages

### CMS integration challenges or hosting incompatibility (Netlify CMS, Git-based editing)
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Prototype CMS integration in a dedicated branch, ensure authentication flow is clear, prefer CMS with good GitHub Pages compatibility, set up access controls and previews

### Performance regressions from new assets or CSS framework changes
**Probability:** Low
**Impact:** Medium
**Mitigation:** Establish a performance baseline (Lighthouse scores, key metrics) and run regression tests; implement image optimization and CSS tree-shaking/minification

## Performance Considerations

**Scalability:** Static hosting scales easily; CI build time may grow with content volume but remains predictable

**Performance Targets:** Aim for First Contentful Paint < 1.5s, Time to Interactive < 3s on mobile/desktop for key pages

**Optimization Strategies:** Image optimization pipeline (compression, formats, responsive sizes),Minify CSS/JS, remove unused styles, prefer critical CSS,Enable lazy loading for images and defer non-critical assets

## Security Considerations

**Threat Model:** Static site: low runtime risk; risk mainly from CMS/editor access and build-time integrity

**Security Controls:** Lock down CMS access, enforce strong authentication for editors,Use least-privilege permissions for CI secrets (GitHub Secrets),Validate inputs if any dynamic content is used in templates

**Compliance:** Adhere to general web security best practices; ensure providers used (CMS hosting, DNS) comply with relevant data protection requirements

## Development Strategy

**Methodology:** Agile with incremental improvements; maintain a minimal viable CI/CD baseline first

**Testing Strategy:** CI build validation, basic accessibility checks, manual visual checks; future: automated Lighthouse runs

**Deployment Strategy:** Deploy on push to main branch via CI to gh-pages; consider preview deployments if CMS is enabled

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