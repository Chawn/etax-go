# PLAN — etax-go

Status: **M0 scaffolded** · Owner: @Chawn · Last updated: 2026-10-06

## Goal
A Go library to **build, validate and sign** Thai e-Tax Invoice & e-Receipt XML documents
following the ETDA standard (ขมธอ. 3-2560 and its later revisions), so Thai ERP/POS/billing
systems written in Go don't each have to decode a 200-page spec. This is niche but
exactly the kind of "infrastructure nobody wants to write" that collects dependents.

## Non-goals
- Being a certified Service Provider or submitting to the Revenue Department on the user's behalf.
- Managing private keys/certificates (users supply a `crypto.Signer`; we never store keys).
- Tax advice. README must say the output should be verified against the official validator.

## ⚠️ Step 1 is reading the spec
Before code, the agent writes `docs/SPEC-NOTES.md`:
- Which ETDA standard version(s) we target and links to the official XSD/schematron files.
- The document types and their codes (to verify — commonly cited: `380` ใบแจ้งหนี้, `388` ใบกำกับภาษี,
  `T01` ใบรับ, `T02` ใบแจ้งหนี้/ใบกำกับภาษี, `T03` ใบเสร็จรับเงิน/ใบกำกับภาษี,
  `T04` ใบส่งของ/ใบกำกับภาษี, `T05` ใบกำกับภาษีอย่างย่อ, `80` ใบเพิ่มหนี้, `81` ใบลดหนี้).
- Required vs optional elements per type; rounding rules for VAT; seller/buyer branch codes (`00000` = head office).
- Submission channels (Service Provider, direct upload, e-Tax Invoice by Email with
  PDF/A-3 + timestamp) — document them, don't implement submission in v0.1.
**Ben reviews SPEC-NOTES.md before M2.**

## Design
- Base on the UN/CEFACT CrossIndustryInvoice structure that ETDA profiles. Generate Go
  structs from the official XSD (record the generator + command in `docs/`), then hand-write a
  friendly builder on top so users never touch the raw structs.
- Money as `int64` satang (or a tiny decimal type with 2–4 dp as the spec requires) — **no float**.
- Signing: XAdES-BES enveloped signature, `crypto.Signer` interface so HSM / PKCS#11 / file keys all work.
  Evaluate `github.com/beevik/etree` + a minimal XMLDSig implementation vs. an existing lib; C14N
  correctness is the hard part — test against the official validator output.

## API (v0.1.0 target)
```go
inv := etax.NewTaxInvoice(etax.T02).
    ID("INV-2026-0001").IssueDate(t).
    Seller(etax.Party{TaxID: "0105...", Branch: "00000", NameTH: "...", Address: ...}).
    Buyer(etax.Party{...}).
    AddLine(etax.Line{Name: "Software development", Qty: 1, UnitPriceSatang: 10_000_00, VAT: etax.VAT7}).
    Build()                       // returns *etax.Document, error (validates business rules)
xmlBytes, _ := inv.MarshalXML()
err := etax.ValidateSchema(xmlBytes)         // embedded XSD
signed, _ := xades.Sign(xmlBytes, signer, certChain)
```

## Milestones
- [x] **M0 — Scaffold**
- [ ] **M1 — SPEC-NOTES.md** + fetch official XSDs into `schema/` (check their license allows redistribution; else download at build time) → **Ben reviews**.
- [ ] **M2 — Models + builder** for T02 (most common), totals/VAT calculation with exhaustive tests.
- [ ] **M3 — Schema validation** (pure Go if feasible; otherwise document `xmllint` path for CI only).
- [ ] **M4 — Remaining doc types** (T01, T03, T04, T05, 80, 81, 388).
- [ ] **M5 — XAdES-BES signing** with test certs generated in tests; cross-check with the official validator.
- [ ] **M6 — PDF/A-3 embedding** helper (optional, separate module).
- [ ] **M7 — Release v0.1.0** with a complete example: build → sign → write files.

## Definition of done
vet, race tests, lint; golden-file tests for XML output (`testdata/golden/*.xml`, update with `-update` flag);
README updated; PLAN ticked.

## Open questions
- Ben: does บริษัท ชาวพุทธ จำกัด (or a client) have a test certificate / access to the ETDA validator to verify output?
