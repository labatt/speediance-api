# Login, session slots and headers

## One session per client type

Speediance keeps **one session per client type**, not one per account. Logging in
with a given `App_type` takes that slot and signs out whoever held it. The
displaced session's next request fails with `code` 90 ("offline"); an ordinary
expiry gives `code` 91.

| `App_type` | Slot | Notes |
|---|---|---|
| `SOFTWARE` | the phone app | using it signs the phone out |
| `HARDWARE` | **the machine itself** | using it signs you out at the Gym Monster |
| `NANO`, `BIKE` | other Speediance products | free unless you own that product |
| `WEB` | web client | needs an unpublished request signature (`code` 1005) |

Any other value gives `code` 1002 ("invalid appid").

**Pick a slot you don't otherwise use.** A script that logs in as `SOFTWARE`
signs you out of your phone every run, and `HARDWARE` signs you out at the
machine mid-workout. If you own neither a Nano nor a Bike, those slots are free
and a login there leaves both the phone and the machine signed in. A token
obtained on `NANO` or `BIKE` works for ordinary requests afterwards.

Non-`SOFTWARE` logins must send `Versioncode: 1`. A full version code there gives
`code` 1004 ("invalid nonce string"), and omitting it gives 1003.

## Logging in

```
POST /api/app/v2/login/verifyIdentity   {"type": 2, "userIdentity": "<email>"}
POST /api/app/v2/login/byPass           {"userIdentity": "<email>", "password": "...", "type": 2}
```

`byPass` returns `data.token` and `data.appUserId`, sent on later requests as the
`Token` and `App_user_id` headers. `POST /api/app/login/logout` ends the session.

**A `NANO` or `BIKE` login response omits `unit`.** It carries only `weightUnit`,
which reads `0` even on an imperial account. Carry the unit over from a previous
`SOFTWARE` session or from configuration; trusting the login response silently
stores every weight in the wrong unit.

## Headers

| Header | Value |
|---|---|
| `App_type` | the client type above |
| `Versioncode` | app version code, or `1` for non-`SOFTWARE` logins |
| `Token`, `App_user_id` | from the login response |
| `Timestamp` | milliseconds since the epoch |
| `Mobiledevices` | JSON device description |
| `Timezone`, `Utc_offset` | e.g. `America/New_York`, `-0400` — ASCII only |
| `Accept-Language` | e.g. `en` |

## Version gating

`Versioncode` gates content by declared app version. Too low and the API returns
`code` 98 ("please upgrade the APP version") or silently returns less — the
exercise library shrinks, for instance. The gate keys off the *client's declared*
version, not the hardware.

Do not jump far above the real app version or change it rapidly: implausible
values get throttled with "invalid nonce string". Step modestly, and don't lower
it below what your features need.

## Units

Weights are returned **and accepted** in the account's display unit; nothing
converts on either side. Write the same unit you read. The one exception found so
far is the exercise library's `recommendedWeight`, which is always kilograms.
