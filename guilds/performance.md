---
title: Performance Guild
icon: fas fa-tachometer-alt
layout: page
---

# Performance Guild

## The Reality

Performance work at Beyond is happening constantly, but it is scattered across teams, reactive to production pain, and invisible until something breaks. Engineers across every OBT have spent significant effort on query optimization, memory leak fixes, caching strategies, OOM prevention, and frontend load time improvements, often discovering the same patterns independently or learning hard lessons that could have been shared.

There are no formal SLOs. No performance budgets. No automated regression gates in CI. No shared playbook for profiling, benchmarking, or capacity planning. When a large account trips a slow endpoint or an OOM kills a worker, the fix is ad-hoc and team-scoped. The knowledge stays local.

This guild exists to change that.

---

## Purpose

The Performance Guild brings together engineers and product managers across OBTs to:

- **Establish and enforce SLOs** for critical user-facing endpoints and background jobs
- **Build shared processes** for performance budgets, automated regression testing, and capacity planning
- **Surface cross-team patterns** so lessons learned on one service benefit all services
- **Create visibility** into performance health through dashboards, alerts, and regular reviews
- **Unlock resources** by giving product and engineering leadership the data they need to prioritize performance work alongside feature delivery

---

## In Scope

**SLOs and Performance Budgets:**

- Define response time SLOs for critical API endpoints
- Maintain existing background job SLAs (integration workers: at least one price posting every 24h, one reservation sync every 48h, monitored via [Datadog dashboard](https://app.datadoghq.com/dashboard/v43-e8c-yxg))
- Establish frontend performance budgets (Core Web Vitals, bundle size, time-to-interactive) using Datadog RUM
- Build automated SLO monitoring and alerting in Datadog

**Performance Regression Prevention:**

- Automated performance benchmarks in CI (catch regressions before deploy)
- Load testing infrastructure and regular load test runs
- Query performance analysis with Datadog DBM

**Profiling, Observability, and Tooling:**

- APM instrumentation standards (Datadog spans, business context tags, request-to-query correlation)
- Shared profiling playbooks (Datadog APM, DBM, RUM, postgres pg_stat_statements, ClickHouse query analysis, Nomad vitals monitoring)
- Memory profiling and OOM prevention patterns
- Frontend performance profiling (React DevTools, bundle analysis, Lighthouse)

**Cross-Team Knowledge Sharing:**

- Post-mortems and case studies from performance incidents
- Patterns catalog (multi-level caching, deferred fields, subquery refactors, batch processing, pagination)
- Architecture reviews with a performance lens

**Caching Strategy:**

- Standards for when and how to use the multi-level cache (local LRU + Redis)
- Cache invalidation patterns and tag-based invalidation guidelines
- Frontend cache strategy (TanStack Query stale times, prefetching patterns)

---

## Strategic Alignment

The Performance Guild supports Beyond's engineering strategy through:

- **Enterprise readiness** - Our largest accounts (80k+ listings) expose performance cliffs that smaller accounts never hit. SLOs and scalability testing ensure we grow without degrading.
- **Engineering productivity** - Shared profiling tools, patterns, and playbooks mean engineers spend less time rediscovering known solutions.
- **Product quality** - Fast, responsive UIs directly impact user satisfaction and retention. Performance is a feature.
- **Cost efficiency** - Optimized queries and right-sized infrastructure reduce cloud spend. Memory leak prevention reduces OOM-related restarts and wasted compute.
- **Informed prioritization** - SLO dashboards give PMs and leadership objective data to balance performance investment against feature delivery.

---

## Recent Cross-Team Performance Work

Performance investment is already happening across the organization. The guild's job is to connect these efforts, share the learnings, and build the processes that make this work systematic rather than reactive.

**Insights & Dashboard Optimization** - Leonardo Ostjen, Felipe de Morais, and Mila Antonova have driven multiple rounds of ClickHouse query optimization, multi-level caching, replica routing, and pagination for large-account dashboard performance.

**Grid View Performance** - Joe Girgis and Justin Moon shipped grid pagination, compact grid optimizations, frontend loop optimization, and filter caching to make the pricing grid usable at scale.

**Algo & Pricing Engine** - Diego Pasqualin optimized the FactorsModel with lazy-loaded aggregates and cached properties, reduced geometry database calls, and improved calendar endpoint performance.

**Admin Page Bottlenecks** - Yiorgos Krypotos and Jorge Lima eliminated a 315-second RPC call and 32-second balance recalculation from the user admin page, added indexes, and introduced batch queries with prefetching.

**Unsellable Nights Report** - Felipe Souza and Justin Moon used generators, data stripping, garbage collection, and heap trimming to tame memory usage in a report that was crashing workers.

**Dramatiq Migration** - Pedro Gaspar and Marcos Lamuria 💙 migrated scraping jobs from RQ to Dramatiq, with Lucas Sossai tuning worker configs for throughput and memory efficiency.

**Data Pipeline Scaling** - Pedro Gaspar, Gabriel Medeiros, Jayalakshmi Balasubramaniam, and others have optimized Spark jobs, ClickHouse ingestion DAGs, and data pipeline memory usage across multiple services.

**Frontend Bundle Optimization** - Davi Vieira and Stephanie Hanson introduced code splitting and lazy loading for insights dashboard tabs and menu components.

**Super-Proxy** - Jorge Lima and Marcos Lamuria improved session matching, TLS cipher suites, and added Datadog metrics for the Go proxy service.

**Memory Leak Prevention** - Multiple engineers across teams (Ines Martins, Ahmet Korkmaz, Nikolas Zour, Aaron Navarro, Tamas Simak) have independently tackled OOM issues in integration workers, data pipelines, and background jobs.

**History Fields Infrastructure in ClickHouse** - Jon Skulski 💙 built the infrastructure for history fields in ClickHouse, enabling time-series analytics on listing data without impacting the primary database.

---

## Out of Scope

To keep the guild focused on measurable performance outcomes:

- **Feature design and product decisions** - We advise on performance implications, we don't own product roadmaps
- **Security and compliance** - Important, but addressed by other processes
- **General code quality** - Code review standards are team-scoped; we focus specifically on performance-impacting patterns

---

## Target Audience

Anyone who cares about making Beyond fast and reliable. Engineers across all stacks (backend, frontend, data, infrastructure), tech leads, engineering managers, and product managers who want to understand performance tradeoffs and help prioritize this work on roadmaps.

---

## Meeting Structure

- **Cadence:** Every two weeks
- **Format:** Rotating between SLO reviews, case study deep-dives, and working sessions on tooling/process
- **Outputs:** Action items tracked on the Jira board, learnings shared via Slack
- **Hosting:** Rotating among participants

---

## Slack Channel

**#dev-performance**

Join to share performance wins, report regressions, ask for profiling help, post Datadog dashboards, and coordinate guild meetings.

---

## Project Tracking

[Performance Guild Jira Board](https://beyondpricing.atlassian.net/jira/software/c/projects/PERF/boards/816) for tracking SLO definitions, tooling initiatives, and performance improvement projects.
