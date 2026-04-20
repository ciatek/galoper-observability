# galoper-observability

Docker Compose and configs for Galoper’s shared observability plane (hub + edge agents). Serves the broader Galoper ecosystem—ERP, POS, and other components—not a single app.

## TODO
- [x] Replace Promtail And Grafana Agent with Aloy
- [X] Add Grafana OnCall
- [X] Replace Prometheus with Mimir
- [ ] Add frontend dashboards in grafana
- [ ] Add Google Cloud billing dashboard
- [ ] Add Monitoring Dashboard for GCS
- [ ] Add blob storage in gcs to save logs and metrics
- [ ] Upgrade to loki v3
- [ ] Fix Nginx Dashboard
- [ ] Fix Containers logs
- [ ] Fix syslog-ng
- [x] Add Pyroscope
- [ ] Fix prometheus dashboard
- [ ] Link the 4 data types tracing logging metrics and profiles
- [ ] A Dockerfile for oncall_engine that auto creates .env file and adds it the container
- [ ] Add Mongodb dashboard and integration
- [ ] Move alerting to Grafana Alerts
- [ ] Explore k6 integration
- [ ] Collect and expose metrics related to the observability stack (alloy metrics, loki metrics, tempo metrics...)
- [x] Implement Grafana Faro
- [ ] Fix percent of 2xx requests in dashboard to ignore 401
- [ ] Link backend frontend and faro sessions and add them link them betweenn the logs, profiles, metrics