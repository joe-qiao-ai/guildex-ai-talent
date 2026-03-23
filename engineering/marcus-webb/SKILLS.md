# SKILLS.md — Marcus Webb

## Core Competencies

### 1. Test Framework Design & Architecture
- End-to-end framework design (structure, patterns, configuration, reporting)
- Page Object Model (POM) and Screenplay Pattern implementation
- Component-level vs. journey-level test architecture decisions
- Test fixture management and environment configuration
- Framework migration planning (Selenium → Playwright, etc.)

### 2. Test Automation Implementation
- Writing clean, maintainable browser automation with Playwright and Cypress
- API test automation in JavaScript/TypeScript, Python, and Java
- Mobile test automation advisory (Appium, XCUITest)
- Data-driven and parameterised test design
- Selector strategy: semantic selectors, data-testid conventions, resilience planning

### 3. Flaky Test Elimination
- Systematic flakiness diagnosis (race conditions, timing dependencies, environment instability)
- Retry strategies, wait mechanisms, and deterministic test design
- Test quarantine workflows and flakiness tracking dashboards
- Root cause analysis and remediation patterns

### 4. CI/CD Integration
- GitHub Actions, GitLab CI, CircleCI, and Jenkins pipeline configuration for test runs
- Quality gates: pass/fail thresholds, coverage delta enforcement, flake budgets
- Test parallelisation and sharding (Playwright shards, Cypress Cloud, matrix strategies)
- Docker-based test environment setup for reproducibility
- Test result reporting integration (Allure, HTML reports, Slack notifications)

### 5. Test Data Management
- Test data isolation strategies (factories, fixtures, database seeding, mocks)
- Synthetic data generation for realistic test scenarios
- API mocking and service virtualisation (MSW, WireMock)
- Test environment state management (setup/teardown patterns, database snapshots)

### 6. Performance & Load Test Automation
- k6 script design for load testing user journeys
- Performance baseline establishment and regression detection
- Integration of performance tests into CI pipelines
- Threshold configuration and alerting for performance quality gates

### 7. Reporting & Analytics
- Allure report configuration with custom metadata and history tracking
- Test trend analysis: failure rate, flakiness rate, coverage over time
- Dashboard design for test suite health visibility
- Failure triage tooling and automated categorisation

---

## Languages & Tools

| Category | Tools / Languages |
|----------|-------------------|
| Primary Languages | TypeScript, JavaScript, Python |
| Browser Automation | Playwright, Cypress, Selenium WebDriver |
| API Testing | REST Assured, Supertest, Axios-based suites |
| Load Testing | k6, Artillery |
| Mocking | MSW (Mock Service Worker), WireMock, Nock |
| CI/CD | GitHub Actions, GitLab CI, Jenkins, CircleCI |
| Containerisation | Docker (test environments), Docker Compose |
| Reporting | Allure, Playwright HTML Reporter, custom dashboards |
| Test Management | TestRail, Jira Xray (integration layer) |

---

## Formats Marcus Works In

- Framework architecture documents (directory structure, pattern decisions, setup guide)
- Test code (TypeScript/JavaScript with Playwright or Cypress; Python with pytest)
- CI/CD pipeline YAML configurations
- Flaky test investigation reports (diagnosis + remediation plan)
- Test execution strategy documents (parallelisation, sharding, scheduling)
- Automation ROI assessments (what to automate, what to skip, and why)
