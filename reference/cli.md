# CLI Reference

Complete command reference for Tiiny SDK CLI **v0.0.3**. Generated from the shipped binary's own help output, so flags and defaults here match what the tool actually accepts.

Every command also accepts the [global flags](./global-flags.md) — `--json`, `--device-address`, `--config`, `-y/--yes`, `-h/--help`. They are not repeated below.

For the HTTP call behind each command, see [HTTP Endpoints](./http.md).

---

## Authentication

### `tiiny init`
Activate a device, create account credentials, and choose a password recovery mode. All input is collected interactively.

Recovery modes:
- **Email** — requires Wi-Fi and verification. Enables password reset by emailed code.
- **Local only** — credentials stored on-device. **If the password is lost with no email bound, the account cannot be recovered.**

### `tiiny login`
Sign in with an existing Tiiny account. Prompts for username and password.

> The CLI allows three attempts before locking out. Whether that counter is enforced on the device or only in the client is unconfirmed — the HTTP endpoint is reachable directly.

### `tiiny logout`
Sign out. Removes the auth key from the local config file. Sends no HTTP request.

### `tiiny auth info`
Show the current account username and bound email.

### `tiiny auth username`
Update the account display name. Prompts interactively.

### `tiiny auth passwd`
Change the account password. Prompts for the current password (three attempts), then the new password twice. **Logs you out on success.**

### `tiiny auth forgot-passwd`
Reset the password by emailed verification code. Requires a bound email.

### `tiiny auth email-bind`
Bind an email address for password recovery.

### `tiiny auth email-update`
Replace the bound address. Prompts for current password, then new email, then code.

### `tiiny auth key`
Fetch API credentials for SDK or OpenAI-compatible clients.

| Flag | Description |
|---|---|
| `--provider <name>` | Show credentials for one provider: `openai`, `anthropic`, or `ollama`. |

Reads the auth key from the local config and prints it with the protocol base URLs. Sends no HTTP request.

> **The key expires 24 hours after issue.** Any client configured with it — an OpenAI SDK, a coding agent, a chat UI — will begin returning `401` after that window and must be re-keyed. Plan for rotation rather than pasting it once.

---

## Models

### `tiiny ls`
List models. With no flag, lists downloaded models.

| Flag | Description |
|---|---|
| `-r`, `--remote` | Store models: name, ID, type, download status, size, updated |
| `-d`, `--downloaded` | Downloaded: name, ID, type, NPU size, downloaded, source |
| `-l`, `--loaded` | Loaded: name, ID, type, NPU usage, start time, last called, source |

### `tiiny info <model_id>`
Show detailed metadata for a model. Includes reasoning-mode support, which is the only reliable way to know whether a model accepts `--thinking`.

### `tiiny download <model_id>`
Download a model from the store. `P` pauses, `C` cancels. Requires Wi-Fi; prompts to connect if offline.

### `tiiny import <hugging_face_model_url>`
Import from Hugging Face. Fetches metadata, downloads, converts, and writes to the model store.

| Flag | Description |
|---|---|
| `-d`, `--description <text>` | Model description |

Supported bases: `Qwen3.5-9B`, `Z-Image-Turbo`, and fine-tuned variants. Files must be `safetensors`.

### `tiiny rm <model_id>`
Delete a downloaded model from the device.

### `tiiny load <model_id>`
Load a downloaded model into NPU memory. Required before inference.

### `tiiny unload <model_id>`
Release a model from NPU memory.

| Flag | Description |
|---|---|
| `--all` | Unload every loaded model |

### `tiiny top`
Show current NPU utilization and loaded model activity.

### `tiiny tasks ls`
List inference tasks currently consuming model resources. Matches the GUI's NPU Utilization panel.

Returns a `tasks` array of `{task_name, model_id}`, where `task_name` is `Chat`, an agent name, `API`, or `Auto Summary`.

### `tiiny interrupt <model_id>`
Interrupt an active inference task.

| Flag | Description |
|---|---|
| `--all` | Interrupt every active task |

### `tiiny bench <model_id>`
Benchmark inference performance for a **loaded text-generation** model. ASR and other modalities are not supported.

| Flag | Default | Description |
|---|---|---|
| `--token-sizes <ints>` | `256` | Token lengths to test, e.g. `--token-sizes 256,512` |
| `--think` | off | Benchmark with thinking enabled |

CLI-only — no HTTP endpoint backs this command.

### `tiiny downloads ls`
Show download tasks for both models and agents.

### `tiiny downloads cancel <resource_name>`
Cancel an in-progress download. Sends the cancel to both the model and agent services and ignores a not-found from either.

---

## Inference

### `tiiny run`
Run text or multimodal inference.

| Flag | Default | Description |
|---|---|---|
| `-m`, `--model <id>` | — | Target model |
| `-p`, `--prompt <text>` | — | User prompt |
| `-s`, `--system <text>` | — | System prompt |
| `-f`, `--file <path>` | — | Attach an image or document. Repeatable. |
| `-t`, `--thinking` | off | Enable thinking mode |
| `--thinking-depth <level>` | **`medium`** | `low`, `medium`, or `high` |

> **Correction:** the published reference states `--thinking-depth` defaults to `low`. The binary's default is `medium`. It also omits the `-t` shorthand for `--thinking`.

Thinking support varies by model — some always think, some allow toggling, some support depth. Check `tiiny info <model_id>`.

> **Profile is not injected.** `tiiny run` does not pull `occupation`, `preferences`, or `more` into context. Profile injection happens only in TiinyOS Task Mode. To use profile content from the CLI, read it with `tiiny profile show` and pass it via `-s`.

### `tiiny run embed`
Generate embeddings from input text.

| Flag | Description |
|---|---|
| `-d`, `--dimensions <int>` | Output vector dimensions |

### `tiiny run image`
Generate an image from a prompt.

| Flag | Default | Description |
|---|---|---|
| `--size <w:h>` | `512:512` | Output dimensions |
| `--steps <int>` | `9` | Sampling steps |
| `-o`, `--output <path>` | `output.png` | Output file |

> **Unresolved.** The binary's help text says *"for example 512, 1024, or 2048"* — bare integers. The published reference requires `width:height`, both multiples of 16, satisfying `ceil_to_32((w/16)*(h/16)) <= 9216`. These contradict; the constraint needs a live test. Documented presets: `512:512`, `432:768`, `768:432`, `768:480`, `480:768`, `640:480`, `480:640`, `864:1536`, `1536:864`, `1536:960`, `960:1536`, `1024:1024`, `1280:960`, `960:1280`.
>
> Note the axis separator differs by interface: `width:height` for the CLI, `widthxheight` for the HTTP `size` field.

### `tiiny run asr`
Transcribe an audio file. Pass the file with the global `-f`.

| Flag | Description |
|---|---|
| `--language <lang>` | Recognition language override |

### `tiiny run tts`
Convert text to speech.

| Flag | Default | Description |
|---|---|---|
| `--voice <preset>` | — | Voice preset |
| `-o`, `--output <path>` | `output.mp3` | Output file |

> Whether `--voice` is required depends on the model — `Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice` supplies a default, `Supertone/supertonic-3` does not. There is currently no documented way to determine this for an arbitrary model short of trying it.

### `tiiny run rerank`
Reorder candidate documents by relevance.

| Flag | Description |
|---|---|
| `-q`, `--query <text>` | Query string |
| `-f`, `--file <path or string>` | Candidate documents — accepts a file path or a literal string |

### `tiiny run ocr`
Extract text from an image. Pass the image with the global `-f`.

> **Music generation is not exposed by the CLI** in v0.0.3. There is no `tiiny run music`. Use the HTTP endpoint directly.

---

## Vault

Formerly `tiiny kb`. **The `kb` namespace no longer exists** — the published API Reference index still lists all nine vault commands under `tiiny kb`, which will fail on v0.0.3.

### `tiiny vault ls`
List files and their index status. `is_indexed: false` means uploaded but not yet retrievable.

### `tiiny vault add <file_path>`
Upload a file. Vectorized automatically if an embedding model is loaded. The CLI enforces the size cap locally before sending.

### `tiiny vault index <file_name>`
Generate embeddings for an uploaded file. Requires login, a loaded embedding model, and the file already uploaded.

### `tiiny vault query <query_str>`
Semantic search across indexed files.

| Flag | Default | Description |
|---|---|---|
| `--top_k <int>` | `3` | Maximum results |
| `-t`, `--threshold <float>` | — | Relevance threshold |
| `--full` | off | Show full text instead of truncating at 80 characters |

> All three are **client-side display filters** — none are sent to `/kb/retrieve`. The published reference omits `--threshold` entirely and does not state the `--top_k` default.
>
> `-t` here means `--threshold`, not `--thinking`. See [flag collisions](./global-flags.md#flag-shorthand-collisions).

### `tiiny vault rename <old_name> <new_name>`
Rename a file.

### `tiiny vault download <file_name>`
Download a file to the host.

| Flag | Description |
|---|---|
| `-o`, `--output <path>` | Output path |

### `tiiny vault rm <file_name>`
Delete a file and its index data.

### `tiiny vault summary-config show`
Show the schedule and model for automatic daily summaries.

### `tiiny vault summary-config set`
Configure automatic daily summaries. Requires the target model to be downloaded.

| Flag | Description |
|---|---|
| `-t`, `--time <HH:MM>` | Daily start time, 24-hour |
| `-m`, `--model <id>` | Summarization model |

> **Hourly granularity only.** Minutes are discarded — `12:45` takes effect at `12:00`. The CLI sends an eight-hour window: `-t 02:00` becomes `{nightly_start_hour: 2, nightly_end_hour: 10}`.

---

## Profile

Personalization fields used by TiinyOS Task Mode.

### `tiiny profile show`
Show the profile and `updated_at`, including which source last wrote each field.

### `tiiny profile set <field> <value>`
Set a field. Takes effect from the next conversation.

| Field | Limit | API field |
|---|---|---|
| `occupation` | 50 chars | `occupation` |
| `preferences` | 1000 chars | `preferences` |
| `more` | 1000 chars | `more_about_you` |

### `tiiny profile clear <field>`
Clear one field, or `all` for every field.

**Write precedence:** manual (`profile set`) is highest and is never overwritten. Auto-summary may update `preferences` and `more`. Task Mode tool-calls write immediately when you tell the model to remember something.

> These commands read and write profile data only. **They do not inject it into inference** — see the note under `tiiny run`.

---

## Connectors

> Connection management only in this release. **Execution** actions run through TiinyOS Task Mode.

Built-in: Outlook, Google Calendar, Outlook Calendar, X, Telegram Bot, Discord Bot, WhatsApp. Custom connectors via MCP (HTTP, stdio, SSE).

### `tiiny connector ls`
List connectors with `name`, `category` (email / calendar / app / mcp), and `status`.

### `tiiny connector status <connector_name|server_id>`
Connection details: `name`, `status`, `permissions`, `connected_at`. Accepts an MCP server ID as well as a connector name.

### `tiiny connector connect <connector_name>`
Connect a connector. The flow depends on type:

| Type | Connectors | Flow |
|---|---|---|
| OAuth | outlook-mail, google-calendar, outlook-calendar | Prints an authorization URL, polls for completion |
| IMAP/SMTP | other-mail | Prompts for provider, address, authorization code |
| Token | telegram, discord | Prompts for bot token |
| Dual token | x | Prompts for `auth_token` and `ct0` |
| QR | whatsapp | Renders a QR in the terminal; `R` refreshes, `Q` cancels |

> Credentials are transmitted and stored in cleartext over HTTP.

### `tiiny connector disconnect <connector_name>`
Disconnect and remove stored credentials. Reconnectable at any time.

### `tiiny connector mcp add`
Add a custom MCP connector.

| Flag | Description |
|---|---|
| `-f`, `--file <path>` | Import from a JSON config |
| `--manual` | Interactive input |
| `--run-on-tiiny` | Run the server on the device instead of the host |

**Choose the runtime carefully — it is read-only after saving.** Run on the host (default) for servers needing local apps, files, or browsers; run on Tiiny for servers that need nothing from the host. Host-side dependencies are installed before the server starts.

### `tiiny connector mcp rm <connector_id>`
Remove a custom MCP connector.

---

## Hardware

### `tiiny scan`
Discover devices over USB-C and the local network. Broadcasts `GADGET_DISCOVER_V1` on UDP 39217; `curl` cannot reproduce this step.

Unactivated devices must be attached over USB-C. Activated devices are discoverable over USB-C or the local network.

> `--json` is accepted and ignored — output is the same ASCII table either way.

### `tiiny connect <device_ip>`
Select the active device. Validates by fetching `device.json` on port 39218 and writes the selection to the local config; there is no connect call.

> The selected view can be either the normal network path or the **local USB-direct path** — a distinction the published documentation does not mention.

If the device is unactivated this begins activation; if activated but not logged in, it begins login.

### `tiiny status`
Device address, connection state, activation state, NPU/SOC/RAM/disk/NPU-RAM usage, and loaded models.

### `tiiny wifi [on|off]`
Enable or disable the Wi-Fi module.

### `tiiny wifi connect`
Scan and connect. Lists networks for arrow-key selection, then prompts for the password.

> SSID and pre-shared key are posted as cleartext JSON over HTTP.

### `tiiny wifi disconnect`
Disconnect from the current network.

### `tiiny wifi status`
Current Wi-Fi state and connected network.

### `tiiny upgrade`
Upgrade device firmware. Checks for a target version, downloads and prepares it, polls until `PREPARED`, applies, then polls until `SUCCESS`.

---

## Shell completion

Undocumented in the published reference, fully functional.

```bash
tiiny completion zsh > $(brew --prefix)/share/zsh/site-functions/_tiiny
tiiny completion bash > $(brew --prefix)/etc/bash_completion.d/tiiny
tiiny completion fish > ~/.config/fish/completions/tiiny.fish
tiiny completion powershell | Out-String | Invoke-Expression
```

| Flag | Description |
|---|---|
| `--no-descriptions` | Omit completion descriptions |

---

## Not present in v0.0.3

The published API Reference indexes these commands. They do not exist in this build.

| Command | Documented as | Status |
|---|---|---|
| `tiiny backup` | Back up device data | Not in binary; reference page also missing |
| `tiiny restore <file_path>` | Restore from a backup | Not in binary; reference page also missing |
| `tiiny recovery` | Factory reset | Not in binary; vendor indicates early September |
| `tiiny kb *` (9 commands) | Vault operations | Renamed to `tiiny vault`; index not updated |

`tiiny --help` also renders an empty `Skill:` group heading — a command group registered with no commands under it.
