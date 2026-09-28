# Worked examples

## 1. nohumans.directory

Free record lookup at `GET /v1/resolve?url=`, paid verdict over x402 at `GET /v1/verdict/{id}`. Today emits `pay | caution | avoid` with prose reasons; under this schema that becomes the `recommendation` object citing a `verdict` built from the closed reason set.

## 2. Generic example

```json
{
  "verdict": "false",
  "reason": "wrong-domain",
  "evidence": "probe",
  "observed_at": "2026-09-28T18:40:00Z",
  "checked_at": "2026-09-28T18:40:00Z",
  "untested": ["purchase"],
  "digest": {
    "scheme": "exact",
    "network": "eip155:8453",
    "asset": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "payTo": "0x2076045...",
    "amount": "10000",
    "extra": { "name": "USDC", "version": "2" }
  },
  "mapping": {
    "entry": "exact/eip155:8453/0x833589f...",
    "contract": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    "read_at": "2026-09-28T00:00:00Z"
  }
}
```

## 3. Paddock (verify_before_pay)

Reads the live 402 and decodes payTo/chain/asset off the challenge itself, biased toward inconclusive over a false pass.

- settled-under-terms / probe-ok ← route: true, split by whether an independent paid-fulfillment record exists
- wrong-domain, terms-mismatch ← route: false, with the mismatched field named
- inconclusive ← route_state: "inconclusive", always carrying untested rather than defaulting to a pass
- observed_at ← timestamp of the strongest check that ran; checked_at ← timestamp of this call regardless of level reached
- digest ← the decoded challenge fields
- `attestation` ← `attest: true` issues a signed record fetchable at a URL independent of the checked resource's own domain; `key_url` points at Paddock's published signing key, `record_url` at the specific attestation.
