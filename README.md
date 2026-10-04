# TI Primitive Limit Test

This repository is a deterministic Scanner Services test fixture.

`urls.txt` contains exactly 4,100 unique HTTPS URL primitives, four above the
Security Enrichment V2 schema limit of 4,096 primitives. All entries use the
IANA example domain and differ only by a numbered path.

Expected artifact-limit telemetry:

- Limit: `primitive-count`
- Actual: `4100`
- Maximum: `4096`

The repository intentionally contains no application code or dependencies.
