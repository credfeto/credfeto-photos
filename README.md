# credfeto-photos

Immich Docker Compose config

## Watchtower update schedule

Watchtower removes old images after updating (`WATCHTOWER_CLEANUP=true`) and updates on the cron schedule `WATCHTOWER_SCHEDULE`.

- Default: `0 48 * * * *` (hourly, at 48 minutes past the hour).
- The format has 6 fields, seconds first: `second minute hour day month weekday`.
- To override it, set `WATCHTOWER_SCHEDULE` in the project's `.env`, for example `WATCHTOWER_SCHEDULE="0 7 * * * *"`.

## Host firewall

`update` runs `firewall/configure`, which locks the photos host down for ingress and egress, the same way as the DNS, monitoring and key servers:

| Filter                                 | Traffic                     | Allowed                                                                                                                                                                       |
|----------------------------------------|-----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pho_ingress` (nftables)               | LAN -> published port       | Immich (tcp 2283) from the reverse proxies only, since the web UI and the phone apps use `photos.markridgwell.com` through Traefik                                            |
| `host_out` (nftables `pho_egress`)     | host -> anywhere            | DNS to the fleet's DNS servers, NTP to the router, HTTPS to the proxies, journald uploads to monitoring, GitHub, ghcr.io, Docker Hub, Cloudflare, chaotic-aur, pkgstats, the Arch WKD, ICMP |
| `docker_fwd` (nftables `pho_egress`)   | containers -> outside world | DNS, HTTPS to the proxies, GitHub (Immich's release check), Hugging Face (machine learning model downloads), ghcr.io, Docker Hub and Cloudflare (Watchtower)                 |

SSH to the host is left to the host's base configuration. Postgres, Redis and the machine learning service are only reached over the compose network and publish no ports.

Ingress is an nftables table of its own (`firewall/ingress.nft.template`, loaded by `photos-ingress-firewall.service` before Docker starts) because firewalld accepts Docker's translated traffic before its policies run. Egress is a table of its own too: the whole table, address sets included, is replaced in a single `nft -f` transaction, so a refresh never makes firewalld reload, and a ruleset that fails to load leaves the previous one in place.

Two timers keep it current:

* `photos-egress-refresh.timer` (30 seconds after boot, then every 10 minutes) rebuilds the table's address sets for destinations without a stable address: names resolved from DNS (a name that does not resolve keeps its last good addresses), plus GitHub's and Cloudflare's published ranges (`firewall/github-egress-ranges` and `firewall/cloudflare-egress-ranges`). It only loads the table when it differs from the loaded one.
* `photos-firewall-sync.timer` (every 15 minutes) pulls the checkout and re-runs `firewall/configure` when anything under `firewall/` changed since the commit it last applied. Container changes still wait for `update`.

The `Repo: Update egress ranges` workflow regenerates the two range files daily and commits any change to main, refusing a list that has shrunk by more than a fifth.

`PHOTOS_FIREWALL_ENFORCE` in `.env` is the only switch. Unset or anything other than `true` is observation mode: every rule is in place, but traffic that matches none of them still flows and is logged with the prefix `pho-ingress-unmatched:`, `pho-egress-unmatched:` or `pho-egress-docker-unmatched:` (rate-limited to 30 a minute). Run in observation mode first and check the journal for anything legitimate that is not covered:

```sh
sudo journalctl -k --since -1d | grep -E 'pho-[a-z-]+-unmatched'
```

then set `PHOTOS_FIREWALL_ENFORCE=true` and run `./update`. Check `curl -fsS https://photos.markridgwell.com/api/server/ping` from another host straight after enforcing. A new or changed machine learning model is downloaded on first use, so after changing models in the Immich settings, check the log for new `pho-egress-docker-unmatched:` lines. To see the live egress rules:

```sh
sudo nft list table inet pho_egress
```
