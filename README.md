# TI Primitive Limit Test

This repository is a deterministic Scanner Services test fixture.

`urls.txt` contains exactly 10,250 unique HTTPS URL primitives. This is above
both the former 4,096 V2 primitive boundary and the URL scanner's 10,000
occurrence-reporting threshold. All entries use the IANA example domain and
differ only by a numbered path.

Expected Security Enrichment V2 behavior:

- URL coverage `discovered`: `10250`
- URL primitives in the artifact: `10250`
- No `primitive-count` limit failure
- Additional URLs after occurrence 10,000 retained as inventory
- Success while the serialized artifact remains within the tenant's configured
  byte ceiling, which defaults to 32 MiB

Quick fixture validation in PowerShell:

```powershell
$urls = Get-Content .\urls.txt
$urls.Count
($urls | Sort-Object -Unique).Count
```

Both commands should return `10250`.

The repository intentionally contains no application code or dependencies.
Generated SARIF scan results are ignored.
