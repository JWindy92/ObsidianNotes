#idea/dataeng Build F1 telemetry analysis application

A. Data ingestion and management
- [ ] Download session data through FastF1
- [ ] Discover seasons, events, sessions, drivers, and laps
- [ ] Cache source data to avoid unnecessary downloads
- [ ] Normalize timestamps, units, and identifiers
- [ ] Validate schemas, missing values, and anomalous records
- [ ] Retry failed ingestion and safely rerun completed jobs
- [ ] Store normalized datasets in Parquet
- [ ] Track ingestion status and data provenance

B. Analytical data layer
- [ ] Query datasets using DuckDB
- [ ] Maintain consistent session, lap, telemetry, and weather schemas
- [ ] Filter by season, event, session, driver, and lap
- [ ] Calculate derived telemetry features
- [ ] Support incremental dataset expansion
- [ ] Document data sources, limitations, and field definitions

C. Telemetry analysis engine
- [ ] Compare two drivers or laps
- [ ] Align telemetry by track distance
- [ ] Plot speed, throttle, brake, gear, and RPM where available
- [ ] Calculate lap-time and sector-time differences
- [ ] Calculate cumulative time gained or lost around a lap
- [ ] Identify braking points, acceleration zones, and corner exits
- [ ] Compare available weather and tire/stint context
- [ ] Handle missing channels and incompatible laps explicitly

D. Interactive dashboard
- [ ] Browse sessions and select drivers and laps
- [ ] View synchronized telemetry charts
- [ ] Display a circuit map colored by speed or time delta
- [ ] Select track segments to inspect local performance
- [ ] Compare lap times, sectors, and telemetry side by side
- [ ] Show data-quality warnings and analysis assumptions
- [ ] Export selected analysis results or charts

E. Backend API
- [ ] Expose session and driver metadata
- [ ] Serve lap and telemetry queries
- [ ] Provide comparison and analysis endpoints
- [ ] Validate requests and return structured errors
- [ ] Use typed Go request and response models
- [ ] Include API documentation and integration tests

F. Reliability and operations
- [ ] Run the application through Docker Compose
- [ ] Automate tests and quality checks with GitHub Actions
- [ ] Emit structured logs and useful error messages
- [ ] Track ingestion duration, failures, and data volume
- [ ] Measure API latency and query performance
- [ ] Test duplicate runs, interrupted jobs, and recovery
- [ ] Expose operational metrics through Prometheus
- [ ] Document configuration and deployment

G. Performance and portfolio
- [ ] Support multiple circuits and race weekends
- [ ] Benchmark ingestion and analytical queries
- [ ] Measure storage footprint and processing time
- [ ] Optimize at least one measured bottleneck
- [ ] Provide reproducible setup instructions
- [ ] Publish architecture and data-flow diagrams
- [ ] Include a technical case study with real racing insights
- [ ] Record a short demonstration video