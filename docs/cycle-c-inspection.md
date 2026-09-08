# Cycle C — Adversarial inspection

Target order from the experiment: `7b5f6d8` (BIP-352 future SegWit), then wallet-export ranges, then PSBT output derivation (`hodlDeclaredOutput` / #194).

Write the human notes first. Do not read the agent notes below until your sealed pass is written.

## Human sealed notes (Gregory)

Tried:

Held (failed as intended):

Looked broken:

Did not try:

## Agent sealed notes

Tried (against `7b5f6d8` / [`src/js/bip352.js`](../src/js/bip352.js)):

- Mixed eligible + native future witness programs: v2 32-byte (existing test), v2 2-byte, v16 40-byte; prevout as `{ scriptPubKey.hex }`, raw hex string, and `Uint8Array`.
- Future-only vin; P2WSH v0 mixed in; OP_1 with a 2-byte program (SegWit v1, not > 1); 3-byte `5201aa` (not a witness program); non-canonical `OP_2 OP_PUSHDATA1` shape.
- Esplora-style `prevout.scriptpubkey` (lowercase, no `.hex`).
- Published BIP-352 vectors and the bitcoin/bips `reference.py` (the reference does not implement the future-SegWit skip; this gate is extra vs the published vectors).

Held (failed as intended):

- Every canonical native witness program with version 2–16 made send throw `BIP-352 v0 cannot send with an input that spends a future SegWit version` and made scan return the empty skip. Hex-string and byte prevouts matched the object form.
- Shapes that are not BIP-141 witness programs (too short, non-canonical push, v0 P2WSH, unknown v1) did not trip the gate; send still produced the published vector output from the eligible vins. That matches “SegWit version > 1”, not “any unusual script”.

Looked broken:

- Esplora `scriptpubkey` throws `Cannot read properties of undefined (reading 'hex')` instead of the future-SegWit error. Fail-closed (no silent-payment outputs), but it is a parser hole, not a BIP-352 send/scan disagreement. Not taken as the cycle miss.

Did not try:

- P2SH-wrapped future witness redeem scripts (Core’s usual check is `IsWitnessProgram` on the prevout scriptPubKey, not after BIP-16 unwrap).
- UI paste paths beyond `hodlSpParseVins` passing the array through unchanged.

Fallback `677b681` (wallet-export ranges): displayed `addressBranches` indexes plus the 1000-address gap match the new test. No proven Core-range miss from committed fixtures.

Fallback #194 (`hodlDeclaredOutput`): **proven miss.** The function scanned only `PSBT_OUT_BIP32_DERIVATION` (type 2). A false `PSBT_OUT_TAP_BIP32_DERIVATION` (type 7 / BIP-371) naming this session wallet returned `null`, so the inspector never printed “PSBT lies … Do not sign”. The same lie on type 2 already returned `lie`. Taproot wallets commonly emit type 7 and omit type 2.

## Join / inspectability work

- **Agent found:** type-7 taproot derivation lies were invisible to `hodlDeclaredOutput`.
- **Human found:** pending (section above).
- **Both missed (until the human pass):** the original #194 tests and the `7b5f6d8` future-SegWit test never constructed a BIP-371 output derivation record.

The type-7 hole is the documented miss that changes a test (and the scan in `hodlDeclaredOutput`).
