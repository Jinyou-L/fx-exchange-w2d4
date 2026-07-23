# Day 4 status — testing, CI/CD, and supply chain

## Tested at each altitude

- **Unit:** `com.fx.api.ConversionServiceTest` mocks `RateRepository` and covers conversion rounding, retail fees, net amount, and the unknown-pair exception.
- **Web slice:** `RateControllerTest` checks rate-list JSON, unknown-pair 404 responses, a valid conversion POST (201), and invalid negative input (400), without a database.
- **Integration:** `RateRepositoryIT` runs the real `JdbcTemplate` queries against a throwaway MySQL. Liquibase creates and seeds its schema: `findLatest()` returns 10 pairs, EUR/USD is 1.0818, and an unknown pair is empty.
- Mockito uses its subclass mock maker in tests, avoiding a runtime JVM-agent attachment requirement; every mocked collaborator here is mockable without the inline engine.

## What CI guards

- Pull requests run `mvn -B verify`, covering unit, web-slice, and Testcontainers integration tests; the integration-test summary must show that none were skipped.
- The CI workflow also validates Maven configuration and runs a non-blocking OWASP dependency check. Configure `build` as a required status check in the `main` branch ruleset so a red PR cannot merge.
- Dependabot is configured weekly for Maven and GitHub Actions. Team rule: review Dependabot PRs within one working day; merge security updates the same day when CI is green.

## What CI does not guard

- CI does not run a full user-facing compose-stack smoke test. On a clean machine, `docker compose up` must serve `/api/rates` with 10 rates.
- The monitor UI has no Day 4 automated coverage; it is outside this day's scope.
- Repository settings still require human configuration: enable Dependabot alerts/security updates, require the `build` check for `main`, and make `staging` require a reviewer.

## Dependency inventory

- `./mvnw dependency:tree` was reviewed on 2026-07-23. Two transitive examples are `org.apache.tomcat.embed:tomcat-embed-core:10.1.30`, pulled in by `spring-boot-starter-web` → `spring-boot-starter-tomcat`, and `org.apache.tomcat.embed:tomcat-embed-el:10.1.30`, pulled in by `spring-boot-starter-validation`.
- Manual NVD lookup: Tomcat 10.1.30 is affected by **CVE-2024-52317** (HTTP/2 request/response recycling; upgrade recommendation: 10.1.31) and later advisories, including CVE-2026-24880. The version is Spring Boot 3.3.4-managed, so address it by upgrading the Boot dependency management rather than overriding a single transitive jar. Dependabot and the non-blocking OWASP job surface follow-up updates.

## Definition of Done

A change is **done** only when all of these hold:

1. From `fx-app-spring/`, `./mvnw verify` is green — unit, slice, and integration tests, with nothing wrongly skipped — and `docker compose up` on a clean machine serves `/api/rates` returning 10 rates.
2. The new behaviour has tests at the right altitude: a new endpoint has slice happy and failure paths; a new calculation has unit tests with boundaries; new or changed SQL has an `*IT`.
3. A teammate reviewed the PR, checked out the branch, ran `./mvnw verify`, and approved it.
4. The change merged to `main` only through that PR — never a direct push.
5. `main` remains green after the merge, and a fresh clone builds and verifies.

Not done: a failing or wrongly skipped test; a change that only works on one laptop; or code merged without review.
