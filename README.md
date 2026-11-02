<div align="center">

<img src="snap/gui/sure.png" alt="Sure logo" width="128" />

# Sure

**Personal finance, self-hosted**

[![sure](https://snapcraft.io/sure/badge.svg)](https://snapcraft.io/sure)
[![Base: core26](https://img.shields.io/badge/Base-core26-2C8C3C?logo=canonical&logoColor=white)](https://snapcraft.io/core26)
![Confinement: strict](https://img.shields.io/badge/Confinement-strict-2C8C3C)
[![Maintainer: sidaliteir](https://img.shields.io/badge/Maintainer-sidaliteir-2C8C3C?logo=snapcraft&logoColor=white)](https://snapcraft.io/publisher/sidaliteir)
![Architectures: amd64 and arm64](https://img.shields.io/badge/Architectures-amd64%20%7C%20arm64-2C8C3C?logo=linux&logoColor=white)

[Getting Started](#getting-started) · [Configure](#configure) · [Limitations](#limitations) · [Build](#build) · [Support](#support) · [License](#license)

Unofficial snap packaging for [Sure](https://github.com/we-promise/sure), a self-hosted personal finance app for accounts, budgets, investments and net worth, the community continuation of Maybe Finance.

</div>

---

## Getting Started

<a href="https://snapcraft.io/sure"><img alt="Get it from the Snap Store" src="https://snapcraft.io/en/dark/install.svg" width="180"></a>

```bash
sudo snap install sure
```

Open [http://localhost:3000](http://localhost:3000) and create your account. The first account becomes the administrator; new sign-ups can be closed or limited to invitations in the admin settings.

The snap is self-contained. It runs four services, which start automatically:

| Service | Role |
| --- | --- |
| `sure.web` | Puma web server; applies database migrations on start |
| `sure.worker` | Sidekiq worker for syncs, imports and scheduled jobs |
| `sure.postgres` | Bundled PostgreSQL 18 database |
| `sure.valkey` | Bundled Valkey (Redis-compatible) server |

PostgreSQL and Valkey listen only on Unix sockets in `/var/snap/sure/common/run`, never on the network, and every service runs as the unprivileged `snap_daemon` user. The web interface listens on this machine only.

### Commands

```bash
sudo sure console                  # Rails console
sudo sure runner 'puts User.count' # any bin/rails command
sudo sure.pg-dump > sure.sql       # back up the database
sudo sure.psql                     # SQL shell on the database
```

## Configure

| Option | Accepted value | Default |
| --- | --- | --- |
| `port` | Integer from `1024` to `65535` | `3000` |
| `host` | IP address or host name to listen on | `127.0.0.1` |

```bash
sudo snap set sure host=0.0.0.0 port=3000
sudo snap get sure
```

Changes restart the web service.

Other upstream settings, such as email (SMTP), the AI assistant, OpenID Connect or market data providers, go in `/var/snap/sure/common/sure.env`, one `VARIABLE=value` per line in shell syntax (see upstream's [.env.example](https://github.com/we-promise/sure/blob/main/.env.example)). Run `sudo snap restart sure` after editing it. The file can also point Sure at an external PostgreSQL (`DB_HOST`, `DB_PORT`, `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`) or Redis (`REDIS_URL`) instead of the bundled ones. The snap options above take precedence over `PORT` and `BINDING` in this file.

Behind a reverse proxy that terminates TLS, add `RAILS_ASSUME_SSL=true` to `sure.env`.

The Rails secret, the Active Record encryption keys (which encrypt tokens and secrets in the database) and the database password are generated at install in `/var/snap/sure/common/secrets.env`, readable by root only. Keep a copy with your backups: encrypted data cannot be read without these keys.

Data lives in `/var/snap/sure/common`: the database in `postgresql/18`, uploaded files in `storage`.

## Limitations

- The web server runs as an unprivileged user and cannot bind ports below 1024; use a reverse proxy to serve Sure on port 80 or 443.
- Remote access also requires the chosen port to be allowed by your firewall.
- The bundled database is PostgreSQL 18. A future snap moving to another major version will need a dump and restore with `sure.pg-dump` and `sure.psql`.
- The backup sidecar and the Docker-based self-update of upstream's compose setup are not included; use `sure.pg-dump` and copy `/var/snap/sure/common/storage` and `secrets.env`.

## Build

```bash
git clone https://github.com/sidalit/sure-snap.git
cd sure-snap
sudo snap install snapcraft --classic
snapcraft pack
```

## Support

- **Installation or snap issues:** [open a packaging issue](https://github.com/sidalit/sure-snap/issues).
- **Application bugs:** [report them upstream](https://github.com/we-promise/sure/issues).

## License

Snap packaging: [MIT](https://github.com/sidalit/sure-snap/blob/main/LICENSE).
