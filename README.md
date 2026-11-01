# credfeto-photos

Immich Docker Compose config

## Watchtower update schedule

Watchtower removes old images after updating (`WATCHTOWER_CLEANUP=true`) and updates on the cron schedule `WATCHTOWER_SCHEDULE`.

- Default: `0 48 * * * *` (hourly, at 48 minutes past the hour).
- The format has 6 fields, seconds first: `second minute hour day month weekday`.
- To override it, set `WATCHTOWER_SCHEDULE` in the project's `.env`, for example `WATCHTOWER_SCHEDULE="0 7 * * * *"`.
