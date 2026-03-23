# SKILLS.md — Elena Vasquez

## Core Competencies

### 1. Performance Testing Strategy & Planning
- Defining performance requirements, SLAs, and acceptance criteria
- Test type selection: load, stress, spike, soak/endurance, scalability, volume
- Traffic profile modelling from real analytics data
- Risk-based performance testing prioritisation
- Entry/exit criteria for performance sign-off

### 2. Load & Stress Testing Execution
- k6 script design: realistic user journeys, staged load profiles, custom metrics
- Apache JMeter test plan design and distributed execution
- Spike testing for sudden traffic surge scenarios (product launches, marketing events)
- Endurance testing for memory leaks and long-run degradation
- Breaking point identification and system recovery behaviour

### 3. Web Performance & Core Web Vitals
- LCP (Largest Contentful Paint) measurement and optimisation
- FID / INP (Interaction to Next Paint) analysis and reduction
- CLS (Cumulative Layout Shift) diagnosis and elimination
- Lighthouse and WebPageTest analysis and interpretation
- Real User Monitoring (RUM) data interpretation
- Image and asset optimisation, CDN configuration review
- JavaScript bundle analysis and code splitting strategy

### 4. Backend & API Performance
- API response time profiling: TTFB, processing time, database query time
- Database query performance analysis (slow query logs, execution plans)
- Connection pool sizing and optimisation
- Cache effectiveness analysis and caching strategy design
- Microservices latency profiling and dependency mapping

### 5. Bottleneck Identification & Root Cause Analysis
- Systematic variable isolation (application layer vs. database vs. infrastructure)
- CPU, memory, network, and disk I/O profiling under load
- Thread pool exhaustion and connection saturation diagnosis
- Garbage collection behaviour analysis (JVM and Node.js)
- Query plan analysis and index effectiveness evaluation

### 6. Capacity Planning
- Traffic growth modelling from historical data and projections
- Infrastructure sizing recommendations with cost-performance trade-offs
- Auto-scaling policy validation and break-even analysis
- Multi-region latency planning

### 7. CI/CD Performance Integration
- Performance regression detection in deployment pipelines
- k6 and Grafana k6 Cloud integration with GitHub Actions
- Performance budgets enforcement (API response time, bundle size, Core Web Vitals thresholds)
- Automated performance trend reporting and alerting

### 8. Monitoring & Observability
- APM tool configuration for performance visibility (Datadog, New Relic, Elastic APM)
- Grafana dashboard design for load test results and production performance
- SLI/SLO/SLA definition and alerting configuration
- Distributed tracing setup for latency attribution in microservices

---

## Tools & Platforms

| Category | Tools |
|----------|-------|
| Load Testing | k6, Apache JMeter, Artillery, Gatling |
| Web Performance | Lighthouse, WebPageTest, Chrome DevTools, PageSpeed Insights |
| APM / Monitoring | Datadog, New Relic, Elastic APM, Grafana |
| Profiling | Node.js Profiler, py-spy, JVM Flight Recorder |
| Database Analysis | EXPLAIN ANALYZE (PostgreSQL), MySQL slow query log, MongoDB Atlas profiler |
| Infrastructure | AWS CloudWatch, GCP Monitoring, InfluxDB + Grafana |
| CI/CD | GitHub Actions, GitLab CI (k6 integration) |

---

## Formats Elena Works In

- Performance test plans (objectives, test types, load profiles, thresholds, schedule)
- k6 test scripts (TypeScript/JavaScript, staged load, custom metrics)
- Performance analysis reports (baseline, test results, bottleneck diagnosis, recommendations)
- Before/after comparison reports with statistical confidence
- Capacity planning documents with growth scenarios and infrastructure sizing
- Performance budget specifications (frontend and API)
- SLI/SLO definition documents
