## Docling Exploration — Outreachy 2026 Contribution
- **Issue:** [Issue #122 — Docling: explore document processing basics](https://forge.fedoraproject.org/commops/interns/issues/122)
- **Author:** Dohou Daniel Favour
- **Date:** 2026-03-27
- **Task:** Docling - Document Processing Basics (Exploration)

---

### Background and Purpose

**[Docling](https://www.docling.ai/)** is the document processing layer that makes this possible. Before a language model can reason about a PDF, that PDF must be converted into clean, structured text. Docling does that conversion. It is not a simple text extractor — it is a document understanding system that uses machine learning models to identify layout regions, reconstruct table structure, determine reading order, and produce output optimised for downstream AI workflows.

This task establishes a working understanding of Docling's CLI, its output formats, its processing options, and the trade-offs between them — all of which are directly relevant to how Ramalama's RAG pipeline behaves in practice.

---

## Environment

| Component | Version |
|---|---|
| OS | Linux 6.14.0-37-generic (Ubuntu) x86\_64 |
| Python | CPython 3.12.3 |
| pip | 24.0 |
| docling | **2.82.0** |
| docling-core | 2.70.2 |
| docling-ibm-models | 3.12.0 |
| docling-parse | 5.6.1 |
| GPU | None (CPU-only inference) |

All commands were run inside a Python virtual environment (`venv`). The environment was activated with:

```bash
source venv/bin/activate
```
<br>

---

## Source Document

**File:** `pytorch-conference.pdf`
**Size:** 4.7 MB
**Content:** PyTorch Conference 2026 — Sponsorship Prospectus (Paris, France, 7–8 April 2026)

This document was chosen because it is an excellent stress-test for a document processing pipeline:

- **Multi-column layout** — pages use side-by-side column arrangements
- **Large sponsorship comparison table** — a 7-column, 20-row table listing benefits across Diamond, Gold, Silver, Bronze, Startup, and Non-Profit tiers with pricing from $4,000 to $50,000
- **Multiple embedded images** — logos, graphics, decorative elements (25 images total)
- **Mixed content** — text blocks, headings, lists, figures, and structured tables all on the same pages
- **Real-world document** — not a synthetic test; produced by a professional typesetter

A sponsorship brochure is more challenging than a plain text document and more representative of the kinds of documents a RAG system would need to process in production.

---