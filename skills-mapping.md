## System Context

* **Part of:** `@ai-orchestrator/system/ORCHESTRATOR.md`
* **Used by:** `@ai-orchestrator/system/SKILL_DISCOVERY.md` — consulted during domain identification
* **Uses:** nothing
* **Outputs to:** `@ai-orchestrator/system/SKILL_EXECUTOR.md` (via skill selection in SKILL_DISCOVERY)

---

# SKILL MAPPING SYSTEM

## Purpose

Select relevant skills based on problem context.

---

## Discovery Command

`npx skills find`

---

## Auto-Selection Rules

### Frontend (React Native / Expo)

npx query: `npx skills find react-native`

Trigger when:

* `.tsx` / `.jsx` files are in scope
* screens or components are mentioned
* UI behavior or rendering issues are reported

Possible skills:

* `react-native`
* `frontend-performance`
* `re-renders`
* `component-architecture`

### Frontend (Web — React / Vue / Svelte / Angular)

npx query: `npx skills find frontend`

Trigger when:

* `.tsx` / `.jsx` / `.vue` / `.svelte` files are in scope
* browser-specific APIs, DOM manipulation, or SSR are involved
* bundle size, hydration, or client-side routing issues appear

Possible skills:

* `frontend`
* `web-performance`
* `accessibility`
* `component-architecture`

### Backend (Node / Express)

npx query: `npx skills find backend-validation`

Trigger when:

* routes, controllers, or middleware are involved
* validation, auth, or error behavior is in scope

Possible skills:

* `backend-validation`
* `api-design`
* `security`
* `error-handling`

### Backend (Python — Django / FastAPI / Flask)

npx query: `npx skills find python-backend`

Trigger when:

* `.py` files with route handlers, views, or serializers are in scope
* ORM usage, middleware, or async patterns are involved

Possible skills:

* `python-backend`
* `api-design`
* `security`
* `error-handling`

### Backend (Go)

npx query: `npx skills find go-backend`

Trigger when:

* `.go` files with HTTP handlers, middleware, or gRPC services are in scope
* concurrency patterns, goroutine management, or channel usage are involved

Possible skills:

* `go-backend`
* `api-design`
* `concurrency`
* `error-handling`

### iOS / Swift

npx query: `npx skills find swift`

Trigger when:

* `.swift` files are in scope
* SwiftUI, UIKit, or Combine patterns are involved
* app lifecycle, navigation, or platform-specific APIs are mentioned

Possible skills:

* `swift`
* `ios-performance`
* `swiftui`
* `app-architecture`

### Android / Kotlin

npx query: `npx skills find kotlin-android`

Trigger when:

* `.kt` files are in scope
* Jetpack Compose, Android lifecycle, or coroutines are involved
* Gradle build, dependency injection, or platform APIs are mentioned

Possible skills:

* `kotlin-android`
* `android-performance`
* `compose`
* `app-architecture`

### Database

npx query: `npx skills find database-performance`

Trigger when:

* schema, queries, migrations, or indexes are involved
* ORM configurations, connection pooling, or query optimization are in scope

Possible skills:

* `database-performance`
* `indexing`
* `schema-design`
* `transactions`

### Performance

npx query: `npx skills find performance`

Trigger when:

* measurable slowness is reported (latency, re-renders, load time)
* repeated fetching, cache misses, or memory issues are present
* profiling data or benchmarks are available

Possible skills:

* `performance`
* `caching`
* `memoization`
* `state-optimization`

### Infrastructure / DevOps

npx query: `npx skills find infrastructure`

Trigger when:

* Terraform, Docker, Kubernetes, or CI/CD configs are in scope
* deployment pipelines, environment configs, or infrastructure-as-code are involved
* scaling, monitoring, or reliability concerns appear

Possible skills:

* `infrastructure`
* `docker`
* `ci-cd`
* `monitoring`

### Testing

npx query: `npx skills find testing`

Trigger when:

* test files or test coverage gaps are in scope
* flaky tests, missing assertions, or test architecture issues are reported
* migration from one test framework to another is involved

Possible skills:

* `testing`
* `test-architecture`
* `integration-testing`
* `test-coverage`

### Architecture

npx query: `npx skills find architecture`

Trigger when:

* files are large or boundaries are unclear
* ownership, scaling, or dependency concerns appear
* monolith decomposition or module extraction is discussed

Possible skills:

* `architecture`
* `modularization`
* `separation-of-concerns`

---

## Rules

* Select at most 3-5 skills.
* Prefer specific skills over general skills.
* Include performance-related skills when evidence supports it.
* Document why each skill was selected.

---

## Output Format

* `selected_skills[]`
* `reason_per_skill`
* `expected_value_per_skill`
