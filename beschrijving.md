# Docker OpenLDAP

## Inleiding

Dit project biedt een Docker-image voor het draaien van een OpenLDAP server (versie 2.4.57). Het is een fork van `osixia/openldap` en biedt een productie-klare LDAP-server met ondersteuning voor TLS, replicatie en uitgebreide configuratiemogelijkheden.

## Samenvatting

- **Image naam**: `reconi/openldap`
- **OpenLDAP versie**: 2.4.57
- **Basis**: `osixia/light-baseimage` (Debian 10 Buster)

### Snel starten

```bash
# Bouwen
make build

# Draaien met docker-compose
podman compose -f example/docker-compose.yml up -d
```

### Belangrijke mappen

| Map | Doel |
|-----|------|
| `image/` | Docker image bronbestanden |
| `example/` | Voorbeeldconfiguraties en docker-compose |
| `test/` | Automatische tests (bats) |

### Standaard instellingen

- **Host:**: localhost
- **Base:**: dc=example,dc=org
- **Poorten**: 389 (LDAP), 636 (LDAPS)
- **Username**: cn=admin,dc=example,dc=org
- **Admin wachtwoord**: `admin`

Overig:
- **Domein**: `example.org`
- **phpLDAPadmin**: http://localhost:8080/