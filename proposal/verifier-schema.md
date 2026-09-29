# Verifier response schema

Proposed for the verification pointer object in issue #5 ({url, method}). Field set builds on the straw man from minia2auk, jalcodev, and nohumans.directory in that thread.

## Verdict object

- `verdict`: `true` | `false` | `inconclusive`
- `reason`: a code from a closed, append-only set. Codes are never renumbered, split, or reused with a different meaning, since a persisted audit trail would silently reread as whatever the code means today.
  - `settled-under-terms`, `probe-ok`, `no-domain`, `wrong-domain`, `mapping-unknown`, `mapping-stale`, `terms-mismatch`, `digest-unbound`, `self-referential`
  - `probe-unreachable` — the probe was attempted and got no response. `evidence` records the strongest level reached by a check that completed before the probe; if none completed, `evidence: none`. The unreachable attempt itself never advances `evidence`.
- `evidence`: `none` | `terms` | `probe` | `purchase`, weakest to strongest. Reports the strongest level reached, never downgraded to a fresher weaker check. `none` when no check reached a level, for example when the only check that ran was a digest verification that failed (`reason: digest-unbound`). This is what makes `untested: [terms, probe, purchase]` reachable.
- `detail`: companion to `reason`, `{field, expected, got}`. Required when `reason` is `wrong-domain`, `terms-mismatch`, `mapping-unknown`, or `mapping-stale`, the codes that reference a specific field. Optional otherwise. `reason` stays a closed code; `detail` carries the specifics without growing the code set.
- `observed_at`: when that evidence was obtained, not when the verdict was assembled. A cache-served verdict must not read as freshly observed.
- `checked_at`: the last time the verifier ran any check at all, independent of what evidence level that check produced. Answers "how stale is my picture of this seller." A three-week-old purchase and a probe run a minute ago both report `evidence: "purchase"`, `observed_at` three weeks back, and only `checked_at` shows the minute-old check happened.
- `untested`: subset of `{terms, probe, purchase}`, the complement of `evidence`, not a separate vocabulary. A check that yields no level (e.g. digest recomputed against the live challenge) does not belong here; it belongs in `reason`.
- `digest`: over the stable per-accept terms, `scheme, network, asset, payTo, amount, extra`. `maxTimeoutSeconds` stays out; it's verdict lifetime, not terms. A challenge carrying more than one accept has one digest per accept; `accepts_index` (below) says which one this verdict applies to. Exactly one accept is assumed only when `accepts_index` is absent.
- `mapping`: `{entry: "<scheme>/<network>/<asset>", contract: "0x…", read_at: "<ts>"}`.
- `accepts_index`: integer index into the challenge's `accepts` array. Required on every verdict; a single-accept challenge carries `0`. A challenge with more than one accept must name which one this verdict, digest, and mapping apply to, since two accepts can share `(scheme, network, asset)` and still differ in `payTo` or `amount`.
- `attestation`: `{key_url, record_url, subject: {resource_domain, verifier_domain}, valid_until}`, a signed record making the independence claim below checkable. `subject` names the two identities being compared: the resource's own domain and the verifier's. A client discounts the attestation if `verifier_domain` is not distinct from `resource_domain`, or if `key_url`/`record_url` do not resolve under `verifier_domain`. `valid_until` bounds how long the record is trusted.

## Recommendation object (separate, cites the verdict)

`recommendation` is not part of the verdict object. Verifiers that emit a policy judgment (pay/caution/avoid or similar) put it here, citing the verdict rather than replacing it. `verdict: false` means "the claim was refuted," not "do not use." Every risk threshold lives in the client's own policy, and two verifiers can agree on evidence while recommending differently without either being wrong.

## Client guidance

A client SHOULD prefer a verification object independent of the resource operator over one the operator names in its own record. A self-referential pointer (verifier and resource operator share ownership, infrastructure, or configuration control) SHOULD be treated the same as an absent verification object: it MUST NOT block the payment, and its verdict carries `reason: self-referential`. A self-referential pointer pairs with `verdict: inconclusive`, not `false`. A verifier sharing ownership with the resource operator has not refuted anything about the resource; it has failed at being independent, which is what `inconclusive` is for and matches the bias toward inconclusive over a false pass stated elsewhere in this schema. Where enforceability matters, `attestation` (above) lets a client check independence against a published artifact rather than trust the pointer's self-declaration alone.
