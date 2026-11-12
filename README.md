# courras-model-catalog

Published AI model catalog for the Courras Platform (tarjim.li, Courras Studio, and any
future composition). Governed by
[`ADR-072`](https://github.com/muatazkhataan/courras-platform/blob/main/engineering/decisions/ADR-072-model-catalog-updates.md).

## What this repository is

A **data-only** distribution point. Every published version is a GitHub Release carrying
two assets:

| Asset | Contents |
| --- | --- |
| `model-catalog.json` | the curated catalog: pricing, semantic tier, routing eligibility, bilingual display names, context limits, lifecycle |
| `model-catalog.json.sig` | a detached Ed25519 signature over the exact bytes of `model-catalog.json` |

Clients fetch the stable path:

```
https://github.com/muatazkhataan/courras-model-catalog/releases/latest/download/model-catalog.json
https://github.com/muatazkhataan/courras-model-catalog/releases/latest/download/model-catalog.json.sig
```

## What this repository is not

- **Not product source.** No application code, no build scripts, no configuration for the
  Courras applications. The source of truth for the catalog lives in the private
  `courras-platform` repository under `engineering/model-catalog/`; a workflow there
  generates, signs and publishes these assets. Nothing is edited here by hand.
- **Not the trust root.** A public key may be mirrored here for transparency, but a client
  must never fetch it from here. The only trusted copy is the one compiled into the
  application build — taking the key from the same origin as the document it verifies
  would make the signature meaningless.

## Release conventions

- Releases must be **published, non-draft, non-prerelease**. The
  `/releases/latest/download/` path resolves only to a full release; a prerelease leaves
  the stable path unresolvable.
- `version` inside the document is **monotonic**. Clients reject a document whose version
  is not greater than the one they already hold, so a version is never reused.
- Assets are immutable per tag. **Rollback is a new release** with a higher `version`
  carrying the prior content — never an overwrite of an existing asset.
- Each document carries `schemaVersion`, `minAppVersion` and `expiresAt`. A client rejects
  a future `schemaVersion`, an application older than `minAppVersion`, and an expired
  document, keeping whatever it already had.

## Verifying a release

```sh
gh release download --repo muatazkhataan/courras-model-catalog --pattern 'model-catalog.json*'

# Ed25519 verification against the published public key
openssl pkeyutl -verify \
  -pubin -inkey model-catalog-pub.pem \
  -rawin -in model-catalog.json \
  -sigfile model-catalog.json.sig
```

A verification failure is terminal: do not use the document.

## Privacy

Clients fetch these assets with an unauthenticated GET and send no API keys, key hashes,
user identity, project names, provider inventory or identifying `User-Agent`. This
repository therefore learns nothing about the people using it beyond ordinary CDN access
logs.
