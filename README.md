# CLIProxyAPI Codex–Antigravity compatibility image

This repository builds a patched CLIProxyAPI image from the latest stable upstream release. The patch improves OpenAI Responses compatibility when Codex uses Antigravity-backed Gemini or Claude models.

The patch adds end-to-end support for:

- Codex Desktop `additional_tools` declarations;
- Responses `function`, `custom`, and `namespace` tools;
- `custom_tool_call` and `custom_tool_call_output` history;
- reversible namespace and long/colliding tool-name mappings;
- streaming and non-streaming tool-call events;
- upstream function-call IDs as Responses `call_id` values;
- explicit `response.failed` events when an upstream model stops after reasoning without producing a message or tool call;
- recovery of consecutive trailing reasoning-only items after a completed tool response, keeping Cloud Code requests function-response terminated;
- management model probes inherit the matching Codex provider's proxy setting even when the UI omits `auth_index`, so direct LAN upstreams do not fall back to the global proxy;
- empty-string enum sentinels are removed from Gemini tool schemas, which otherwise rejects the entire GenerateContent request;
- existing Gemini and Claude reasoning-signature behavior.

## Images

GitHub Actions publishes an x86-64 image for `linux/amd64`, matching the target NAS:

```text
ghcr.io/zpvel/cliproxyapi-codex-antigravity:patched-latest
ghcr.io/zpvel/cliproxyapi-codex-antigravity:v7.2.94-patched
```

`patched-latest` follows the newest successfully tested upstream stable release. Exact `vX.Y.Z-patched` tags are retained for rollback.

## Update behavior

The scheduled workflow resolves the latest GitHub Release from [`router-for-me/CLIProxyAPI`](https://github.com/router-for-me/CLIProxyAPI), checks it out from scratch, applies the patches in [`patches/`](patches), runs the full Go test suite, verifies the server build, and only then publishes an image.

If an upstream change conflicts with the patch or a test fails, no new image is published. Existing image tags remain available, so a NAS does not silently receive an untested build.

The workflow can also be run manually with a specific upstream release tag.

## Docker Compose

Use the patched image in the existing CPA service while keeping configuration and authentication data in NAS-mounted volumes. For example:

```yaml
services:
  cli-proxy-api:
    image: ghcr.io/zpvel/cliproxyapi-codex-antigravity:patched-latest
    ports:
      - "8317:8317"
    volumes:
      - ./config.yaml:/CLIProxyAPI/config.yaml:ro
      - ./auths:/root/.cli-proxy-api
      - ./logs:/CLIProxyAPI/logs
    restart: unless-stopped
```

Adapt the volume paths to the existing deployment. Secrets, OAuth files, proxy settings, and API keys are not included in this repository or image.

For maximum reproducibility, deploy an exact `vX.Y.Z-patched` tag and update it after the corresponding workflow succeeds.

## Local verification

The patch is maintained as standard `git format-patch` files. To verify it manually:

```bash
git clone --branch v7.2.94 --depth 1 https://github.com/router-for-me/CLIProxyAPI.git source
git -C source am ../patches/*.patch
cd source
go test ./...
go build -buildvcs=false -o test-output ./cmd/server
```

## License

The automation and patch packaging in this repository are MIT licensed. The patched source and resulting image remain subject to CLIProxyAPI's upstream license and notices.
