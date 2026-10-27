# Stage 01 — Relevé

Images construites par le workflow initial, scannées le 2026-10-27 avec Trivy (sévérités HIGH et CRITICAL).

## Synthèse

| | web | votes |
|---|---|---|
| Image | ghcr.io/estelle-tamalet/lab404-web:latest | ghcr.io/estelle-tamalet/lab404-votes:latest |
| Digest | sha256:4000f99936deacb5253c4399aa170aeb1281e1cfa6e7619888554fffe68b8908 | sha256:c393d09c1be2801f24b7e352d816eb30e63e55b3bd40bcbd9af7903c24400b73 |
| Taille | 1 149 Mo | 1 291 Mo |
| Utilisateur | non défini (root) | non défini (root) |
| Base | python:3.14 (Debian 13, Python 3.14.10) | golang:1.26.10 (Debian 13) |
| Build | single-stage | single-stage |
| Shell | oui (/bin/sh) | oui (/bin/sh) |
| Toolchain | gcc, g++, make, git… | Go, gcc, g++, make, git… |
| Composants SBOM | 519 (469 deb, 49 pypi) | 261 (207 deb, 41 golang) |
| Vulnérabilités OS | 467 (450 HIGH, 17 CRITICAL) | 149 (146 HIGH, 3 CRITICAL) |
| Vulnérabilités applicatives | 4 HIGH (Python) | 7 HIGH (binaire Go) |

## Historique des couches

### web

| Couche | Instruction |
|---|---|
| Base | debian trixie + buildpack-deps (curl, git, gcc, autoconf…) |
| Base | Python 3.14.10 compilé depuis les sources |
| App | WORKDIR /app |
| App | COPY requirements.txt |
| App | RUN pip install --no-cache-dir -r requirements.txt |
| App | COPY . . |
| App | EXPOSE 8080 |
| App | CMD opentelemetry-instrument gunicorn --bind=0.0.0.0:8080 app:app |

### votes

| Couche | Instruction |
|---|---|
| Base | debian trixie + buildpack-deps (curl, git, gcc, g++, make…) |
| Base | Go 1.26.10 |
| App | WORKDIR /app |
| App | COPY go.mod go.sum |
| App | RUN go mod download |
| App | COPY . . |
| App | RUN go build -o votes . |
| App | EXPOSE 8081 |
| App | CMD ./votes |

## Vulnérabilités applicatives

| Image | Paquet | Version | Corrigé en |
|---|---|---|---|
| web | msgpack | 1.1.2 | 1.2.1 |
| web | setuptools | 70.3.0 | 78.1.1 |
| web | urllib3 | 2.7.0 | 2.8.0 |
| votes | golang.org/x/net | v0.55.0 | 0.56.0 |
| votes | golang.org/x/text | v0.37.0 | 0.41.0 |
| votes | google.golang.org/grpc | v1.81.1 | 1.83.1 |

## Fichiers

- `stage01/web.cdx.json` : SBOM CycloneDX de web
- `stage01/votes.cdx.json` : SBOM CycloneDX de votes