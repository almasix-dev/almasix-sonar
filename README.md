# `@almasix/sonar`

First-party browser client for **Sonar**, Almasix's native realtime websocket
server (`/broadcasting/socket`). Speaks the `almasix:*` frame protocol the
server uses today. **Does not require `pusher-js`.**

Docs: [Broadcasting](https://docs.almasix.com/broadcasting/).

## Install

```bash
npm install @almasix/sonar
```

Requires Node.js **≥ 22** (browser bundles are fine on any modern browser).

## Quick start

Point `BROADCAST_CONNECTION` at `websocket` or the `sonar` alias, then:

```js
import Sonar from '@almasix/sonar';

const sonar = new Sonar({
  // defaults: same host as the page, path /broadcasting/socket
  // authEndpoint: '/broadcasting/auth',
});

await sonar.connect();

// Public
sonar.channel('announcements').listen('post.published', (data) => {
  console.log(data);
});

// Private — POSTs /broadcasting/auth with credentials/cookies
sonar.private(`authors.${name}`).listen('post.published', (data) => {
  console.log(data);
});

// Presence
sonar
  .join('rooms.lobby')
  .here((members) => console.log(members))
  .joining((member) => console.log('joined', member))
  .leaving((member) => console.log('left', member));

// Exclude this tab from broadcasts (server `to_others()`)
fetch('/posts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    ...sonar.socketIdHeader(), // { 'X-Socket-ID': '...' }
  },
  body: JSON.stringify({ title: 'Hi' }),
  credentials: 'include',
});
```

## Options

| Option | Default | Meaning |
| --- | --- | --- |
| `wsUrl` | derived | Full `ws://` / `wss://` URL |
| `host` / `port` / `path` / `scheme` | page location, `/broadcasting/socket` | URL pieces when `wsUrl` is unset |
| `forceTLS` | `false` | Prefer `wss` |
| `authEndpoint` | `/broadcasting/auth` | Private / presence auth |
| `auth.headers` / `auth.params` | — | Extra auth request fields |
| `reconnect` | `true` | Reconnect on close |

## Protocol

Compatible with Almasix Sonar frames:

- server → `almasix:connection_established` `{ socket_id }`
- client → `subscribe` / `unsubscribe` / `ping` / `client-*`
- server → `almasix:subscription_succeeded`, `almasix:member_added` / `member_removed`, broadcasts as `{ event, channel, data }`

Optional: inbound `pusher:*` system names are normalized to `almasix:*` if a
relay ever speaks them; the default client always **sends** Sonar/`almasix` shapes.

## Develop

```bash
npm install
npm run build
npm test
```

## Publishing (maintainers)

Releases use [Trusted Publishing](https://docs.npmjs.com/trusted-publishers/) — no `NPM_TOKEN` secret. CI authenticates with a short-lived OIDC token.

**npmjs.com (one-time):** package **Settings → Trusted Publisher → GitHub Actions** with:

| Field | Value |
|-------|--------|
| Organization or user | `almasix-dev` |
| Repository | `almasix-sonar` |
| Workflow filename | `publish.yml` |
| Environment | _(leave empty)_ |

Enable **`npm publish`** under allowed actions.

**Release:**

```bash
# Bump version in package.json so it matches the tag without "v"
git tag v0.1.1
git push origin main
git push origin v0.1.1
```

Pushing `v*` runs [`.github/workflows/publish.yml`](.github/workflows/publish.yml) (build, test, `npm publish` via OIDC).

## Alternatives

If you prefer a hosted bus, see the Broadcasting docs for **Pusher.js**,
**Ably**, and **Socket.IO** — those are supported alternatives, not the default.

## License

[MIT](LICENSE)
