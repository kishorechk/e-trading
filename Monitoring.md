Technical Design: Splunk Monitoring for EGB E-Trading Platform

Status: Draft for Review
Scope: Application observability, infrastructure monitoring, alerting and production support

1. Objective

Establish a consistent, end-to-end monitoring solution for the EGB electronic trading platform using Splunk and OpenTelemetry.

The solution must provide visibility into service availability, pricing performance, RFQ lifecycle, market-data freshness, WebSocket delivery and infrastructure health.

2. Application Architecture

Component	Technology	Responsibilities
Trader UI	React	Trader interactions, RFQ submission and price/quote display
Backend for Frontend (BFF)	C# / Windows	UI APIs, WebSocket connections and delivery of pricing and market data
Pricing Engine	Java / Linux	Pricing calculations and quote generation
RFQ Engine	Java / Linux	RFQ processing and lifecycle management
Static Data Service	C# / Windows	Bloomberg reference and instrument data
Market Data Service	C# / Windows	Refinitiv market-data ingestion and distribution

3. Proposed Monitoring Architecture

The solution will use a common telemetry standard across Windows/.NET and Linux/Java services.

* Logs: Structured JSON application logs collected using Splunk Universal Forwarder or supported OpenTelemetry collection pipelines.
* Metrics: Host, runtime and application-level metrics collected using supported exporters and instrumentation.
* Distributed tracing: OpenTelemetry .NET and Java instrumentation, supplemented with custom spans for pricing and RFQ operations.
* Frontend monitoring: Browser errors, API latency, WebSocket connection health and price-delivery timing.
* Central monitoring: Splunk Enterprise/Cloud Platform for log analytics, dashboards and alerting. Splunk Observability Cloud may be used for APM, service maps and trace analysis, subject to licence availability.

All telemetry must include consistent service names, environment, version and timestamps.

4. Monitoring Requirements

Area	Key indicators
Service availability	Health checks, service restarts, request success/error rates
Pricing engine	Request throughput, calculation latency (p95/p99), failures and validation errors
RFQ engine	RFQ volumes, outcomes, processing latency, timeouts and stuck states
Market data	Feed connectivity, update freshness, subscription failures, gaps and recovery time
Static data	Request latency, failures, missing instrument mappings, cache health and refresh status
BFF / WebSocket	Active connections, reconnects, delivery latency, queue depth and failed subscriptions
Infrastructure	CPU, memory, disk, network, JVM metrics, .NET runtime and garbage collection
Monitoring pipeline	Collector/forwarder health, ingestion delay and missing telemetry

Market-data freshness thresholds must reflect feed behaviour, instrument liquidity and trading-session requirements. A lack of updates must not automatically be classified as a feed failure.

5. Distributed Tracing and Correlation

Implement end-to-end tracing across the relevant RFQ and pricing workflow.

Illustrative flow:

React UI → C# BFF → Java RFQ Engine → Java Pricing Engine → BFF/WebSocket → React UI

Market-data and static-data dependencies must be instrumented at their actual points of interaction.

Requirements:

* Propagate W3C trace context across supported service boundaries.
* Propagate correlation context explicitly for WebSocket messages and asynchronous operations where required.
* Include trace_id, span_id, rfq_id and quote_id where applicable.
* Record operation duration, outcome and controlled error codes.
* Distinguish pricing calculation latency, RFQ processing latency, market-data lag and trader-perceived latency.
* Configure sampling to balance diagnostic coverage, performance and telemetry cost.

6. Logging Standards

All services must adopt a common structured logging schema.

Required fields, where applicable:

timestamp, service.name, service.version, deployment.environment, host.name, event.name, severity_text, trace_id, span_id, rfq_id, quote_id, duration_ms, outcome and error_code.

Logs must use UTC timestamps, controlled event names and consistent error classifications. Credentials, access tokens and unnecessary customer or trade-sensitive information must not be logged.

High-cardinality identifiers must not be used as unrestricted metric dimensions.

7. Dashboards and Alerting

Dashboards

1. Trading Operations: Overall health, RFQ outcomes, pricing latency, WebSocket sessions and market-data status.
2. Pricing and RFQ: Processing latency, failures, throughput and stuck RFQs.
3. Market and Static Data: Feed health, freshness, reference-data failures and recovery.
4. BFF and Trader UI: API performance, connection stability and price-delivery latency.
5. Infrastructure: Host health, JVM/.NET runtime metrics and service restarts.
6. Production Support: Active alerts, deployments, incidents and telemetry collection health.

Alerting principles

* Critical service unavailability or compromised pricing inputs: immediate escalation according to agreed trading-risk criteria.
* RFQ timeouts, abnormal failure rates and stale data: severity based on impact and persistence.
* Latency and resource alerts: thresholds based on measured baselines and agreed SLOs.
* Telemetry gaps: alert when critical sources stop reporting unexpectedly.
* Include service ownership, severity, relevant dashboard and runbook in each actionable alert.

Final thresholds and severity assignments require agreement with application owners, production support and the trading desk.

8. Implementation Plan

Phase	Deliverables
1. Foundation	Service inventory, structured logging, host monitoring and collection-health dashboards
2. APM	.NET/Java instrumentation, trace propagation and pricing/RFQ latency metrics
3. Trading-specific monitoring	Market-data freshness, static-data validation, WebSocket delivery and RFQ state monitoring
4. Production readiness	Alert tuning, runbooks, access controls, telemetry-overhead testing and failure-scenario validation

9. Review Decisions and Open Questions

1. Is the available Splunk capability limited to Enterprise/Cloud Platform, or does it include Observability Cloud and APM?
2. What are the agreed latency and availability SLOs for pricing, RFQs and trader-facing delivery?
3. What market-data freshness rules apply to Refinitiv feeds and individual instruments?
4. Which transport mechanisms connect the BFF, RFQ engine and pricing engine?
5. What are the required retention, access-control and audit policies for trading telemetry?
6. Which team owns each service, alert and incident runbook?

10. Acceptance Criteria

* All in-scope services have observable health and collection status.
* Support can trace a representative RFQ across the relevant services.
* Pricing latency, RFQ outcomes and market-data freshness are measurable.
* Critical failure scenarios generate actionable alerts.
* Dashboards and runbooks support incident diagnosis.
* Telemetry overhead, access controls and retention comply with production standards.

Recommendation: Approve the common telemetry schema, tracing strategy and trading-specific monitoring requirements before implementing individual dashboards and alerts.
