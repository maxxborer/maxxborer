# Engineering notes

A closer look at implementation choices behind selected public projects. These are examples of engineering work, **not claims about production scale, business impact, or test coverage**.

## 1. Recovering from expired sessions without a request storm

**Project:** [Token Renewal Example](https://github.com/maxxborer/token-renewal-example) (archived reference implementation)  
**Stack:** Vue 3, TypeScript, Axios, Apollo GraphQL, Express, Nx

**Problem.** Several requests can fail with `401 Unauthorized` at once. If each failure refreshes credentials independently, clients create redundant refresh calls and risk inconsistent retries.

**Approach.**
- Coordinate concurrent failures around **one in-flight refresh**.
- Hold pending requests until that refresh resolves; retry after success or reject when it fails.
- Preserve the original request context across REST and GraphQL transports.
- Treat cancellation and failed refresh as first-class paths, not afterthoughts.

**Trade-off.** The extra coordination layer adds state and complexity to an otherwise simple interceptor. It must be designed for overlapping requests, cancellation, and failure, rather than only the happy path.

**Inspect:** [architecture and sequence diagrams](https://github.com/maxxborer/token-renewal-example/blob/main/docs/architecture.md) · [manual test scenarios](https://github.com/maxxborer/token-renewal-example/blob/main/docs/test-cases.md) · [source and demo](https://github.com/maxxborer/token-renewal-example)

This is a demonstration project; its documented test scenarios should not be interpreted as a verified production SLA.

## 2. Keeping a multi-app TypeScript workspace coherent

**Project:** [maseaaao.tv](https://github.com/maseaaao/maseaaao.tv)  
**Stack:** TypeScript, Bun workspaces, Vite, Biome, Turborepo, Cloudflare Workers, GitHub Actions

**Problem.** A website, OBS browser widgets, companion integrations, and tooling share design and build infrastructure but have different runtime and distribution needs.

**Approach.**
- Organize browser apps, integrations, shared packages, and build tooling in one workspace.
- Maintain shared UI/design packages and centralize dependency versions.
- Run type checks, repository checks, tests, and build/release steps through defined scripts and CI.
- Keep deployment separate from artifact generation so published outputs come from a verified build.

**Trade-off.** A monorepo simplifies shared code and coordinated changes at the cost of more deliberate build orchestration. Not every check is run in every workflow: for example, the repository documents a separate manual C# test command.

**Inspect:** [repository overview](https://github.com/maseaaao/maseaaao.tv) · [scripts and workspace boundaries](https://github.com/maseaaao/maseaaao.tv/blob/main/package.json) · [website build and deployment decisions](https://github.com/maseaaao/maseaaao.tv/blob/main/apps/website/README.md)

## 3. Making networking and security boundaries explicit

**Project:** [LAN Preview for OBS Studio](https://github.com/maseaaao/obs-preview-plugin)  
**Stack:** C++, OBS Studio SDK, HTTP(S), MJPEG, Windows

**Problem.** Previewing OBS Program output on a second device should not require a cloud service or dedicated client app.

**Approach.**
- Expose local MJPEG, snapshot, and health endpoints.
- Offer browser access over HTTP, with optional HTTPS for trusted devices.
- Document performance settings and their impact on OBS.
- Treat install, packaging, and security guidance as part of the deliverable.

**Trade-off.** MJPEG is broadly compatible and easy to consume in a browser, but encoding frames has a CPU cost. HTTPS encrypts traffic; it does **not** authenticate viewers. The current project explicitly limits use to trusted private networks.

**Inspect:** [README and installation](https://github.com/maseaaao/obs-preview-plugin) · [performance methodology](https://github.com/maseaaao/obs-preview-plugin/blob/main/docs/performance-measurement.md) · [security policy](https://github.com/maseaaao/obs-preview-plugin/blob/main/SECURITY.md)

## 4. Shipping a small native product with clear privacy boundaries

**Project:** [Just Calendar Widget](https://github.com/maxxborer/just-calendar-widget)  
**Stack:** Swift, SwiftUI, WidgetKit, Xcode, GitHub Actions

**Problem.** A glanceable desktop calendar does not need a user account, calendar-event permissions, or network analytics.

**Approach.**
- Support three native widget layouts, respecting system locale, first weekday, and time zone.
- Use native tests and repeatable UI-capture tooling in CI.
- Publish installable DMG artifacts with documentation covering signed and ad-hoc builds.

**Trade-off.** Using native WidgetKit limits platform reach but integrates with macOS. Ad-hoc builds are less convenient to install than notarized ones, so the project documents the difference rather than hiding it.

**Inspect:** [README](https://github.com/maxxborer/just-calendar-widget) · [CI workflow](https://github.com/maxxborer/just-calendar-widget/blob/main/.github/workflows/ci.yml) · [releases](https://github.com/maxxborer/just-calendar-widget/releases)

---

I prefer linking to inspectable code, documented behavior, and explicit limitations over attributing unverified impact numbers to personal projects.
