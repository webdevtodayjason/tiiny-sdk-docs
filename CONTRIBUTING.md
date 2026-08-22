# Contributing

The most useful contributions are confirmations against real hardware.

## Confirming a `?` endpoint

Endpoints marked **?** are documented but absent from the binary's string table.
If you can reach a device, a confirmation looks like this:

```
Endpoint:  POST /v1/audio/speech
Vhost:     openai.api.tiiny
SDK:       v0.0.3 (MD5 4aef4f7808179703d001188fa6a67828)
Result:    200 / 404 / other
Body:      <paste the response>
```

Open an issue with that block and the table gets updated with your result.

## Filling in a response shape

Several endpoints are listed without request or response bodies because they have
not been observed. If you have a real capture, paste it — redact your API key, your
device IP, and anything from your vault or profile first.

## Corrections

If something here contradicts your device, your device wins. Say which SDK version
and platform you are on, since the tables are version-pinned to v0.0.3 darwin/arm64.

## What does not belong here

Please do not open issues or PRs containing device API keys, account credentials,
connector tokens, or personal data pulled from a vault or profile.
