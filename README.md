# exporter-gitlab-users

A Prometheus exporter that scrapes GitLab user and repository metrics (including commits and, optionally, line-level statistics) and exposes them on `/metrics` for consumption by Prometheus.

## Features

- **User counts**: total, active (made commits), and passive (signed in but no commits)  
- **Repository counts**: total in groups, plus personal projects  
- **Commit metrics**: per-user and per-repository commit counts  
- **Optional line stats**: lines added/removed per user and per repository (`ENABLE_LINE_STATS=true`)  
- **Heartbeat & scrape stats**: internal metrics about scraping duration, request counts, etc.  

## Prerequisites

- GitLab personal access token with **api** scope  
- Docker (for containerized builds) or Go 1.19+ (for local build)  
- No other external dependencies (binaries like `jq`, `yq`, `kubectl` are bundled in the Docker image)  

## Installation

Clone the repo and switch into the `src` directory:

    git clone https://github.com/lukaspastva/exporter-gitlab-users.git
    cd exporter-gitlab-users/src

### Build Locally

    go build -o exporter-gitlab-users .

### Build & Push Docker Image

    docker build -f Dockerfile -t lukaspastva/exporter-gitlab-users:latest .
    docker tag lukaspastva/exporter-gitlab-users:latest lukaspastva/exporter-gitlab-users:$(git rev-parse --short HEAD)
    docker push lukaspastva/exporter-gitlab-users:latest

## Configuration

The exporter is controlled via environment variables:

| Variable            | Description                                                                                       | Default               |
|---------------------|---------------------------------------------------------------------------------------------------|-----------------------|
| `GITLAB_URL`        | Base URL of your GitLab instance                                                                  | `https://gitlab.com`  |
| `PRIVATE_TOKEN`     | **(required)** GitLab personal access token with `api` scope                                      | —                     |
| `GROUP_ID`          | Group ID to scrape (only used if `SCRAPE_MODE=group`)                                             | `""`                  |
| `SCRAPE_MODE`       | `group` (only the specified group & subgroups) or `all` (all groups & users across the instance) | `group`               |
| `START_DATE`        | Relative start date for scraping (e.g. `"30 days ago"`)                                           | `30 days ago`         |
| `ENABLE_LINE_STATS` | `true` to collect lines added/removed per commit (may slow scrapes)                               | `false`               |
| `RUN_AT_HOUR`       | Hour (0–23) to trigger the daily scrape                                                           | `1`                   |
| `RUN_BEFORE_MINUTE` | Minute window after `RUN_AT_HOUR` during which the scrape will run                                | `5`                   |

## Usage

### Running Locally

    export PRIVATE_TOKEN="YOUR_TOKEN"
    export GROUP_ID="12345"
    ./exporter-gitlab-users

Metrics endpoint:

    http://localhost:9199/metrics

### Running in Docker

    docker run -d \
      --name exporter-gitlab-users \
      -e PRIVATE_TOKEN="YOUR_TOKEN" \
      -e GROUP_ID="12345" \
      -p 9199:9199 \
      lukaspastva/exporter-gitlab-users:latest

## Exposed Metrics

All metrics use the `gitlab_` prefix:

- **Users & repos**  
  - `gitlab_total_users`, `gitlab_active_users`, `gitlab_passive_users`  
  - `gitlab_total_repositories`, `gitlab_total_user_projects`  
- **Commits**  
  - `gitlab_total_commits`  
  - `gitlab_user_commits{user_email="…"}`  
  - `gitlab_repo_commits{project_id="…",project_name="…"}`  
- **Optional line stats**  
  - `gitlab_user_lines_added`, `gitlab_user_lines_removed`  
  - `gitlab_repo_lines_added`, `gitlab_repo_lines_removed`  
- **Scrape & heartbeat**  
  - `gitlab_heart_beat`, `gitlab_scrape_time`, `gitlab_curl_requests_total`, `gitlab_curl_requests_failed`  

See `src/bin/metrics.sh` and `src/main.go` for implementation details.

## Grafana Dashboard

A sample dashboard is provided at `src/bin/grafana-dashboard.json`. To import:

1. In Grafana, go to **Dashboards → Import**  
2. Upload `grafana-dashboard.json` and select your Prometheus data source  

## Continuous Integration

A GitHub Actions workflow (`.github/workflows/build.yaml`) automatically builds and pushes a Docker image to Docker Hub on every push to `main`.

## Dependency Management

Dependabot is set up (`.github/dependabot.yaml`) to check Go module updates on a **monthly** schedule.

## Contributing

Bug reports and feature requests use the templates in `.github/ISSUE_TEMPLATE/`. Pull requests are very welcome!

## Funding

This project is funded via GitHub Sponsors (see `.github/FUNDING.yml`). If you’d like to support ongoing development, please consider sponsoring **lukaspastva**.
