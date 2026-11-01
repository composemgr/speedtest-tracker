## 👋 Welcome to speedtest-tracker 🚀

Track your internet speed over time

## 📋 Description

Speedtest Tracker records scheduled speedtest results on a schedule and keeps
the history so you can see how your connection has changed over weeks or
months. It ships a public results dashboard and an admin panel where the test
schedule and server list are configured.

This is a single-service stack on SQLite, so there is no separate database to
run. The SQLite file lives in the config volume alongside the admin panel
state.

## 🚀 Services

- **app**: lscr.io/linuxserver/speedtest-tracker:latest

### Infrastructure Components

None. Speedtest Tracker stores everything in SQLite inside `/config`.

## 📦 Installation

### Option 1: Quick Install
```bash
curl -q -LSsf "https://raw.githubusercontent.com/composemgr/speedtest-tracker/main/docker-compose.yaml" -o compose.yml
```

### Option 2: Git Clone
```bash
git clone "https://github.com/composemgr/speedtest-tracker" ~/.local/srv/docker/speedtest-tracker
cd ~/.local/srv/docker/speedtest-tracker
cp default.env .env
docker compose --env-file .env up -d
```

### Option 3: Using composemgr
```bash
composemgr install speedtest-tracker
```

## 🔧 Configuration

### Environment Variables

```shell
TZ=America/New_York
BASE_DOMAIN_NAME=example.com
BASE_HOST_NAME=speedtest.example.com
SERVICE_USER=1000
SERVICE_GROUP=1000
APP_ORG_NAME=Speedtest Tracker
APP_ADMIN_EMAIL=admin@example.com
APP_ADMIN_PASS=replace-with-a-random-password
APP_SECRET_KEY=replace-with-a-random-32-byte-hex-string
PUBLIC_DASHBOARD=true
SPEEDTEST_SCHEDULE=0 */4 * * *
```

See `docker-compose.yaml` for complete list of configurable options.

Notable settings:

- `SPEEDTEST_SCHEDULE` is a cron expression for when the test runs; the
  default here runs it every four hours
- `SPEEDTEST_SERVERS` is an optional comma-separated list of Ookla server IDs
  to restrict the tests to. Leave it empty to let Speedtest Tracker pick
- `PRUNE_RESULTS_OLDER_THAN` prunes results older than the given number of
  days. `0` keeps everything
- `PUBLIC_DASHBOARD` controls whether the results dashboard is reachable
  without logging in

### Environment Files

`default.env.sample` holds the full set of variables and is copied to `.env`
before starting the stack:

```bash
cp default.env .env
docker compose --env-file .env up -d
```

`app.env.sample` holds the app-specific overrides and is copied to `app.env`:

```bash
cp app.env.sample app.env
```

`composemgr up` applies `app.env` and `default.env` automatically. Raw
`docker compose` commands need `--env-file .env` to pick up the same values.

### Reverse Proxy

Three standalone compose files ship in this repo:

| File | Use |
|------|-----|
| `docker-compose.yaml` | Default, published on `172.17.0.1:64351` |
| `docker-compose.traefik.yaml` | Behind an existing external `traefik` network |
| `docker-compose.tunnel.yaml` | Behind an existing external `cloudflare` tunnel network |

```bash
docker compose -f docker-compose.traefik.yaml --env-file .env up -d
```

Only the `app` service carries the proxy labels and joins the external
network.

## 🌐 Access

- **Web Interface**: http://172.17.0.1:64351

Log in at `/login` with `APP_ADMIN_EMAIL` and `APP_ADMIN_PASS`. When
`PUBLIC_DASHBOARD` is enabled, `/` shows the results without logging in.

## 📂 Volumes

- `./volumes/config/speedtest-tracker` - Application config and the SQLite
  database, mounted at `/config`

## 🔐 Security

- Change `APP_ADMIN_PASS` and generate a real `APP_SECRET_KEY` before the
  first start. The defaults are `admin@example.com` / `password`, and the
  secret only protects sessions, so a shared default means shared sessions
- Generate a secret with `openssl rand -hex 32`
- The admin credentials and `APP_KEY` are only applied when the config
  volume is empty, so changing them later means editing the values already
  in the admin panel
- Set `PUBLIC_DASHBOARD=false` if the results are not meant to be public;
  with it on, anyone who can reach the port can read your speedtest history
- Set `SERVICE_USER` and `SERVICE_GROUP` to the uid and gid that own
  `./volumes/config/speedtest-tracker`, otherwise the container writes the
  SQLite file as a different owner and you can lose access to it
- Use HTTPS via the reverse proxy in production
- Regularly update Docker images for security patches
- Backup your data regularly

## 🔍 Logging

```shell
docker compose --env-file .env logs -f app
```

## 🛠️ Management

```bash
# Start services
docker compose --env-file .env up -d

# Stop services
docker compose --env-file .env down

# Update to latest images
docker compose --env-file .env pull && docker compose --env-file .env up -d

# View logs
docker compose --env-file .env logs -f

# Restart services
docker compose --env-file .env restart
```

## 📋 Requirements

- Docker Engine 20.10+
- Docker Compose V2+
- Outbound internet access, Speedtest Tracker reaches Ookla's servers to run
  the tests

## 🤝 Author

🤖 casjay: [Github](https://github.com/casjay) 🤖  
🦄 composemgr: [Github](https://github.com/composemgr) 🦄
