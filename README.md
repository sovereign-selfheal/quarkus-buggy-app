# quarkus-buggy-app

Quarkus 3 REST service with intentional, randomly-triggered failures, used as the target
of the `triage-agent` demo (part of the *Sovereign, Self-Healing Platform* demo).

| Endpoint | Failure | Rate | HTTP code |
|---|---|---|---|
| `GET /api/products` | `NullPointerException` | 30% | 500 |
| `GET /api/orders` | 3-second sleep | 20% | 200 (slow) |
| `GET /api/inventory` | `ServiceUnavailable` | 40% | 503 |

Exposes `/q/metrics` (Micrometer + Prometheus) and `/q/health` (SmallRye Health). A
built-in `TrafficGenerator` calls all three endpoints every 5 seconds, so metrics keep
moving even without external traffic.

## Build

Built with the Quarkus Jib extension (no Dockerfile needed):

```bash
./mvnw package \
  -Dquarkus.container-image.build=true \
  -Dquarkus.container-image.push=true \
  -Dquarkus.container-image.tag=<git tag> \
  -Dquarkus.container-image.username=<quay-user-or-robot> \
  -Dquarkus.container-image.password=<quay-token-or-password> \
  -DskipTests
```

CI (`.github/workflows/build.yml`) runs on pull requests and pushes to `main` to verify
Maven/tests and image build without pushing. On `v*` tags it pushes with Jib directly to
`quay.io/sovereign-selfheal/quarkus-buggy-app`, using the repository secrets
`QUAY_USERNAME` and `QUAY_PASSWORD`.

## Consumer

Kubernetes manifests (Deployment, Service, Route, ServiceMonitor) and the pinned image
digest live in the `gitops` repo, `components/quarkus-buggy-app/`. This repo only owns
the source and the build. After a new tag is pushed and the image is resolved, open a PR
in `gitops` bumping the digest comment in `components/quarkus-buggy-app/values.yaml`.
