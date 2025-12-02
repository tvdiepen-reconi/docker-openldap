# Copilot Instructions for docker-openldap

## Project Overview

This is a Docker image for running OpenLDAP (version 2.4.57), forked from `osixia/openldap`. The image is based on `osixia/light-baseimage` and provides a production-ready LDAP server with TLS, replication, and extensive configuration options.

## Architecture

### Key Components
- **`image/`** - Docker image source
  - `Dockerfile` - Multi-stage build using Debian Buster (archived repos) with backports
  - `environment/` - YAML-based configuration (`default.yaml` for runtime, `default.startup.yaml` for first-start secrets)
  - `service/slapd/` - OpenLDAP service scripts and assets
    - `startup.sh` - Main initialization script (582 lines) handling bootstrap, TLS, replication
    - `assets/config/bootstrap/` - LDIF and schema files loaded at first start

### Configuration Flow
1. Environment variables defined in YAML files (`image/environment/*.yaml`)
2. `startup.sh` processes variables using `{{ VARIABLE }}` substitution in LDIF files
3. Bootstrap schemas/LDIF from `assets/config/bootstrap/schema/` and `ldif/`
4. Custom configs go in `ldif/custom/` or `schema/custom/` directories

## Build & Test Commands

```bash
# Build image (uses Makefile, image name: reconi/openldap)
make build                    # Standard build
make build-nocache           # Force rebuild without cache
podman build -t reconi/openldap:1.5.0 --rm image  # Direct podman/docker

# Run tests (requires bats - Bash Automated Testing System)
make test

# Full release workflow
make release                 # build -> test -> tag-latest -> push

# Run with docker-compose
podman compose -f example/docker-compose.yml up -d
```

## Important Patterns

### Environment Variables
- **Runtime vars** (`default.yaml`): `LDAP_LOG_LEVEL`, `LDAP_NOFILE`, `DISABLE_CHOWN`
- **Startup-only vars** (`default.startup.yaml`): `LDAP_ADMIN_PASSWORD`, `LDAP_CONFIG_PASSWORD`, `LDAP_DOMAIN` - deleted after first start for security
- Docker secrets supported via `_FILE` suffix: `LDAP_ADMIN_PASSWORD_FILE`

### Extending the Image
See `example/extend-osixia-openldap/`:
```dockerfile
FROM osixia/openldap:1.5.0
ADD bootstrap /container/service/slapd/assets/config/bootstrap
ADD certs /container/service/slapd/assets/certs
ADD environment /container/environment/01-custom
```

### LDIF Template Variables
Bootstrap LDIF files support substitution:
- `{{ LDAP_BASE_DN }}`, `{{ LDAP_DOMAIN }}`, `{{ LDAP_BACKEND }}`
- `{{ LDAP_READONLY_USER_USERNAME }}`, `{{ LDAP_READONLY_USER_PASSWORD_ENCRYPTED }}`

### Volume Paths
- `/var/lib/ldap` - LDAP database files
- `/etc/ldap/slapd.d` - LDAP config files
- `/container/service/slapd/assets/certs/` - TLS certificates

## Windows Development Notes
- The Dockerfile includes `dos2unix` conversion for shell scripts (line ending fix)
- All `.sh` files get `chmod +x` during build
- Use `--copy-service` flag when mounting files to avoid permission issues

## Testing
Tests use [bats-core](https://github.com/bats-core/bats-core). Key test patterns in `test/test.bats`:
- `run_image` with `-e LDAP_TLS=false` for simpler testing
- `wait_process slapd` before LDAP operations
- `ldapsearch` commands validate database state

## Key Files Reference
| File | Purpose |
|------|---------|
| `image/Dockerfile` | Main build definition with Debian archive repo fixes |
| `image/service/slapd/startup.sh` | Core initialization logic |
| `image/environment/default.startup.yaml` | All available startup env vars |
| `example/docker-compose.yml` | Full stack with phpLDAPadmin |
| `Makefile` | Build/test/release automation |
