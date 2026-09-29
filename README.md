# Speediance API notes (unofficial)

Community notes on the HTTP API used by the Speediance Gym Monster mobile app.

**Unofficial and unaffiliated.** Not endorsed by Speediance. Nothing here is a
specification: it is what the app appears to do, written down so that owners can
work with their own training data. Endpoints change without notice.

## What's here

| File | Contents |
|---|---|
| [`docs/endpoints.md`](docs/endpoints.md) | 665 API route names, grouped by resource |
| [`docs/verified.md`](docs/verified.md) | The routes actually called, with parameters and response fields |
| [`docs/auth.md`](docs/auth.md) | Login, session slots, and the headers every request needs |

## How the route list was made

The Android app is built with Flutter, so almost all of its code compiles into
`libapp.so` inside the ABI split (`split_config.arm64_v8a.apk`) — `base.apk`
holds only Java class names. API paths survive in that library as plain strings,
so the list is a filtered string extraction. Route names carry no request or
response shape, which is why `verified.md` exists separately: a route only moves
there once a real call has confirmed it.

No Speediance code is reproduced in this repository, and no credentials,
account identifiers or personal training data appear in it.

## Scope and etiquette

These notes came out of making a personal training account's own data usable.
Please keep to the same shape:

- read your own account, not anyone else's;
- go slowly — the API throttles, and a burst of probes looks like an attack;
- prefer the documented session slots in [`docs/auth.md`](docs/auth.md) so you
  don't sign yourself out of your own machine or phone;
- treat write endpoints as sharp: a wrong value lands in real training history.

## Contributing

Corrections and additions are welcome, especially newly confirmed parameters.
Please include how you confirmed a route, and never include tokens, account ids
or personal data in an example.

## License

[MIT](LICENSE) — documentation only.
