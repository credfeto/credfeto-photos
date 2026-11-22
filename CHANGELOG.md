# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

<!--
Please ADD ALL Changes to the UNRELEASED SECTION and not a specific release
-->

## [Unreleased]
### Security
- Host firewall lockdown for ingress and egress (`firewall/configure`, run by `update`): an nftables table lets only the reverse proxies reach the published Immich port, and firewalld policies limit what the host and the containers may connect to, with CDN-fronted destinations kept in ipsets refreshed every 10 minutes. `photos-firewall-sync.timer` re-applies it when the firewall config changes, and the daily `Repo: Update egress ranges` workflow refreshes the GitHub and Cloudflare ranges. Observation mode (log unmatched traffic, drop nothing) until `PHOTOS_FIREWALL_ENFORCE=true`
### Added
- Added .ai-instructions and ai/local/index.md from cs-template standard
### Fixed
### Changed
- Enabled immich-server healthcheck, matching upstream's v3.0.0 docker-compose.yml
- Watchtower now cleans up old images and uses an overridable WATCHTOWER_SCHEDULE (default 0 48 * * * *) instead of a poll interval
### Deprecated
### Removed
### Deployment Changes

<!--
Releases that have at least been deployed to staging, BUT NOT necessarily released to live.  Changes should be moved from [Unreleased] into here as they are merged into the appropriate release branch
-->

## [0.0.0] - Project created