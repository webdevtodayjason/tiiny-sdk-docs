# Tiiny SDK Reference

An independent, device-verified reference for the **Tiiny SDK CLI v0.0.3** — every
command, every HTTP endpoint it calls, every error it can return.

Written while building two applications against a Tiiny Pocket. Everything here was
checked against the shipped binary and, where noted, against live hardware.

## Contents

| Page | What it covers |
|---|---|
| [CLI Reference](./reference/cli.md) | All 70 command paths, with the flags and defaults the binary actually accepts |
| [HTTP Endpoints](./reference/http.md) | Every endpoint the CLI calls, grouped by the vhost that serves it |
| [Errors](./reference/errors.md) | Error codes, what they mean, and which ones are worth retrying |
| [Global Flags](./reference/global-flags.md) | `--json`, `--device-address`, `--config`, `-y`, and the shorthand collisions |

## How this was built

The CLI reference is generated from the binary's own `--help` output, so flags and
defaults match what the tool accepts rather than what any document claims. The HTTP
reference is recovered from the string table of Tiiny SDK CLI v0.0.3
(darwin/arm64, MD5 `4aef4f7808179703d001188fa6a67828`).

Two conventions run through the endpoint tables:

- **✓** — the path is a literal present in the shipped binary.
- **?** — the path is documented but does not appear in the binary, and needs
  confirmation against a device.

Request and response bodies are omitted where they have not been observed against
live hardware. Nothing here is inferred from documentation alone.

## Scope

This covers **v0.0.3, darwin/arm64**. If your binary's MD5 differs, the CLI and
endpoint tables may be stale — regenerate from your own `--help` output before
relying on them.

## Contributing

Corrections welcome, especially from anyone who can confirm a `?` path or fill in a
response shape. See [CONTRIBUTING.md](./CONTRIBUTING.md).

## Related

[Turnstile](https://github.com/webdevtodayjason/turnstile) — a small stdlib-only
library for sharing one Tiiny between multiple applications without collisions.

---

Not affiliated with or endorsed by Tiiny AI. Tiiny is their trademark; this is an
independent reference maintained by a device owner.
