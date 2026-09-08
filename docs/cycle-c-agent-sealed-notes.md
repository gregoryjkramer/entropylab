# Cycle C — agent sealed notes (BIP-352 future SegWit)

**Verdict: no miss.**

Inspected commit `7b5f6d8` (`fix: reject future SegWit inputs in BIP352`, upstream OogaBoogaX #318) on this fork only. Gregory is writing separate sealed notes; this file is independent. No coordination, no invented miss, no PR.

Branch: `cursor/cycle-c-agent-sealed-notes-964f`  
HEAD at inspection: `7b65890` (`rock`, includes `7b5f6d8` and `677b681`)  
Scope: send/scan disagreement with BIP-352 v0 — wrong throw, wrong skip, or computing a silent-payment output that v0 must ignore.

Calculator only. Committed fixture keys only. No `crypto.getRandomValues`, no network, no entropy generation.

---

## Target change (what is on disk)

`src/js/bip352.js:162` (not exported):

```js
const isFutureSegwit = (spk) =>
  spk.length >= 4 && spk.length <= 42 &&
  spk[0] >= 0x52 && spk[0] <= 0x60 &&
  spk[1] === spk.length - 2;
```

Matches BIP-352 / BIP-141 witness-program shape: version `OP_2`–`OP_16` (`0x52`–`0x60`), program length 2–40, length byte equals remaining bytes.

Gate sites:

- Send: `createSilentPaymentOutputs` at `src/js/bip352.js:384–386` throws  
  `BIP-352 v0 cannot send with an input that spends a future SegWit version.`
- Scan: `scanSilentPaymentOutputs` at `src/js/bip352.js:468–470` returns  
  `{ outputs: [], inputPubKeySum: null, tweak: null, sharedSecret: null }`
- Both gates run **before** `eligibleInputKeys` (`:387` send, `:473` scan).

Prevout decode (`src/js/bip352.js:354`):

```js
const vinPrevoutScript = (vin) =>
  vin.prevout instanceof Uint8Array
    ? vin.prevout
    : hexToBytes(typeof vin.prevout === "string" ? vin.prevout : vin.prevout.scriptPubKey.hex);
```

UI path (`src/js/app.js:8938–8944` `hodlSpParseVins`) is `JSON.parse` of a vin array (or `{ vin: [...] }`). It does not reshape prevouts. The silent-payment send/verify buttons call `createSilentPaymentOutputs` / `scanSilentPaymentOutputs` only (`app.js:9044`, `:9058`). They never call `eligibleInputKeys` directly.

Existing coverage: `test/bip352.test.mjs:193–219` appends one synthetic vin  
`prevout.scriptPubKey.hex = "5220" + "33"*32` (OP_2, 32-byte program) onto **vector 0** (`"Simple send: two inputs"`, fixture lines 3–110). Asserts send throw + scan hard-skip.

---

## Fixtures (published vectors)

File: `test/fixtures/bip352-send-and-receive.json` only. 28 comments. 19 unique `prevout.scriptPubKey.hex` values.

Prefix inventory (no other prefixes exist in the file):

| Prefix | Meaning | Count |
| --- | --- | --- |
| `76a9` | P2PKH | 7 |
| `5120` | P2TR | 6 |
| `0014` | P2WPKH | 3 |
| `a914` | P2SH | 3 |

**Zero** fixture scripts match `isFutureSegwit`. There is no published-vector future-SegWit case. Gate coverage is synthetic only.

Skip-shaped published vectors that are *not* future SegWit (used as contrast, not as future-gate inputs):

- `"P2PKH and P2WPKH Uncompressed Keys are skipped"` (line 2623) — ineligible keys, send returns empty outputs, does **not** throw the future-SegWit error.
- `"Skip invalid P2SH inputs"` (line 2755) — same class: ignore those vins, do not skip the tx.
- `"No valid inputs, sender generates no outputs"` (line 2992) — empty outputs, no future throw.
- `"Input keys sum up to zero / point at infinity: sending fails, receiver skips tx"` (line 3090) — different skip reason.
- `"Single recipient: taproot input with NUMS point"` (line 2357) — `extractInputPubKey` drops the NUMS vin (`test/bip352.test.mjs:168–174`).

---

## Probe method

Local Node script (`/tmp/cycle-c-probe.mjs`, not committed) imported `src/js/bip352.js` and reused vector 0’s committed keys / recipients / expected x-only output `3e9fce73d4e77a4809908e3c3a2e54ee147b9312dc5044a193d1fc85de46e3c1` (fixture lines 44, 86, 101). 49 assertions. 49 pass.

Pass = send/scan agrees with BIP-352 v0 for that shape. Fail would have been: future script computes an output, non-future script gets the future throw/hard-skip, or send/scan disagree with each other.

---

## Category 1 — witness versions and program lengths

Predicate plus send/scan on `vector0.vins + one extra vin`.

**Failed as intended (future → send throw + scan hard-skip):**

| Script | Shape |
| --- | --- |
| `5202` + 2 bytes | OP_2, min program length 2 |
| `5228` + 40 bytes | OP_2, max program length 40 |
| `5220` + 32 bytes | existing test shape (`test/bip352.test.mjs:200`) |
| `6002` + 2 bytes | OP_16, min length |
| `6020` + 32 bytes | OP_16, 32-byte program |
| `6028` + 40 bytes | OP_16, max length |
| future-only (no eligible siblings) | same throw / hard-skip; does not fall through to the empty-keys path at `:388` / `:474` |

**Failed as intended (not future → must not throw/skip; vector 0 output still computed):**

| Script | Why it is not future |
| --- | --- |
| `5201` + 1 byte (3-byte script) | too short (`len >= 4` fails) |
| `5200` (2-byte script) | too short |
| `5229` + 41 bytes (43-byte script) | too long (`len <= 42` fails) |
| `5220` + 31 bytes | length byte 32 ≠ remaining 31 |
| `5220` + 33 bytes | length byte 32 ≠ remaining 33 |
| `5120` + 32 bytes | OP_1 P2TR (`isP2tr` at `:161`); version 1 is not future |
| `5114` + 20 bytes | OP_1, not P2TR (wrong program length), not future |
| `5102` + 2 bytes | OP_1, 2-byte program |
| `5128` + 40 bytes | OP_1, 40-byte program |
| `0014` + 20 bytes | OP_0 P2WPKH |
| `0020` + 32 bytes | OP_0 P2WSH (ineligible input, not a skip-tx) |
| `0002` + 2 bytes | OP_0 non-v0-wallet shape |
| `0028` + 40 bytes | OP_0, 40-byte program |
| `6120` + 32 bytes | first byte past OP_16 |
| `524c02` + 2 bytes | non-minimal PUSHDATA1; not a BIP-141 witness program |

No miss. Edges of `isFutureSegwit` match BIP-352 v0 / BIP-141. OP_1 that is not P2TR is an ignored input, not a transaction skip. OP_0 non-P2WPKH is the same.

---

## Category 2 — prevout encodings the UI/JSON path feeds

`vinPrevoutScript` accepts three encodings. All three, plus uppercase hex, trip the gate on `5220`+32 bytes and still compute vector 0 when the prevouts are the published P2PKHs:

| Encoding | Future gate | Eligible vector 0 |
| --- | --- | --- |
| `{ scriptPubKey: { hex } }` (fixture / UI paste) | throw + hard-skip | output `3e9fce73…` |
| hex string `vin.prevout` | throw + hard-skip | output `3e9fce73…` |
| `Uint8Array` `vin.prevout` | throw + hard-skip | output `3e9fce73…` |
| uppercase hex string | throw + hard-skip | (not re-run; decode is case-insensitive at `hexToBytes`, `:21`) |
| mixed object + hex-string siblings | — | output `3e9fce73…` |

JSON.parse in `hodlSpParseVins` cannot produce a `Uint8Array`; that encoding is library-only. Fixture-shaped objects are the UI path.

**Non-fixture encodings (observed, not counted as a miss):**

| Shape | What happens |
| --- | --- |
| `prevout.scriptPubKey` as a bare hex string (no `.hex`) | `Invalid hexadecimal input.` on both send and scan |
| `0x`-prefixed hex string | same parse error |
| `scriptPubKey` on the vin root, no `prevout` | `Cannot read properties of undefined (reading 'scriptPubKey')` |

These are not BIP-352-shaped inputs. They do not compute a silent-payment output. They also do not emit the future-SegWit message. That is a parse failure, not a v0 send/scan disagreement.

---

## Category 3 — mixed vins

| Mix | Result |
| --- | --- |
| vector 0 two P2PKH + future OP_2/32 | send throw, scan hard-skip (same as `test/bip352.test.mjs:193`) |
| future-only | send throw, scan hard-skip |
| NUMS vector vins + future | `extractInputPubKey` flags on the NUMS sending vins were `[false, true, false]`; adding future still throw/skip. The future vin is in the “extract would ignore anyway” class — without the gate, the remaining eligible taproot key would still form a shared secret. With the gate it does not. |
| vector 0 + native P2WSH `0020`+32 bytes (not future) | still computes `3e9fce73…` / scan finds 1. Correct: P2WSH is ignored, not a skip-tx. |
| uncompressed-skip vector + future | throws future-SegWit (does not silently take the empty-outputs path). Correct: a future vin forces skip-tx even when every *other* vin is already ineligible. |

No miss.

---

## Category 4 — send throw vs scan skip alignment

Checked OP_2/32, OP_16/2, OP_2/2, OP_2/40, too-short `5201cc`, OP_1 20-byte.

For every future shape: send throws the future-SegWit error **and** scan returns the hard-skip object.  
For every non-future shape: send does not throw that error **and** scan does not hard-skip (vector 0 output still produced).

Throw vs empty-outputs: BIP-352 v0 says the sender must not create silent-payment outputs for a future-SegWit tx. This commit implements that as a throw (send) and a hard-skip object (scan). Other skip reasons in the same file return empty outputs without throwing (`:388` no eligible keys, `:400` zero sum, `:429` `K_max`). That is an API choice, not a v0 disagreement. The committed test (`test/bip352.test.mjs:203–206`) requires the throw. Not recorded as a miss.

---

## Category 5 — `eligibleInputKeys` after the future-SegWit gate

`eligibleInputKeys` (`src/js/bip352.js:365–381`) has **no** future-SegWit check. On `vector0 + future` it still returns 2 pubkeys — the same two as vector 0 alone. The future vin itself yields `extractInputPubKey(...) === null` (no overlap between `isFutureSegwit` and `isP2pkh` / `isP2sh` / `isP2wpkh` / `isP2tr`).

Looked like a hole. It is not a send/scan miss:

- `createSilentPaymentOutputs` throws at `:384` before `:387`.
- `scanSilentPaymentOutputs` returns the hard-skip at `:468` before `:473`.
- UI (`app.js:9044`, `:9058`) never calls `eligibleInputKeys`.
- The published-vector loop in `test/bip352.test.mjs:72` *does* call `eligibleInputKeys` first, but no fixture vin is future SegWit, so that loop never exercises the ungated helper on a skip-tx.

After the gate fires, those sibling keys cannot contribute to an output. That is the property BIP-352 v0 requires.

---

## What looked broken (and why it is not a miss)

1. **`eligibleInputKeys` still extracts sibling keys** when a future vin is present. See category 5. Not on the send/scan path after the gate.
2. **Published fixtures never mention future SegWit.** A later official vector that expects empty send outputs (no throw) would currently fail `test/bip352.test.mjs:79` because `createSilentPaymentOutputs` throws. That is a hypothetical fixture-shape mismatch, not a present published-vector break, and not a case of computing a forbidden output.
3. **Non-fixture prevout shapes throw parse errors** instead of the future-SegWit string. They do not produce outputs. Out of scope for “BIP-352-shaped input.”

Nothing in the probe computed `3e9fce73…` (or any other output) for a vin set that contained a BIP-141 future witness program.

---

## What I did not try

- Network fetch of `bitcoin/bips` `send_and_receive_test_vectors.json` to diff against the committed fixture. Air-gapped rule; fixture file only.
- Browser paste through `#sp-send-vins` / `#sp-verify-vins`. Library send/scan plus `hodlSpParseVins` is a straight `JSON.parse`; no second decoder.
- P2SH redeemScript that *looks* like a future witness program. BIP-352 v0 tests the prevout `scriptPubKey`, not the redeem. A P2SH prevout is `a914…87` and is not `isFutureSegwit`. Not constructed.
- Generating keys, scanning chain, or any `crypto.getRandomValues` for key material.
- Opening a PR to OogaBoogaX, merging, or coordinating with Gregory.
- Rebuilding WASM or running the browser suite. Not needed for this library predicate.

---

## Optional one-time fallback: `677b681`

One read of `fix: preserve wallet export address ranges` (`src/js/wallet-export.js:147–167`, test at `test/wallet-export.test.mjs:361–395`). Then stopped.

What the commit does: `walletdescriptor` `next_index` / `range_start` / `range_end` follow displayed address indexes (`addressBranches[].rows` or `account.receive` / `account.change`), with `rangeEnd = min(0x7fffffff, maxIndex + 1000)`. The new test checks indexes 5000–5002 → `{ nextIndex: 5003, rangeStart: 5000, rangeEnd: 6002 }` and change 7000 → `{ 7001, 7000, 8000 }`.

No proven miss against that committed assertion. Did not invent empty-`rows` / sparse-index cases. Did not import a wallet.dat into Bitcoin Core.

---

## Result

**No miss** on `7b5f6d8` against BIP-352 v0 and the committed fixtures.  
No test change. No draft PR. Sealed notes only.
