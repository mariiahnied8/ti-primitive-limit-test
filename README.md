# TI Primitive Completeness Test

This repository is a deterministic Scanner Services fixture for validating
Security Enrichment V2 after the corresponding Scanner Services and TI Scanner
changes have been deployed.

## Expected inventory

| Kind | Expected unique primitives | Fixture |
| --- | ---: | --- |
| URL | 10,250 | `urls.txt` |
| IP address | 10,253 | `ips.txt` and the three boundary files |
| Certificate | 1,000 | `certificates.pem` |

`urls.txt` crosses both the former 4,096 V2 primitive boundary and the URL
scanner's 10,000 occurrence-reporting threshold. Every URL uses the IANA
example domain.

`ips.txt` contains 10,250 unique documentation IPv6 addresses in one file. It
crosses the 10,000 per-file location-evidence threshold while still expecting
all 10,250 unique identities in the V2 artifact. Three additional IP fixtures
exercise the streaming changes:

- `ip-after-former-8mib.txt` places `2001:db8:1::1` at byte offset 8,389,625,
  after the former 8 MiB per-file boundary.
- `ip-after-former-64k-line.txt` places `2001:db8:2::1` at byte offset 70,001
  on one line, after the former 64 KiB line boundary.
- `ip-crosses-64k-window.txt` starts `2001:db8:3::1` at byte offset 65,531 so
  the address crosses the 65,536-byte streaming-window boundary.

`certificates.pem` contains 1,000 unique valid synthetic ECDSA P-256
certificates and no private keys. It crosses the former 256-PEM-block boundary
while remaining below the 4 MiB certificate-container limit.

## Expected deployed scan behavior

- V2 emits 10,250 URL, 10,253 IP-address, and 1,000 certificate identities.
- No primitive-count, PEM-block-count, per-file, or long-line truncation drops
  an identity.
- IP location evidence may be bounded, but the complete unique IP inventory is
  retained.
- The scan succeeds while the exact serialized UTF-8 artifact remains within
  the tenant limit. New API deployments default to 64 MiB; a new TI worker
  talking to an older API intentionally retains a 32 MiB compatibility limit.

Scanning this fixture only tests the PR when the target environment contains
the PR build. Pushing the source branch alone does not deploy it.

## Validate the source fixture

```powershell
$urls = Get-Content .\urls.txt
$ips = Get-Content .\ips.txt
$pem = Get-Content .\certificates.pem -Raw

@{
    UrlLines = $urls.Count
    UniqueUrls = ($urls | Sort-Object -Unique).Count
    IpLines = $ips.Count
    UniqueInventoryIps = ($ips | Sort-Object -Unique).Count
    CertificateBlocks = ([regex]::Matches($pem, '-----BEGIN CERTIFICATE-----')).Count
    ContainsPrivateKey = $pem -match 'PRIVATE KEY'
}
```

Expected values are 10,250 URL lines, 10,250 unique URLs, 10,250 IP lines,
10,250 unique inventory IPs, 1,000 certificate blocks, and no private key.

After downloading `security-enrichment.v2.json`, summarize emitted identities:

```powershell
$artifact = Get-Content .\security-enrichment.v2.json -Raw | ConvertFrom-Json
$artifact.primitives | Group-Object kind | Sort-Object Name |
    Select-Object Name, Count
```

Generated SARIF and enrichment artifacts are ignored.
