# status

[![CI](https://github.com/sogeor/status/actions/workflows/lint.yml/badge.svg)](https://github.com/sogeor/status/actions/workflows/lint.yml)
[![Release](https://img.shields.io/github/v/release/sogeor/status?sort=semver)](https://github.com/sogeor/status/releases)
[![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/sogeor/status/badge)](https://scorecard.dev/viewer/?uri=github.com/sogeor/status)
[![License: source-available](https://img.shields.io/badge/license-source--available-informational)](LICENSE)

> Public status page of Sogeor services: Gatus configuration for status.sogeor.com

The page at <https://status.sogeor.com> shows whether Sogeor servers, websites, DNS and email routing work. It is served by [Gatus](https://github.com/TwiN/gatus) on a separate small server, so it keeps working when the main infrastructure does not, and failures are sent to the maintainers on Telegram.

## Table of Contents

- [Security](#security)
- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [Development](#development)
- [Maintainers](#maintainers)
- [Contributing](#contributing)
- [License](#license)

## Security

Report vulnerabilities privately as described in the [security policy](https://github.com/sogeor/.github/blob/HEAD/SECURITY.md). This repository holds no secrets: the Telegram bot token and chat ID are kept encrypted in [sogeor/ansible](https://github.com/sogeor/ansible) and passed to Gatus as environment variables.

The page is public, so endpoints hide hostnames, ports and URLs, and server checks also hide error messages. Do not add checks whose name or result would reveal internal addresses.

## Background

| Path | Contents |
| --- | --- |
| `config.yaml` | Gatus configuration: endpoints, alerting, page appearance, announcements |
| `assets/` | logo and favicon served next to the page (copies from [sogeor/brand](https://github.com/sogeor/brand)) |

How a change reaches the page:

```text
pull request ──▶ checks: lint, Gatus starts with config.yaml
merge to master ──▶ repository_dispatch "status-config" ──▶ sogeor/ansible deploy
                                                                │
status.sogeor.com ◀── Caddy (TLS) ◀── Gatus (127.0.0.1:8080) ◀──┘ config.yaml, assets/
```

Gatus and Caddy run as Podman containers pinned by digest; the role `status_vps` in sogeor/ansible installs them and copies this repository to the server.

## Install

Nothing to install: merging to `master` deploys the configuration. The deployment needs the `sogeor-release-bot` GitHub App (variable `RELEASE_APP_ID`, secret `RELEASE_APP_PRIVATE_KEY`) to send the dispatch event to sogeor/ansible.

## Usage

Add an endpoint to `config.yaml`:

```yaml
endpoints:
  - name: example.sogeor.com
    group: Website
    url: https://example.sogeor.com
    interval: 5m
    conditions:
      - "[STATUS] == 200"
      - "[CERTIFICATE_EXPIRATION] > 336h"
    alerts:
      - type: telegram
    ui: *public
```

Announce an outage or maintenance:

```yaml
announcements:
  - timestamp: 2026-11-01T10:00:00Z
    type: warning
    message: Maintenance of odin from 10:00 to 11:00 UTC.
```

Condition syntax and endpoint types are described in the [Gatus documentation](https://github.com/TwiN/gatus#configuration).

## Development

```bash
pre-commit install          # gitleaks, linters, secret checks
pre-commit run --all-files
docker run --rm --network host -e TELEGRAM_BOT_TOKEN=test -e TELEGRAM_CHAT_ID=0 \
  -v "$PWD/config.yaml:/config/config.yaml:ro" ghcr.io/twin/gatus:v5.37.0
```

Then open <http://127.0.0.1:8080>. Pull requests start Gatus with the configuration in CI and fail if it does not become healthy.

## Maintainers

[Sogeor maintainers](https://github.com/orgs/sogeor/teams/maintainers)

## Contributing

Issues and pull requests are welcome. Read the [contributing guidelines](https://github.com/sogeor/.github/blob/HEAD/CONTRIBUTING.md) and the [Code of Conduct](https://github.com/sogeor/.github/blob/HEAD/CODE_OF_CONDUCT.md). Pull request titles follow [Conventional Commits](https://www.conventionalcommits.org/); first-time contributors sign the [CLA](https://github.com/sogeor/.github/blob/HEAD/CLA.md).

## License

© 2026 Georgii Sopin. [Sogeor License 1.0](LICENSE) (English and Russian; the Russian version prevails) — you may read the code and contribute, but not copy, redistribute or reuse it. The name and the mark are trademarks; see [sogeor/brand](https://github.com/sogeor/brand/blob/HEAD/TRADEMARKS.md).
