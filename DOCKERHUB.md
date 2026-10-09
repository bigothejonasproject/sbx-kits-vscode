# Publishing the image to Docker Hub

The kit's `spec.yaml` pulls `oiradoc/sbx-vscode`. This image is built from
[`image/Dockerfile`](image/Dockerfile) and published to Docker Hub.

## Build and push manually

```bash
docker login

docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t oiradoc/sbx-vscode:latest \
  --push \
  image/
```

## Build and push via CI

[`.github/workflows/build.yml`](.github/workflows/build.yml) validates the kit
with `sbx kit validate .`, builds a multi-arch image, and on every push to
`main` (and on manual dispatch) pushes both the image and the kit artifact.
Pull requests validate and build but do not push.

Add these repository secrets under **Settings → Secrets and variables → Actions**:

| Secret | Value |
|--------|-------|
| `DOCKERHUB_USERNAME` | `oiradoc` |
| `DOCKERHUB_TOKEN`    | A Docker Hub **access token** (Account Settings → Security → New Access Token). |

## Tags

| Tag | Description |
|-----|-------------|
| `latest` | Current `main` build (VS Code + Claude Code extension). |

To pin a version, add a `git tag` step to the workflow and tag the image with the
release version alongside `latest`.

## Publishing the kit as an OCI artifact

Separately from the container image, the whole kit (spec + files) can be pushed
to a registry so users can reference it without cloning the repo. Use a
**different repo** from the image so the two artifacts don't collide:

```bash
sbx kit push . docker.io/oiradoc/sbx-vscode-kit:latest
```

Note the full registry host (`docker.io/…`) is required. Users then run:

```bash
sbx run docker.io/oiradoc/sbx-vscode-kit:latest .
```

This is a sandbox kit, so it goes in the agent position. `--kit` only accepts
mixins.

Inspect a published kit with `sbx kit inspect docker.io/oiradoc/sbx-vscode-kit:latest`.
