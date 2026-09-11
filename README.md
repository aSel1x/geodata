# geodata

Nightly `geosite.dat` and `geoip.dat` for [Xray](https://github.com/XTLS/Xray-core) and
V2Ray, built from two small config files in this repo.

| file | contents | size |
|---|---|---|
| `geosite.dat` | `youtube` `telegram` `openai` `google-gemini` `instagram` `discord` `anthropic` | ~8 KB |
| `geoip.dat` | `telegram`, `ru`, `private` — IPv4 and IPv6 | ~350 KB |

Built every night at 00:00 UTC. Each build is verified — minimum size plus one assertion per
section named in the configs — before anything is published.

## Download

Three channels, and they are not equivalent — pick by what constrains you.

```
# raw on the release branch — always current, one host, no redirect. Default choice.
https://raw.githubusercontent.com/aSel1x/geodata/release/geoip.dat
https://raw.githubusercontent.com/aSel1x/geodata/release/geosite.dat

# GitHub release — always current, immutable per-tag archive, honours If-Modified-Since.
# Costs a 302 from github.com to a second host.
https://github.com/aSel1x/geodata/releases/latest/download/geoip.dat

# jsDelivr — different network (Cloudflare), but LAGS: see the caveat below.
https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/geoip.dat
```

**Freshness.** raw and the release always serve the latest build. jsDelivr does not: it
caches branch references with `s-maxage=43200` and the purge API does not reliably flush the
branch-to-content mapping, so it can be up to 12 hours behind — measured, not theoretical.
The same lag is visible on the larger projects using this pattern. Since the inputs change
slowly (DB-IP monthly, domain lists a few commits a week), that is usually harmless, but do
not treat jsDelivr as byte-identical to the release.

**Network.** `release-assets.githubusercontent.com`, `raw.githubusercontent.com` and
`objects.githubusercontent.com` all resolve to `185.199.108-111.133`. Moving from the release
asset to raw removes the redirect and the second hostname, but stays on the same IPs. Only
jsDelivr (Cloudflare) is on a different network — that is the trade it offers in exchange for
the lag above.

Do not use `cdn.statically.io` for `geoip.dat`: it truncates the response at 64 KB and
returns HTTP 200 while doing so.

Checksums sit next to each file (`.sha256sum`), and
[`version.json`](https://raw.githubusercontent.com/aSel1x/geodata/release/version.json) on the
`release` branch carries the build timestamp and the upstream commits it was built from — a
200 with a valid checksum does not by itself mean the data is current.

```bash
curl -fsSLO https://raw.githubusercontent.com/aSel1x/geodata/release/geoip.dat
curl -fsSLO https://raw.githubusercontent.com/aSel1x/geodata/release/geoip.dat.sha256sum
sha256sum -c geoip.dat.sha256sum
```

## Auto-update in Xray

Xray-core ≥ v25.8.3 updates geodata itself. Set `outbound` to a working proxy tag and the
files come down through the tunnel, which makes any IP-level block on GitHub irrelevant.

```json
{
  "geodata": {
    "cron": "0 4 * * *",
    "outbound": "proxy",
    "assets": [
      { "url": "https://raw.githubusercontent.com/aSel1x/geodata/release/geoip.dat",   "file": "geoip.dat" },
      { "url": "https://raw.githubusercontent.com/aSel1x/geodata/release/geosite.dat", "file": "geosite.dat" }
    ]
  }
}
```

Xray rolls back if the new file fails to load, but it does not verify checksums.

## Changing what gets built

Edit [`geosite/config.json`](geosite/config.json) (a
[domain-list-community](https://github.com/v2fly/domain-list-community) datprofile — `lists`
are filenames from its `data/` directory) or [`geoip/config.json`](geoip/config.json) (a
[v2fly/geoip](https://github.com/v2fly/geoip) config). Opening a pull request runs the full
build and the verification step without publishing anything.

To rebuild on demand: Actions → Build Xray dat files → Run workflow. Tick `skip_release` to
build and verify without cutting a release.

## Attribution

IP data from [DB-IP](https://db-ip.com) IP to Country Lite, licensed
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Domain lists from [v2fly/domain-list-community](https://github.com/v2fly/domain-list-community)
and the geoip builder from [v2fly/geoip](https://github.com/v2fly/geoip), both MIT.
Telegram prefixes come from Telegram's own published list at
[core.telegram.org/resources/cidr.txt](https://core.telegram.org/resources/cidr.txt),
fetched fresh on every build, plus `95.161.64.0/20` — registered to Global Network
Management Inc with Nikolai Durov as contact and Telegram Messenger Inc for abuse, the same
contacts as the published ranges, but absent from the list itself.

The configuration and workflow in this repository are MIT licensed — see [LICENSE](LICENSE).
The generated `.dat` files carry the licences of their sources.
