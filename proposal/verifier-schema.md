# Verifier response schema

Proposed for the verification pointer object in issue #5 ({url, method}). Field set builds on the straw man from minia2auk, jalcodev, and nohumans.directory in that thread.

## Verdict object

- `verdict`: `true` | `false` | `inconclusive`
- `reason`: a code from a closed, append-only set — codes are never renumbered, split, or reused with a different meaning, since a persisted audit trail would silently reread as whatever the code means today.
  - `settled-under-terms`, `probe-ok`, `no-domain`, `wrong-domain`, `mapping-unknown`, `mapping-stale`, `terms-mismatch`, `probe-unreachable`, `digest-unbound`, `self-referential`
- `evidence`: `terms` | `probe` | `purchase`, weakest to strongest — the strongest level reached, never downgraded to a fresher weaker check.
- `observed_at`: when that evidence was obtained, not when the verdict was assembled. A cache-served verdict must not read as freshly observed.
- `checked_at`: the last time the verifier ran any check at all, independent of what evidence level that check produced. Answers "how stale is my picture of this seller" — a three-week-old purchase and a probe run a minute ago both report `evidence: "purchase"`, `observed_at` three weeks back, and only `checked_at` shows the minute-old check happened.
- `untested`: subset of `{terms, probe, purchase}` — the complement of `evidence`, not a separate vocabulary. A check that yields no level (e.g. digest recomputed against the live challenge) does not belong here; it belongs in `reason`.
- `digest`: over the stable per-accept terms — `scheme, network, asset, payTo, amount, extra`. `maxTimeoutSeconds` stays out; it's verdict lifetime, not terms.
- `mapping`: `{entry: "<scheme>/<network>/<asset>", contract: "0x…", read_at: "<ts>"}`.

## Recommendation object (separate, cites the verdict)

`recommendation` is not part of the verdict object. Verifiers that emit a policy judgment (pay/caution/avoid or similar) put it here, citing the verdict rather than replacing it. `verdict: false` means "the claim was refuted," not "do not use." Every risk threshold lives in the client's own policy, and two verifiers can agree on evidence while recommending differently without either being wrong.

## Client guidance

A client SHOULD prefer a verification object independent of the resource operator over one the operator names in its own record. A self-referential pointer (verifier and resource operator share ownership, infrastructure, or configuration control) SHOULD be treated the same as an absent verification object: it MUST NOT block the payment, and its verdict carries `reason: self-referential`.
