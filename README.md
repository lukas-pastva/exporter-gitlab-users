# exporter-gitlab-users

A Prometheus exporter that scrapes GitLab user and repository metrics (including commits and, optionally, line-level statistics) and exposes them on `/metrics` for consumption by Prometheus.

## Features

- **User counts**: total, active (made commits), and passive (signed in but no commits)  
- **Repository counts**: total in groups, plus personal projects  
- **Commit metrics**: per-user and per-repository commit counts  
- **Optional line stats**: lines added/removed per user and per repository (`ENABLE_LINE_STATS=true`)  
- **Heartbeat & scrape stats**: internal metrics about scraping duration, request counts, etc.  


### Exporter-timespan config

```yaml
# config.yaml
metrics:
  - name: gitlab_total_commits
    aggregation: max  # or avg
    time_window:
      start: "09:00"
      end: "12:00"
    start_date: "2024-10-10"
  - name: another_metric
    aggregation: avg
    time_window:
      start: "10:00"
      end: "14:00"
    start_date: "2024-10-01"
```