# Local Docker Firebase Remote Config — reference

## Why two paths?

| Path | When | Why |
|------|------|-----|
| **Redis FF cache** | Apps that store the RC template JSON in Redis | Cache hit → no live Firebase call |
| **Docker `firebase-mocker` `:9299`** | Clients using `FIREBASE_REMOTE_CONFIG_URL_BASE` | Community HTTP mock of RC REST API |

Official Firebase Emulator Suite does **not** include Remote Config.

## Container (RC mock)

| Item | Default |
|------|---------|
| Name | `app-firebase-rc-local` |
| Image | `node:22-bookworm-slim` + `firebase-mocker@2` |
| Port | `9299` |
| Project | `local-test` |

## Redis cache JSON shape

```json
{
  "parameters": {
    "<flag_key>": {
      "defaultValue": { "value": "<string>" }
    }
  }
}
```

## Inspect helpers

```bash
docker exec app-redis-local redis-cli GET "$FEATURE_FLAG_CACHE_KEY"
curl -s "http://127.0.0.1:9299/v1/projects/local-test/remoteConfig"
```

## Cleanup

```bash
docker exec app-redis-local redis-cli DEL "$FEATURE_FLAG_CACHE_KEY"
docker rm -f app-firebase-rc-local
```

## Limitations

- No official RC emulator; mock ≠ production (conditions, % rollouts, version history).
- Redis TTL expiry → live Firebase if credentials still point at cloud.
- RC mock is in-memory; recreate container → re-seed.

## Related

- Sibling: `local-docker-redis`, `local-docker-pubsub`, `local-docker-firestore`
