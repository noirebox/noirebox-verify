# noirebox-verify — GitHub Action

**Verify a NoireBox export journal in CI. The job fails if the chain was tampered with.**

The flight recorder meets the pipeline: any repository that consumes AI-agent
decisions can gate its builds on the integrity of the journal it received —
without trusting the producer, the host, or anyone.

## What it checks

The standalone [NoireBox verifier](https://github.com/noirebox/noirebox/blob/main/verifier/verifier.py)
recomputes the entire hash chain (SHA-256), checks every Ed25519 signature,
validates the signed attestation and the RFC 3161 anchors — then exits `0`
(`INTACT`) or `1` (`TAMPERING DETECTED`, with the exact event number).

## Usage

```yaml
- uses: noirebox/noirebox-verify@v1
  with:
    export-path: export.json   # the journal you received from a NoireBox instance
    noirebox-ref: main         # optional — pin a tag for reproducibility
```

The `result` output is `INTACT` or `TAMPERING` for downstream steps.

## Producing an export

From any NoireBox instance (API, SDK or MCP):

```console
$ curl -s https://your-noirebox.example.com/api/v1/export > export.json
```

The export contains every sealed event plus the signed attestation — it is
the same dossier a third party would verify offline. One file, one command,
zero trust.

## Design notes

- **Light by design**: the verifier needs only `cryptography` at runtime —
  no scikit-learn, no model download. The action clones the pinned NoireBox
  ref, installs that single dependency, and runs.
- **Separable layers**: this action proves *integrity*, not business
  completeness — the verifier's contract stays exactly what the
  [threat model](https://github.com/noirebox/noirebox/blob/main/docs/THREAT-MODEL.md)
  says it is.
- **Same tool, both sides**: the producer seals with NoireBox; the consumer
  verifies with the same verifier. Nobody imports a "trust me" SDK.

## License

MIT — see [LICENSE](LICENSE). Part of the
[NoireBox](https://github.com/noirebox/noirebox) project.
