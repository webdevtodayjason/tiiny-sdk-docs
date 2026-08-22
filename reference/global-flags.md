# Global Flags

Every `tiiny` command accepts these flags. They are not repeated in the per-command reference.

| Flag | Default | Description |
|---|---|---|
| `--json` | off | Render machine-readable output. Supported on a subset of commands — see [Which commands support `--json`](#json-support). |
| `--device-address <address>` | from config | Target a specific device for this command only. Does not modify the stored configuration. |
| `--config <path>` | `~/.tiiny/config.json` | Use an alternate CLI config file. |
| `-y`, `--yes` | off | Answer yes to every confirmation prompt. |
| `-h`, `--help` | — | Help for the command. |

---

## `--device-address` {#device-address}

`tiiny connect <ip>` writes the selected device into the config file and every later command uses it. `--device-address` overrides that for a single invocation without changing the stored value.

```bash
# Configured device
tiiny status

# A different device, this once
tiiny --device-address 192.168.1.42 status
```

This is the supported way to work with more than one device from one host. Pair it with `--config` to keep fully separate profiles:

```bash
tiiny --config ~/.tiiny/lab-a.json status
tiiny --config ~/.tiiny/lab-b.json status
```

---

## `--config` {#config}

Path to the CLI configuration file. Default `~/.tiiny/config.json`. The file is created on first successful `tiiny connect` — before that it does not exist, and commands requiring a device report:

```
No Tiiny device connected. Run tiiny scan followed by tiiny connect <ip_address> to connect to your device.
```

The config stores the selected device address and the account auth key. **The auth key expires 24 hours after issue** — re-run `tiiny auth key` to refresh it. A stale config produces `401` responses rather than a re-login prompt.

---

## `--json` {#json-support}

Renders structured output instead of the human-readable table.

```bash
tiiny ls -d --json
tiiny status --json
```

> **Not universal.** The flag's own help text says "when the command supports it", and commands that do not support it accept the flag and silently ignore it rather than erroring. `tiiny scan --json` emits the same ASCII table as `tiiny scan`.

<!--
  DOC-DEBT: The support matrix below cannot be completed without a connected
  device, and the output schemas cannot be written at all. This is the single
  largest gap in the reference. Unverified against hardware.
  Needed per command: does --json change the output, and what is the schema?
-->

| Command | `--json` support | Schema |
|---|---|---|
| `tiiny scan` | **Ignored** — identical table output | — |
| all others | *pending device verification* | *pending* |

---

## `-y` / `--yes` {#yes}

Skips confirmation prompts. Relevant to the destructive commands:

```bash
tiiny rm <model_id> -y
tiiny vault rm <file_name> -y
tiiny connector disconnect <name> -y
tiiny profile clear all -y
```

Note that `-y` is global — it applies to any command that prompts, not only the ones whose reference entry mentions it.

---

## Flag shorthand collisions

`-t` is bound to different flags depending on the command:

| Command | `-t` means |
|---|---|
| `tiiny run` and its subcommands | `--thinking` |
| `tiiny vault query` | `--threshold` |
| `tiiny vault summary-config set` | `--time` |

Scripts passing `-t` should use the long form instead.
