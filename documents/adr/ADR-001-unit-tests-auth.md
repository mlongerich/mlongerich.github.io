# Architectural Decision Record: 1) When you say “unit tests to see if the file creation works now,” which specific file-creation scenario are we validating? For example:
- Creating new content files (e.g., Markdown) during site generation
- Generating output HTML or other artifacts in a build/output directory
- Creating files via an API or some automation process
- Something else

2) Who will rely on these tests, and how will they be used in practice? For instance:
- Developers in CI to prevent regressions
- Content authors ensuring their workflow creates expected files
- QA/ops validating build artefacts
- Other stakeholders
What would “success” look like from a business perspective (e.g., reduced regressions, faster feedback, etc.)?

If you can share brief answers to these, I’ll tailor the requirements gathering plan and outline the next, concrete questions.

**Date:** 2025-09-11
**Repository:** mlongerich/mlongerich.github.io
**Status:** Proposed

## Context

This plan proposes a conservative, forum-friendly approach: use GitHub Pages with Jekyll as the baseline for site generation, and add a targeted unit test harness to validate file-creation workflows (e.g., new Markdown content producing HTML in the output directory). The implementation focuses on bringing testability to content-generation steps, integrating with PR-based CI, and providing clear guidance for maintenance, security, and deployment. The approach minimizes disruption while establishing a verifiable quality gate around content creation and page generation.

## Architecture Decisions

### Adopt GitHub Pages with Jekyll as the primary site generation and hosting mechanism, complemented by a CI pipeline to prebuild and validate content changes before deployment.

**Rationale:** GitHub Pages is the natural hosting target for a user/organization site like mlongerich.github.io. Jekyll is the default static site generator supported by GitHub Pages, works well with Markdown content, and aligns with typical sites in this space. This choice minimizes operational overhead for content authors and provides a predictable build pipeline. A CI layer (e.g., GitHub Actions) can validate file-creation scenarios and ensure regressions are caught before changes are merged.

**Impact:** Standardizes tooling around Ruby/Jekyll, simplifies contributor flow, enables straightforward unit/integration tests around file creation and page generation, and integrates cleanly with PR-based CI on GitHub.

**Risks:** GitHub Pages plugin restrictions may limit certain Jekyll plugins; Ruby/Jekyll version compatibility with GH Pages needs to be pinned; build environment must be controlled in CI to avoid drift; potential longer build times for large sites if incremental builds are not leveraged.

## Implementation Plan

### Overview
Establish a testable, CI-driven build flow for the Jekyll-based site and add unit tests focused on file creation workflows. Provide a path to validate that new content files (e.g., Markdown posts) generate corresponding HTML artifacts in the output directory.

### Critical Path
Phase 1 foundations and Phase 2 test coverage must be in place before Phase 3 CI gating can be reliably activated. Ruby/Jekyll version pinning and GH Pages compatibility must be resolved early to avoid CI failures.

## Technology Recommendations

### Static site generator
**Recommendation:** Jekyll (Ruby) with GitHub Pages as the primary workflow
**Rationale:** Aligned with GH Pages conventions, strong Markdown integration, large ecosystem of themes, straightforward publishing from PRs

### CI/CD
**Recommendation:** GitHub Actions for build, test, and deployment; use a matrix for Ruby/Jekyll versions
**Rationale:** Keeps everything in the same ecosystem; easy to trigger on PRs; native visibility in GitHub

### Testing framework
**Recommendation:** Ruby-based tests (RSpec or Minitest) for file-creation harness; optional Node.js tests if using Eleventy
**Rationale:** Leverages platform language of the generator; straightforward filesystem assertions

## Technical Risks

### Ruby/Jekyll version drift or compatibility issues with GitHub Pages
**Probability:** Medium
**Impact:** High
**Mitigation:** Pin exact Ruby and Jekyll versions in Gemfile; use GitHub Actions matrix to lock versions; avoid plugins not supported by GitHub Pages unless hosting elsewhere

### Test harness flakiness due to filesystem nondeterminism or path issues
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Isolate tests in a clean temp directory; mock filesystem where possible; enforce deterministic file naming

### Plugins and build-time dependencies not allowed in GitHub Pages (safe mode restrictions)
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Limit to allowed plugins; if needed, build artifacts in CI and deploy pre-rendered HTML; document plugin usage constraints

### Slow CI feedback due to large site or heavy tests
**Probability:** Medium
**Impact:** Medium
**Mitigation:** Use incremental builds where feasible; run only targeted tests on small PRs; cache dependencies

## Performance Considerations

**Scalability:** Static sites scale well; build time is the primary growth factor. Consider incremental or cached builds as content grows.

**Performance Targets:** Sub-second page render times on end-user devices; acceptable build times < 5-10 minutes for typical site size; CI build time minimized via caching

**Optimization Strategies:** Enable incremental builds where supported by the generator,Minify HTML/CSS/JS assets in pre-deploy steps,Cache dependencies in CI (Bundler cache, gem caching),Lazy-load heavy assets if applicable

## Security Considerations

**Threat Model:** Malicious content in Markdown that could affect rendering,Supply-chain risk from gem dependencies or plugins,Exposure of build secrets in CI configuration

**Security Controls:** Use trusted plugins only; lock dependencies via Gemfile.lock,Run builds in isolated CI environments with read-only checkout when possible,Review content and metadata to prevent injection or XSS in generated pages

**Compliance:** If hosting public content, ensure license compliance for content and dependencies,Follow organization policies for secrets and access control in CI

## Development Strategy

**Methodology:** Agile with PR-based, gated changes; incremental delivery of test harness and CI capabilities

**Testing Strategy:** Unit tests for file creation, integration tests for build output, and manual exploratory checks for content rendering

**Deployment Strategy:** PR checks gate readiness; on merge to main, deploy to GitHub Pages or prebuilt artifact deployment with a follow-up status

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