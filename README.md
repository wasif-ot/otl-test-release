# OTL Auto-Update Test Feed

Public test feed for the OC-620 auto-update engine (`Foundation.FrictionlessUpdate.Engine`
in OtlDesktop). Not a production feed — for local/dev testing only.

## Layout

- `catalog.json` — the manifest the client fetches (single-product schema the
  `UpdateCatalog` parser accepts: `schemaVersion`, `product`, `channel`,
  `latest.version`, `latest.minSupportedVersion`, `latest.package.{url,sha256,sizeBytes}`).
- `releases/<version>/` — the package file for that version.

The client is pointed at this feed via the dev-only registry override:

```
HKCU\Software\OfficeTimeline\DevUpdateCatalogUrl
  = https://raw.githubusercontent.com/ahsan-oto/OTL-Releases-Test/main/catalog.json
```

## Publishing a new test version

1. Add the new package file under `releases/<new-version>/`.
2. Compute its SHA-256 (`Get-FileHash -Algorithm SHA256 <file>` in PowerShell).
3. Update `catalog.json`: bump `latest.version`, point `latest.package.url` at the
   new file's raw URL, update `sha256` and `sizeBytes`.
4. Commit and push to `main`. `raw.githubusercontent.com` caches for a few
   minutes — if the client doesn't pick up the change right away, wait a
   moment or append a cache-busting query string to the URL while testing.

The engine only accepts `https` URLs (or loopback `http`/local paths) and a
64-hex-char `sha256` — a bad edit to either field will surface as a parse or
verification failure on the client, not a silent no-op.
