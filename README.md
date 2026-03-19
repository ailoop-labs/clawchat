# ClawChat

ClawChat is a companion app for `zeroclaw`.

Its job is to make server access simple:

- pair a server with a one-time code
- chat over the existing `zeroclaw` WebSocket gateway
- view health, status, devices, and common management actions

ClawChat is not a new agent runtime.

- `zeroclaw` stays on the server
- model/provider keys stay on the server
- ClawChat acts as the human-facing client

## Docs

- [MVP Plan](docs/clawchat-mvp.md)

## License

MIT

## Initial Scope

- PWA first
- QR code or pairing-code onboarding
- `/ws/chat` session UI
- status, health, and device management

## Repo Status

Initial repository created on 2026-03-19.
