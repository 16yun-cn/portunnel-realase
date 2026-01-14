# Changelog

All notable changes to this project will be documented in this file.

## v0.5

### Added
- Bypass routing with `PROXY`/`DIRECT`/`REJECT` policies and first-match rules.
- DNS local resolution with in-memory LRU cache (TTL-based) for IP-CIDR matching.
- Routing rules stats and `GET /api/routing/rules` API.
- Routing rules UI page (`/routing`) with hit counts.
- Proxy list API `GET /api/v1/get_proxy` (text/JSON) and `remoteProxy.ttl`.

### Changed
- Documentation updated for routing, DNS, and management APIs.

## Unreleased

### Added
- CLI support for `--config`/`-f` and credential update modes (`--username`, `--password`, `--set-credentials`).
- Config `proxy.connectForceClose` to control main-port forced connection close (on/off/auto).
