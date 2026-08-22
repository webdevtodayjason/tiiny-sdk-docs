# Errors

Tiiny returns errors in three different shapes depending on which endpoint you call. The shape follows the protocol you are speaking, not the device.

---

## Before anything else: the 24-hour key

The most common failure against a working device is a `401` on an integration that worked yesterday.

**The auth key expires 24 hours after it is issued.** `pcsvr` is configured with `ExpirationHours: 24`. Any client holding a pasted key — an OpenAI SDK client, a coding agent, a chat UI, a cron job — starts failing after that window.

Re-key with:

```bash
tiiny auth key
```

If you are building anything that runs unattended, fetch the key at startup rather than baking it into config, and treat `401` as "re-authenticate" rather than "credentials are wrong".

---

## Error shapes by endpoint

### OpenAI-compatible (`openai.api.tiiny`)

```json
{
  "error": {
    "message": "Invalid API key.",
    "type": "invalid_request_error",
    "code": "invalid_api_key"
  }
}
```

| Status | Meaning | Common cause |
|---|---|---|
| `401` | Unauthorized | Missing key, malformed key, or **expired key** |
| `404` | Not Found | Model ID does not exist, or is not loaded |
| `422` | Unprocessable Entity | Missing required fields or invalid values |
| `429` | Too Many Requests | Rate limited — back off and retry |
| `500` | Internal Server Error | Inference backend error |

### Anthropic (`anthropic.api.tiiny`)

```json
{
  "type": "error",
  "error": {
    "type": "authentication_error",
    "message": "..."
  }
}
```

| Status | Type | Common cause |
|---|---|---|
| `400` | `invalid_request_error` | Malformed body or missing required fields |
| `401` | `authentication_error` | Missing or invalid key, or **expired key** |
| `403` | `permission_error` | Key lacks permission for the resource |
| `404` | `not_found_error` | Model ID does not exist or is not loaded |
| `429` | `rate_limit_error` | Rate limited |
| `500` | `api_error` | Internal server error |
| `529` | `overloaded_error` | Temporarily overloaded — retry after a short delay |

### Ollama (`ollama.api.tiiny`)

```json
{ "error": "model not found" }
```

| Status | Common cause |
|---|---|
| `400` | Malformed body or invalid parameter values |
| `404` | Model ID does not exist or is not loaded |
| `500` | Internal server error |

### Native endpoints (`p8800.api.tiiny`, `kb.tiiny.local`, `connector.api.tiiny`)

| Status | Common cause |
|---|---|
| `401` | Missing or invalid key, or **expired key** |
| `404` | Model ID does not exist or is not loaded |
| `422` | Missing required fields or unsupported file format |
| `500` | Internal server error |

<!--
  DOC-DEBT: The native error *body* format is unconfirmed. The published
  inference-tiiny-sdk page carries "Error response format pending engineering
  confirmation" and the status table above is inherited from it unverified.
  Confirm against a live device before publishing. Unverified against hardware.
-->

---

## Diagnosing by symptom

**`No Tiiny device connected.`**
No config file, or the stored device is unreachable. Run `tiiny scan`, then `tiiny connect <ip>`. Unactivated devices must be attached over USB-C before they will appear.

**`tiiny scan` returns an empty table.**
The device is not attached over USB-C and not reachable on the local network. Discovery uses a UDP broadcast on port 39217, which does not cross subnets or most VLAN boundaries — a device on another segment will never appear here. Connect by IP directly with `tiiny connect <ip>`.

**`401` on a request that worked before.**
Almost always the 24-hour expiry. Re-run `tiiny auth key`.

**`404` on a model that exists.**
Downloaded is not the same as loaded. Inference requires the model in NPU memory:

```bash
tiiny ls -l          # what is actually loaded
tiiny load <model_id>
```

**`404` on a documented endpoint path.**
Four paths in the published API Reference do not match what the CLI calls. If `/v1/audio/speech`, `/v1/images/generations`, `/api/v1/models/<id>/start`, or `/api/v1/tasks` returns `404`, use `/v1/synthesize`, `/v1/image/generate`, `/api/v1/models/<id>/launch/stream`, or `/api/v1/models/running` instead. See [HTTP Endpoints](./http.md).

**Connection refused on a `*.api.tiiny` hostname.**
Name resolution goes through the `pcsvr` resolver, which is configured for the `tiiny.local` domain. If a bare `*.api.tiiny` name does not resolve, address the device by IP and set the vhost explicitly:

```bash
curl http://<device-ip>/v1/chat/completions -H "Host: openai.api.tiiny"
```

**`server gave HTTP response to HTTPS client`.**
Every Tiiny endpoint is plain HTTP. No endpoint supports TLS. Use `http://`.

**Upload rejected as too large.**
The CLI enforces the vault size cap locally, before sending. The rejection message names the limit.

---

## Rate limits

Both the OpenAI-compatible and Anthropic error tables document `429`, and Anthropic additionally documents `529`. **No published page states what the limits are**, whether they are enforced on-device, or what backoff is expected.

<!--
  DOC-DEBT: Needs a real answer. Also unstated anywhere in the corpus:
  per-model context window, concurrency behaviour (queue vs reject), and
  whether the OpenAI-compatible endpoint supports tools/function calling.
-->

---

## Debug output

**CLI.** Set `LOG_LEVEL=DEBUG` before running. Debug output is written to a log file in the current working directory.

```bash
export LOG_LEVEL=DEBUG        # macOS / Linux
$env:LOG_LEVEL="DEBUG"        # Windows
```

**HTTP.** Add `-v` to any curl request for verbose output including TiinyOS error detail.

```bash
curl -v http://<device-ip>/<endpoint> -H "Host: <vhost>"
```
