# HTTP Endpoints

Every endpoint the Tiiny CLI calls, grouped by the virtual host that serves it.

**Provenance:** recovered from the string table of Tiiny SDK CLI v0.0.3 (darwin/arm64, MD5 `4aef4f7808179703d001188fa6a67828`). Paths marked **✓** are literals present in the binary. Paths marked **?** are documented on dev.tiiny.ai but do **not** appear in the binary and need device confirmation. Request and response bodies are omitted where they have not been observed against live hardware Unverified items are marked inline.

---

## Addressing

Requests reach the device one of two ways.

**By virtual host.** `pcsvr` on the host machine runs a resolver (`127.0.0.1:60053`) and routes by hostname:

```bash
curl http://openai.api.tiiny/v1/chat/completions
```

**By IP with an explicit `Host` header.** Equivalent, and what the CLI does internally:

```bash
curl http://<device-ip>/v1/chat/completions -H "Host: openai.api.tiiny"
```

> The resolver is configured for domain `tiiny.local` only. Whether the bare `*.api.tiiny` names resolve on a stock host is unconfirmed The `Host`-header form works regardless.

Every request outside the activation flow carries `Authorization: Bearer <auth_key>`. **The key expires 24 hours after issue.**

---

## `auth.api.tiiny` — activation and account

> Note the host is `auth.api.tiiny`, **not** `auth.api.tiiny.local`. The published documentation uses the `.local` form on every example; that string does not appear in the binary.

| Method | Path | Purpose | |
|---|---|---|---|
| POST | `/api/v1/connect` | Activation and account state. Unauthenticated returns device readiness; with a bearer token returns `user_info`. | ✓ |
| POST | `/api/v1/account/auth` | Log in with password. | ✓ |
| POST | `/api/v1/account/set_main_password` | Set the account password. Called during activation. | ✓ |
| POST | `/api/v1/account/set_local_account_info` | Set username, region, avatar. Called during activation. | ✓ |
| POST | `/api/v1/account/check_main_password` | Verify current password; returns `auth_token`. | ✓ |
| POST | `/api/v1/account/change_password` | Set a new password using `auth_token`. | ✓ |
| POST | `/api/v1/account/send_mail_code` | Send a verification code. `method`: `bind_email` / `forget` / `change_mail`. | ✓ |
| POST | `/api/v1/account/check_mail_code` | Verify a code. | ✓ |
| POST | `/api/v1/account/check_mail_sign_up_status` | Check whether an address is already registered. | ✓ |
| POST | `/api/v1/account/bind_email` | Bind a verified address. | ✓ |
| POST | `/api/v1/account/change_mail` | Replace the bound address. | ✓ |
| POST | `/api/v1/account/recovery/unlock_with_mail_code` | Password recovery; returns `recovery_token`. | ✓ |
| POST | `/api/v1/account/unlock_with_auth_key` | Undocumented. Purpose unconfirmed. | ✓ |

`tiiny logout` and `tiiny auth key` send **no HTTP request** — both operate only on the local config file.

---

## `p8800.api.tiiny` — models, NPU, inference

### Model lifecycle

| Method | Path | CLI | |
|---|---|---|---|
| GET | `/api/v1/models/online_models` | `tiiny ls -r` | ✓ |
| GET | `/api/v1/models/?including_downloading=true` | `tiiny downloads ls` | ✓ |
| GET | `/api/v1/models/info` | `tiiny info <id>` | ✓ |
| GET | `/api/v1/models/storage` | undocumented | ✓ |
| GET | `/api/v1/models/running` | `tiiny tasks ls` | ✓ |
| POST | `/api/v1/models/<id>/download` | `tiiny download <id>` | ✓ |
| POST | `/api/v1/models/<id>/download/stream` | download progress stream | ✓ |
| POST | `/api/v1/models/<id>/pause_download` | the `P` key during download | ✓ |
| DELETE | `/api/v1/models/<id>` | `tiiny rm <id>` | ✓ |
| POST | `/api/v1/models/<id>/launch/stream` | `tiiny load <id>` | ✓ |
| POST | `/api/v1/models/<id>/stop` | `tiiny unload <id>` | ✓ |
| POST | `/api/v1/models/unload_all` | `tiiny unload --all` | ✓ |
| POST | `/api/v1/models/<id>/interrupt` | `tiiny interrupt <id>` | ✓ |
| POST | `/api/v1/models/interrupt_all` | `tiiny interrupt --all` | ✓ |
| GET | `/api/v1/npu/status` | `tiiny top` | ✓ |

> **Two corrections against the published reference.** It documents `POST /api/v1/models/<id>/start` for loading a model and `GET /api/v1/tasks` for listing tasks. Neither string exists in the binary; the CLI calls `/launch/stream` and `/models/running` respectively.
>
> The binary also contains `/api/v1/models/npu/status` alongside `/api/v1/npu/status`. Which is canonical is unresolved

### Hugging Face import

| Method | Path | Notes | |
|---|---|---|---|
| POST | `/api/v1/models/import` | `tiiny import <hf_url>`. Body `{hf_url, model_desc}`. | ✓ |
| POST | `/api/v1/models/import/inspect` | Inspect before importing. Undocumented. | ✓ |
| POST | `/api/v1/models/import/<id>/pause` | Undocumented. | ✓ |
| POST | `/api/v1/models/import/<id>/cancel` | Undocumented. | ✓ |
| POST | `/api/v1/models/import/resume` | Undocumented. | ✓ |

Supported bases: `Qwen3.5-9B`, `Z-Image-Turbo`, and fine-tuned variants. `safetensors` only.

### Inference

| Method | Path | Modality | |
|---|---|---|---|
| POST | `/v1/chat/completions` | Text and image-text-to-text | ✓ |
| POST | `/v1/embeddings` | Embedding | ✓ |
| POST | `/v1/image/generate` | Image generation | ✓ |
| POST | `/v1/audio/transcriptions` | ASR | ✓ |
| POST | `/v1/synthesize` | TTS | ✓ |
| POST | `/v1/rerank` | Rerank | ✓ |
| POST | `/v1/ocr?model=<id>` | OCR | ✓ |
| POST | `/v1/music/generate/mp3` | Music generation | **?** |
| POST | `/v1/audio/speech` | TTS, as published | **?** |
| POST | `/v1/images/generations` | Image generation, as published | **?** |

> **The two `?` inference paths are the published ones.** `/v1/audio/speech` and `/v1/images/generations` are what the API Reference tells you to call; neither appears in the binary, which calls `/v1/synthesize` and `/v1/image/generate`. It is plausible the OpenAI-compatible vhost serves the published forms as aliases — that is unconfirmed. Until then, the binary's paths are the ones known to work.

`tiiny bench` is CLI-only; no HTTP endpoint backs it.

---

## `kb.tiiny.local` — vault and profile

The only vhost using the `.local` suffix.

| Method | Path | CLI | |
|---|---|---|---|
| GET | `/kb/sources?limit=500` | `tiiny vault ls` | ✓ |
| POST | `/kb/upload` | `tiiny vault add` (multipart `file=@`) | ✓ |
| POST | `/kb/finalize` | `tiiny vault index` (body `{source_id}`) | ✓ |
| POST | `/kb/retrieve` | `tiiny vault query` (body `{question}`) | ✓ |
| POST | `/kb/delete-memory` | `tiiny vault rm` (body `{source_id}`) | ✓ |
| GET | `/kb/files/<source_id>` | `tiiny vault download` | ✓ |
| PATCH | `/kb/sources/<source_id>` | `tiiny vault rename` (body `{filename}`) | ✓ |
| GET | `/kb/memory/schedule` | `tiiny vault summary-config show` | ✓ |
| PUT | `/kb/memory/schedule` | `tiiny vault summary-config set -t` | ✓ |
| GET | `/kb/memory/model` | `tiiny vault summary-config show` | ✓ |
| PUT | `/kb/memory/model` | `tiiny vault summary-config set -m` | ✓ |
| GET | `/kb/user-context` | `tiiny profile show` | ✓ |
| GET | `/kb/user-context/versions?limit=1` | `tiiny profile show` (for `changed_by`) | ✓ |
| PATCH | `/kb/user-context/profile/<field>` | `tiiny profile set` / `clear` | ✓ |

Notes carried from the published docs and worth preserving:
- `vault query`'s `--top_k`, `--threshold`, and `--full` are **client-side only** — they do not appear in the `/kb/retrieve` request body.
- The CLI field `more` maps to the API field `more_about_you`. `occupation` and `preferences` map directly.
- `summary-config set -t 02:00` sends `{nightly_start_hour: 2, nightly_end_hour: 10}` — an eight-hour window, hourly granularity, minutes discarded.
- The CLI rejects uploads over the size cap locally, before the request is sent.

---

## `connector.api.tiiny` — connectors

| Method | Path | Purpose | |
|---|---|---|---|
| GET | `/v1/status` | App connector states | ✓ |
| GET | `/v1/<connector>/status` | OAuth connector state (`X-User-Id` required) | ✓ |
| GET | `/v1/<connector>/check` | App connector state | ✓ |
| GET | `/v1/<connector>/auth-url` | Begin an OAuth flow | ✓ |
| DELETE | `/v1/<connector>/connection` | Disconnect an OAuth connector | ✓ |
| POST | `/v1/telegram/check-with-token` | Verify a Telegram bot token | ✓ |
| POST | `/v1/discord/check-with-token` | Verify a Discord bot token | ✓ |
| POST | `/v2/x/check-with-token` | Verify X `auth_token` + `ct0` | ✓ |
| POST | `/v1/email/imap/verify` | Verify IMAP credentials | ✓ |
| PATCH | `/v1/config/<connector>` | Persist connector credentials | ✓ |
| PATCH | `/v1/config/channels/<connector>` | Clear connector credentials | ✓ |
| POST | `/v1/whatsapp/qr` | Request a pairing QR | ✓ |
| POST | `/v1/whatsapp/bridge/logout` | Disconnect WhatsApp | ✓ |
| GET | `/v1/mcp/servers` | List custom MCP connectors | ✓ |
| POST | `/v1/mcp/servers` | Add one (manual) | ✓ |
| POST | `/v1/mcp/servers/import-json` | Add one from a config file | ✓ |
| GET | `/v1/mcp/servers/<id>/check` | MCP connector status | ✓ |
| DELETE | `/v1/mcp/servers/<id>` | Remove one | ✓ |

OAuth `<connector>` values: `gmail`, `outlook`, `calendar`, `outlook/calendar`. App values: `telegram`, `discord`, `x`, `whatsapp`, `email`.

> Credentials on this vhost travel in cleartext over HTTP: bot tokens, X session cookies, and IMAP/SMTP passwords.

---

## `wifi.api.tiiny` — network

| Method | Path | CLI | |
|---|---|---|---|
| POST | `/api/v1/sys/wifi/connection_switch` | `tiiny wifi on` / `off` | ✓ |
| GET | `/api/v1/sys/wifi/connection_status` | `tiiny wifi status` | ✓ |
| GET | `/api/v1/sys/wifi/wifi_list` | `tiiny wifi connect` (scan step) | ✓ |
| POST | `/api/v1/sys/wifi/wifi_connect` | `tiiny wifi connect` | ✓ |
| POST | `/api/v1/sys/wifi/wifi_disconnect` | `tiiny wifi disconnect` | ✓ |
| — | `/api/v1/sys/network` | undocumented | ✓ |

> `wifi_connect` posts the SSID and pre-shared key as cleartext JSON.

---

## `upgrade.api.tiiny:5555` — firmware

| Method | Path | | |
|---|---|---|---|
| POST | `/api/upgrade/check` | Check for a target version | ✓ |
| POST | `/api/upgrade/download_and_ota_prepare?target_version=<v>` | Stage the upgrade | ✓ |
| GET | `/api/upgrade/status/v2` | Poll; wait for `PREPARED`, then for `SUCCESS` | ✓ |
| POST | `/api/upgrade/upgrade_to?target_version=prepared` | Apply | ✓ |

---

## `agents.api.tiiny` — agents

| Method | Path | | |
|---|---|---|---|
| GET | `/api/services` | List agents | ✓ |
| GET | `/api/services/download/<app_id>/status` | Agent download status | ✓ |
| DELETE | `/api/services/<resource_name>` | Cancel an agent download | ✓ |

---

## Protocol vhosts

Three compatibility endpoints share the same `api_key`. Base URLs as they appear in the binary:

| Protocol | Base URL | Modalities |
|---|---|---|
| OpenAI-compatible | `http://openai.api.tiiny/v1` | Text, Embedding, Rerank, Image, ASR, TTS |
| Anthropic | `http://anthropic.api.tiiny` | Text generation only |
| Ollama | `http://ollama.api.tiiny` | Text generation only |

> These three literals are what the binary contains, matching the Overview page. The `tiiny auth key` sample output published on the API Key page shows a different form (`http://<device-ip>/openai/v1`) that does not appear in the binary

---

## Discovery

Not HTTP, and not reachable with `curl`.

| Transport | Detail |
|---|---|
| UDP 39217 | `tiiny scan` broadcasts `GADGET_DISCOVER_V1` to the IPv4 broadcast and IPv6 multicast addresses. |
| HTTP 39218 | `GET /device.json` returns device metadata. **Unauthenticated.** |

`tiiny connect <ip>` performs no connect call — it validates the IP by fetching `device.json`, then writes the selection to the local config.
