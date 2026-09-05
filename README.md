# EdgeQuant Record Seals

Daily, tamper-evident seals of the EdgeQuant public signal record.

## Files

- **`predictions_public.json`** — the public signal record. **Every signal we sent is here,
  including losses and no-trade calls — no row is left out.** Columns are limited to the
  record itself (what was sent, and how it ended); internal model columns are not published.
  Excluded, by name, so you know exactly what is missing rather than having to guess:
  `features`, `filters`, `model_detail`, `consensus`, `transition`, `trans_label`, `conf`,
  `conf_pass`, `size_mult`, `feature_date`, `forming`, `early_stop`, `chart_admin`,
  the free-text `note` / correction memos, the stop price for the direction that was *not*
  signalled, and alternative-exit research columns (`*_other`).
  Rows carry `corrected: true` when a row was later corrected, so corrections stay visible.
- **`deleted_ids.json`** — ids removed from the record. One so far:
  `20260619101047_NQ`, generated off-grid (10:10:47 instead of the 10:00 slot) on
  2026-06-19, a US early-close session. We publish the fact of removal rather than
  letting the gap speak for itself.
- **`anchors/<stamp>.json`** — that day's seal sheet: the record's SHA-256 fingerprint,
  a one-line digest of the private member-ledger section, and the root hash that binds them.
  A second seal on the same day carries a time suffix (e.g. `2026-09-05T1452Z`) —
  **seals are only ever added, never replaced.**
- **`anchors/<stamp>.tsr`** — RFC 3161 timestamp token for that root, signed by FreeTSA
  (freetsa.org). The signed time lives inside the token; we cannot backdate it.
- **`anchors/<stamp>.ots`** — OpenTimestamps proof anchoring the same root in the Bitcoin
  blockchain.

## Verify it yourself

```bash
# 1. the record's fingerprint must equal track_record.sha256 in the seal sheet
shasum -a 256 predictions_public.json

# 2. the root must follow from public values alone (seal rule v2)
python3 -c "import hashlib,json;a=json.load(open('anchors/2026-09-05T1452Z.json'));\
print(hashlib.sha256(f\"{a['date']}|L:{a['ledgers_digest']}|track:{a['track_record']['sha256']}\".encode()).hexdigest()==a['root'])"

# 3. the signed time comes from FreeTSA, not from us
openssl ts -reply -in anchors/2026-09-05T1452Z.tsr -text | head

# 4. and the same root is anchored in Bitcoin
ots verify anchors/2026-09-05T1452Z.ots
```

Fetch FreeTSA's certificates from freetsa.org directly, not from us.

**Seal rules.** Under rule **v2** (from `2026-09-05T1452Z` on) the root is computed as
`sha256("<date>|L:<ledgers_digest>|track:<record fingerprint>")`, so anyone can recompute it
from the published values. Seal sheets before that use rule **v1**, whose root binds
per-member ledger identifiers that stay private — for those, the token still proves the time
of the root, but you cannot recompute the root from this repository alone.

This repository exists so that copies of our fingerprints live outside our control — its
commit history is a third-party-hosted timeline. Plain-language explanation:
https://app.edgequant.app/?nav=history

Not financial advice. https://edgequant.app
