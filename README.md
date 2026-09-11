# geodata

Nightly `geosite.dat` and `geoip.dat` for [Xray](https://github.com/XTLS/Xray-core) and
V2Ray, built from two small config files in this repo.

| file | contents | size |
|---|---|---|
| `geosite.dat` | `youtube` `telegram` `openai` `google-gemini` `instagram` `discord` `anthropic` | ~8 KB |
| `geoip.dat` | `ru` (IPv4), `private` | ~130 KB |

Built every night at 00:00 UTC. Each build is verified — minimum size plus one assertion per
section named in the configs — before anything is published.

## Download

Pick the mirror, not the file. All three serve identical bytes.

```
# jsDelivr — Cloudflare, no redirect. Use this one from Russia.
https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/geoip.dat
https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/geosite.dat

# GitHub release — immutable per-tag archive, honours If-Modified-Since
https://github.com/aSel1x/geodata/releases/latest/download/geoip.dat
https://github.com/aSel1x/geodata/releases/latest/download/geosite.dat

# raw — same Fastly IPs as the release assets, so it is a convenience, not a fallback
https://raw.githubusercontent.com/aSel1x/geodata/release/geoip.dat
```

`release-assets.githubusercontent.com`, `raw.githubusercontent.com` and
`objects.githubusercontent.com` all resolve to `185.199.108-111.133`. jsDelivr resolves to
Cloudflare. If GitHub's content pool is unreachable on your network, only jsDelivr helps.

Checksums sit next to each file (`.sha256sum`), and
[`version.json`](https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/version.json) on the
`release` branch carries the build timestamp and the upstream commits it was built from — a
200 with a valid checksum does not by itself mean the data is current.

```bash
curl -fsSLO https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/geoip.dat
curl -fsSLO https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/geoip.dat.sha256sum
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
      { "url": "https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/geoip.dat",   "file": "geoip.dat" },
      { "url": "https://cdn.jsdelivr.net/gh/aSel1x/geodata@release/geosite.dat", "file": "geosite.dat" }
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

The configuration and workflow in this repository are MIT licensed — see [LICENSE](LICENSE).
The generated `.dat` files carry the licences of their sources.
