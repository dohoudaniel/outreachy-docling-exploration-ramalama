## Docling Exploration: My Outreachy 2026 Contribution
- **Issue:** [Issue #122: Docling: explore document processing basics](https://forge.fedoraproject.org/commops/interns/issues/122)
- **Author:** Dohou Daniel Favour
- **Date:** 2026-03-27
- **Task:** Docling - Document Processing Basics (Exploration)

---

### Background and Purpose

**[Docling](https://www.docling.ai/)** is the document processing layer that makes this possible. Before a language model can reason about a PDF, that PDF must be converted into clean, structured text. Docling does that conversion. It is not a simple text extractor: it is a document understanding system that uses machine learning models to identify layout regions, reconstruct table structure, determine reading order, and produce output optimised for downstream AI workflows.

This task establishes a working understanding of Docling's CLI, its output formats, its processing options, and the trade-offs between them: all of which are directly relevant to how Ramalama's RAG pipeline behaves in practice.

---

### Environment For This Task

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

---

### Source Document

- **File:** `pytorch-conference.pdf`
- **Size:** 4.7 MB
- **Content:** PyTorch Conference 2026: Sponsorship Prospectus (Paris, France, 7–8 April 2026)

This document was chosen because it is an excellent stress-test for a document processing pipeline:

- **Multi-column layout**: pages use side-by-side column arrangements
- **Large sponsorship comparison table**: a 7-column, 20-row table listing benefits across Diamond, Gold, Silver, Bronze, Startup, and Non-Profit tiers with pricing from $4,000 to $50,000
- **Multiple embedded images**: logos, graphics, decorative elements (25 images total)
- **Mixed content**: text blocks, headings, lists, figures, and structured tables all on the same pages
- **Real-world document**: not a synthetic test; produced by a professional typesetter

A sponsorship brochure is more challenging than a plain text document and more representative of the kinds of documents a RAG system would need to process in production.

![Pytorch Conference PDF 1](./screenshots-for-docs/4-pdf-download.png)

![Pytorch Conference PDF 2](./screenshots-for-docs/7-pdf-download.png)

---

### About the Warnings

Every run produced these warnings:

```
[W327 13:54:08.913322637 NNPACK.cpp:56] Could not initialize NNPACK!
Reason: Unsupported hardware.
```

And occasionally:

```
WARNING docling.models.stages.ocr.rapid_ocr_model: RapidOCR returned empty result!
```

**These are not errors.** They do not affect output quality.

- **NNPACK warning**: NNPACK is an optional CPU acceleration library for PyTorch. This machine's CPU does not support the required instruction sets. PyTorch falls back to standard CPU operations automatically. This warning appears on every run and can safely be ignored.

- **RapidOCR empty result**: Docling's OCR engine (RapidOCR) tried to extract text from a region: likely a decorative image or logo: and found no readable text. This is expected behaviour for graphical elements that contain no text. The rest of the document is unaffected.

---

### Documentation Of All Steps I Carried Out To Complete This Task, Using Docling

### Step 1: Installation of Docling

Docling was installed inside a Python virtual environment to keep dependencies isolated.


```bash
# Create and activate the virtual environment
python3 -m venv venv
source venv/bin/activate

# Install docling
pip install docling
```

![Installation Screenshot](./screenshots-for-docs/1-installation.png)

Docling pulls in a substantial dependency tree including PyTorch, torchvision, and several IBM Research model packages. This is because Docling uses deep learning models for layout analysis and table structure recognition: it is not a lightweight text extractor.

To verify the installation:

```bash
pip show docling
```

Output:

```
Name: docling
Version: 2.82.0
Summary: SDK and CLI for parsing PDF, DOCX, HTML, and more, to a unified
         document representation for powering downstream workflows such as
         gen AI applications.
Author: 
        Author-email: Christoph Auer <cau@zurich.ibm.com>, Michele Dolfi <dol@zurich.ibm.com>, Maxim Lysak <mly@zurich.ibm.com>, Nikos Livathinos <nli@zurich.ibm.com>, Ahmed Nassar <ahn@zurich.ibm.com>, Panos Vagenas <pva@zurich.ibm.com>, Peter Staar <taa@zurich.ibm.com>
License: 
        Location: ./venv/lib/python3.12/site-packages
Requires: accelerate, beautifulsoup4, certifi, defusedxml, docling-core, docling-ibm-models, docling-parse, filetype, huggingface_hub, lxml, marko, openpyxl, pandas, pillow, pluggy, polyfactory, pydantic, pydantic-settings, pylatexenc, pypdfium2, python-docx, python-pptx, rapidocr, requests, rtree, scipy, torch, torchvision, tqdm, typer
```

---

### Step 2: Display the Version

```bash
docling --version
```

Output:

```
Docling version: 2.82.0
Docling Core version: 2.70.2
Docling IBM Models version: 3.12.0
Docling Parse version: 5.6.1
Python: cpython-312 (3.12.3)
Platform: Linux-6.14.0-37-generic-x86_64-with-glibc2.39
```

`--version` reports not just the top-level `docling` package but all sub-packages in the Docling ecosystem:

- **docling**: the CLI and SDK entry point
- **docling-core**: the unified document model (`DoclingDocument`) and shared data structures
- **docling-ibm-models**: the ML models (layout analysis, table structure recognition)
- **docling-parse**: the PDF parsing backend

![Installation Confirmation](./screenshots-for-docs/3-installation.png)


I also aim to get myself familiar with the `man page` of docling:

![Docling Help](./screenshots-for-docs/8-docling-help.png)

---

### Step 3: Default Conversion (Markdown)

```bash
docling pytorch-conference.pdf
```

Since no `--output` was specified initially, the output was written to the current directory, then moved:

```bash
mv pytorch-conference.md output/default/
```

Or equivalently with `--output`:

```bash
docling pytorch-conference.pdf --output ./output/default/
```

- **Output:** `output/default/pytorch-conference.md`
- **File size:** 1.2 MB
- **Line count:** 281

#### Why Markdown is the Default for Docling Conversion

Markdown is the default output format because it serves RAG pipelines better than any other format at the intersection of three needs:

1. **LLM comprehension**: Language models trained on internet data have seen enormous amounts of Markdown and parse it natively. Headings, tables, and emphasis are meaningful to the model.
2. **Chunking structure**: RAG systems split documents into chunks before embedding. Markdown heading hierarchy (`#`, `##`, `###`) gives chunkers natural, semantically meaningful split points.
3. **Human verifiability**: A developer can open a `.md` file and immediately verify conversion quality. JSON or binary formats require additional tooling to inspect.
4. **Structured but not noisy**: HTML has hundreds of tags that are irrelevant to meaning. JSON has schema overhead. Plain text loses all structure. Markdown preserves headings, tables, lists, and emphasis with minimal syntax. An LLM trained on the internet has seen enormous amounts of Markdown and handles it natively.

![Default Conversion](./screenshots-for-docs/9-default-conversion.png)

![Default Conversion](./screenshots-for-docs/10-default-conversion.png)

---

### Step 4: HTML Output

```bash
docling pytorch-conference.pdf --to html --output ./output/html/
```

![HTML Conversion](./screenshots-for-docs/11-html-conversion.png)

![HTML Conversion](./screenshots-for-docs/12-html-conversion.png)

- **Output:** `output/html/pytorch-conference.html`
- **File size:** 1.2 MB

#### What Changes

The PDF dociment is converted and is rendered to HTML:

- Section headers → `<h1>`, `<h2>`, `<h3>`
- Body text → `<p>`
- Tables → `<table>` with proper `<tr>`, `<th>`, `<td>` structure
- Images → `<img src="data:image/png;base64,...">` (embedded by default)

The HTML file is **self-contained**: one file that renders completely in any browser with no external dependencies.

**File size comparison:**

| Format | Size | Notes |
|---|---|---|
| Markdown | 1.2 MB | 25 base64 images embedded |
| HTML (embedded) | 1.2 MB | Same image data, additional HTML tags |

Both files are approximately the same size because both embed the same 25 images as base64 data. The HTML tags add negligible overhead compared to the image data.

---

### Step 4b: HTML with Referenced Images

```bash
docling pytorch-conference.pdf --to html --image-export-mode referenced \
  --output ./output/html-referenced-image-export-mode/
```

- **Output:** `output/html-referenced-image-export-mode/pytorch-conference.html` + 25 PNG files
- **HTML file size:** 22 KB
- **PNG artifacts total:** ~900 KB
- **Combined total:** ~922 KB

![HTML Conversion (Image Compressed)](./screenshots-for-docs/22-image-export-mode.png)

#### What Changes

With `--image-export-mode referenced`, Stage 8 performs two operations instead of one:

1. Each image is extracted from the `DoclingDocument`, rendered as a PNG file, and written to an `_artifacts/` subdirectory with a hash-based filename.
2. The HTML file references those PNG files with relative `<img src="...">` tags instead of embedding base64 data.

The HTML file shrinks from 1.2 MB to **22 KB**: a **~55x reduction** in the primary file size.

**Image export modes available:**

| Mode | HTML file | Image data location | Self-contained? |
|---|---|---|---|
| `embedded` (default) | ~1.2 MB | Inside the HTML as base64 | Yes |
| `referenced` | ~22 KB | Separate PNG files in `_artifacts/` | No |
| `placeholder` | Smallest | Not exported at all | N/A |

#### Why This Matters for RAG

In a RAG pipeline, you typically do not embed images into your text index. You either skip them or process them through a separate vision model. The `referenced` mode gives you that separation automatically:

- The HTML file contains clean text and structure: fast to parse and index
- The PNG files are available separately for a vision model to process if needed
- No base64 decoding overhead when reading the document programmatically

For Ramalama running on a personal laptop, processing a 22 KB HTML file is dramatically more efficient than loading a 1.2 MB file into a text processing pipeline.

**Trade-off:** The HTML file is no longer self-contained. Moving it without its PNG siblings breaks all images. This is acceptable in a pipeline context where file relationships are managed by the system.

---

### Step 5a: JSON Output

```bash
docling pytorch-conference.pdf --to json --output ./output/json/
```

- **Output:** `output/json/pytorch-conference.json`
- **File size:** 6.5 MB

![JSON Conversion](./screenshots-for-docs/15-json-conversion.png)

![JSON Conversion](./screenshots-for-docs/15-json-conversion.png)

#### What JSON Contains

JSON export does not produce a rendered document. It serialises the internal `DoclingDocument` model directly to disk: the same structure Docling uses internally before producing any other format.

Every element carries its full metadata:

```json
{
  "schema_name": "DoclingDocument",
  "version": "...",
  "texts": [
    {
      "label": "section_header",
      "text": "About PyTorch Conference",
      "prov": [{ "page_no": 1, "bbox": { "l": 72.0, "t": 340.0, "r": 540.0, "b": 355.0 } }]
    }
  ],
  "tables": [
    {
      "data": {
        "grid": [
          [{ "text": "DIAMOND 4 AVAILABLE", "is_header": true, ... }]
        ]
      }
    }
  ]
}
```

**Preserved in JSON but absent in Markdown:**
- Exact bounding box of every element on every page
- Semantic label for every element (`section_header`, `text`, `caption`, `table`, `figure`, `footnote`)
- Page number for every element
- Full table cell grid with row/column indices and header flags
- Document hierarchy (which heading owns which paragraphs)
- Base64-embedded images (accounts for the large file size)

#### Why JSON is Largest

| Format | Size | Reason |
|---|---|---|
| Text | 26 KB | Pure text content, no images, no metadata |
| Markdown | 1.2 MB | Text + structure + 25 base64 images |
| HTML | 1.2 MB | Text + HTML tags + 25 base64 images |
| YAML | 6.4 MB | Full document model + all metadata + images |
| JSON | 6.5 MB | Full document model + all metadata + images |

JSON is largest because it contains everything: all the structural intelligence from the ML pipeline (bounding boxes, labels, confidence data) plus the image data.

#### When to Use JSON

JSON is the format for **programmatic RAG pipelines**. Frameworks like LangChain and LlamaIndex consume the JSON document model to:
- Filter elements by semantic label (e.g., extract only tables)
- Know exactly which page a chunk came from (for citations and references)
- Build intelligent chunking strategies based on heading hierarchy
- Process text and images through separate pipelines

---

### Step 5b: Text Output

```bash
docling pytorch-conference.pdf --to text --output ./output/text/
```

- **Output:** `output/text/pytorch-conference.txt`
- **File size:** 26 KB

![Text Conversion](./screenshots-for-docs/17-text-conversion.png)

#### What Text Contains

Text output writes only the string content of each element in reading order, with no formatting markers:

```
7-8 April 2026 | Paris, France
2026 SPONSORSHIP PROSPECTUS
<!-- image -->
About PyTorch Conference
...
```

- Headings lose their `#` syntax: they are indistinguishable from body text
- Tables are **completely flattened**: cell structure is lost
- Images are replaced with `<!-- image -->` comments (not actual image data)
- File size drops to 26 KB because no image data is exported

From the docling CLI help: *"Text, DocTags, and WebVTT outputs do not export images."*

#### What Gets Lost

The sponsorship table: the most knowledge-dense part of this document: becomes unreadable in text format. The 7-column, 20-row structure that clearly maps benefits to sponsorship tiers collapses into a sequence of strings with no cell boundaries. An LLM reading this output cannot reliably answer "How many attendee passes does a Gold sponsor receive?" because the row-column relationship is gone.

#### When to Use Text

Text output is appropriate only for **simple prose documents** with no tables or complex layout: a policy document, a letter, a novel. For any document where tables carry knowledge, text output is the wrong choice for RAG.

---

### Step 5c: YAML Output

```bash
docling pytorch-conference.pdf --to yaml --output ./output/yaml/
```

- **Output:** `output/yaml/pytorch-conference.yaml`
- **File size:** 6.4 MB

![YAML Conversion](./screenshots-for-docs/18-yaml-conversion.png)

#### What YAML Contains

YAML export serialises the identical `DoclingDocument` as JSON export. The content is the same. Only the serialisation syntax differs: no curly braces, no quotes on keys, indentation-based structure:

```yaml
- label: section_header
  text: About PyTorch Conference
  prov:
    - page_no: 1
      bbox:
        l: 72.0
        t: 340.0
        r: 540.0
        b: 355.0
```

#### JSON vs YAML

| Dimension | JSON | YAML |
|---|---|---|
| Content | Identical | Identical |
| File size | 6.5 MB | 6.4 MB |
| Human readability | Moderate | Higher |
| Python stdlib support | `import json` | Requires `pyyaml` |
| Parsing edge cases | Minimal | Known issues (e.g. `no` → `False`) |
| Ecosystem fit | APIs, web services | Config files, CI/CD, ML tools |

For Ramalama's RAG pipeline in Python, **JSON is the practical choice**: no extra dependency, no edge-case parsing risks. YAML is preferable when a downstream tool in the pipeline expects YAML input natively.

---

### Step 6: Legacy Pipeline

```bash
docling pytorch-conference.pdf --pipeline legacy --output ./output/legacy/
```

**Output:** `output/legacy/pytorch-conference.md`
**File size:** 28 KB
**Line count:** 281

![Legacy Conversion](./screenshots-for-docs/20-legacy-flag-conversion.png)

![Legacy Conversion](./screenshots-for-docs/21-legacy-flag-conversion.png)

#### What the `--pipeline` Flag Does

The `--pipeline` flag selects the entire ML stack used for document processing. Available options are `legacy`, `standard` (default), `vlm`, and `asr`.

- **`standard`** — the current pipeline using IBM Research's latest layout analysis and table structure models
- **`legacy`** — the older generation pipeline from before the current models were adopted
- **`vlm`** — vision-language model pipeline (uses models like SmolDocling or Granite)
- **`asr`** — automatic speech recognition pipeline for audio/video files

### Finding: The Legacy Pipeline Does Not Generate Images

The most striking result of this experiment is the **file size**: 28 KB vs 1.2 MB for the standard pipeline — a **43x reduction**.

Examining the output reveals why. Where the standard pipeline embeds base64 image data:

**Standard pipeline output:**
```markdown
![Image](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA...)
```

**Legacy pipeline output:**
```markdown
<!-- 🖼️❌ Image not available. Please use `PdfPipelineOptions(generate_picture_images=True)` -->
```

The legacy pipeline replaces every image with an HTML comment placeholder. It detects that images exist at those positions in the document, but does not render or encode them. All 25 images across the document become 25 identical-looking placeholder comments.

This is not a conversion error — it is a capability difference between pipeline generations. The legacy pipeline was not built to extract and embed picture content.

#### Text and Table Content: Structurally Identical

Excluding image lines, the text and table content of the legacy output is **structurally identical** to the standard output. The sponsorship comparison table is reproduced correctly with all 7 columns, all 20 rows, all cell values, and all tier pricing intact.

This means that for a document where **only text and table content matters** (and images can be ignored), the legacy pipeline produces equivalent quality at a fraction of the processing time and output size.

#### Trade-off Summary

| Dimension | Standard | Legacy |
|---|---|---|
| Output file size | 1.2 MB | 28 KB |
| Images | 25 embedded as base64 | 25 placeholders (no image data) |
| Text quality | High | High (identical on this document) |
| Table quality | High | High (identical on this document) |
| Processing time | Slower (heavier models) | Faster (lighter models) |
| Memory footprint | Higher | Lower |

#### Implication for RAG

The legacy pipeline is a legitimate choice when:
- The document is text and table-heavy with decorative images that carry no semantic content
- Hardware is constrained (no GPU, limited RAM)
- Processing throughput is more important than image fidelity

For the PyTorch sponsorship brochure, the images are logos and decorative graphics — they do not carry answerable knowledge. A RAG system querying this document for sponsorship tier information would perform identically whether images were present or not. The legacy pipeline's 28 KB output is faster to load, faster to chunk, and faster to embed than the 1.2 MB standard output.

---

### Step 6: Fast Table Mode

```bash
docling pytorch-conference.pdf --table-mode fast --output ./output/table-fast/
```

**Output:** `output/table-fast/pytorch-conference.md`
**File size:** 1.2 MB
**Line count:** 283 (vs 281 for default — 2 extra lines due to cell overflow errors)


#### What `--table-mode` Does

The `--table-mode` flag selects the model used for table structure recognition in Stage 4 of the pipeline:

- **`accurate`** (default) — uses the full TableFormer model, a transformer-based architecture trained on scientific and technical documents. Handles merged cells, borderless tables, multi-level headers.
- **`fast`** — uses a lighter model variant that relies more heavily on visible grid lines and whitespace alignment. Faster, but degrades on complex table structures.

#### Finding: Concrete, Verifiable Table Errors

A direct diff of `output/default/pytorch-conference.md` vs `output/table-fast/pytorch-conference.md` reveals specific structural errors in the fast mode output.

**Accurate mode (correct):**
```
| Promotion of Activity in Sponsor Booth: A session... | Promotion of (2) in-booth activities/ time slots | Promotion of (1) in-booth activity/ time slot | ...
| Attendee Registration Contact List: Opt-in only | ✔ (List provided pre and post event) | ✔ (List provided post event) | ...
| Social Media Promotion: From PyTorch X handle... | 1 Custom Post, 1 Group Post, and 1 Re-Post | 1 Group Post and 1 Re-Post | ...
```

**Fast mode (corrupted):**
```
| Promotion of Activity in Sponsor Booth: A session, demo... will be | Promotion of (2) in-booth | Promotion of (1) in-booth activity/ |
| communicated by Sponsor Services, and may not overlap conference sessions. | activities/ time slots | time slot |
| Attendee Registration Contact List: Opt-in only Social Media Promotion: From PyTorch X handle. All custom | ✔ (List provided pre and post event) 1 Custom Post, 1 Group Post, and | ✔ (List provided post event) 1 Group Post and |
```

Three specific failures are visible:

1. **Cell text overflow** — "Promotion of Activity in Sponsor Booth" row: the long cell description was split across two rows, with the overflow text appearing as a second row that does not exist in the original document.

2. **Row merging** — "Attendee Registration Contact List" and "Social Media Promotion" were collapsed into a single row. Two distinct rows became one. The data values from both rows were concatenated inside single cells.

3. **Column truncation in later columns** — The "STARTUP UNLIMITED" and "NON-PROFIT UNLIMITED" columns had cell content truncated, with overflow text from adjacent cells bleeding across column boundaries: `"wifi App Only No physical device provided."` appeared as cell content in the wrong column.

These are not cosmetic differences. An LLM answering "What social media promotion is included with a Gold sponsorship?" would produce an incorrect answer from the fast mode output because the data for that row has been merged with adjacent row content.

#### The Extra 2 Lines

The 283 vs 281 line count difference is explained by the cell overflow errors: one row was split into two rows, adding 2 lines to the output. More lines, but less information — the opposite of what you want.

#### Trade-off Summary

| Dimension | `accurate` (default) | `fast` |
|---|---|---|
| Table structure | Correct | Errors on complex tables |
| Cell overflow handling | Correct | Splits or merges cells |
| Multi-column tables | Reliable | Fragile |
| Processing time | Slower | Faster |
| Output line count | 281 | 283 (2 overflow lines) |

#### Implication for RAG

For this document, `--table-mode fast` produces incorrect data in the primary knowledge structure (the sponsorship table). This is the highest-risk setting in the pipeline for table-heavy technical documents.

`--table-mode fast` is appropriate only for documents where:
- Tables are simple (clear visible borders, no merged cells, single-level headers)
- Table data is secondary to prose content
- Processing speed is a hard constraint

For any document where tables are the primary knowledge source, `--table-mode accurate` is mandatory.

---

### Step 7: Force OCR with Profiling

```bash
docling pytorch-conference.pdf --force-ocr --profiling --output ./output/force-ocr/
```

**Output:** `output/force-ocr/pytorch-conference.md`
**File size:** 1.2 MB
**Line count:** 281

![Forced OCR Profiling](./screenshots-for-docs/23-force-ocr-mode.png)

![Forced OCR Profiling](./screenshots-for-docs/24-force-ocr-mode.png)

#### What `--force-ocr` Does

Under default behaviour, Docling's OCR engine (RapidOCR) runs **selectively** — only on regions that have no native PDF text, such as scanned image areas or figures with embedded text. For a digitally typeset document like `pytorch-conference.pdf`, OCR runs on very few regions because the text layer is already present and complete.

`--force-ocr` overrides this logic. For every region on every page, regardless of whether native text exists, Docling:
1. Rasterises the page area to a bitmap
2. Runs RapidOCR on that bitmap
3. Replaces the native PDF text with OCR output

#### Why This Is the Wrong Tool for This Document

`pytorch-conference.pdf` is a **digitally native** PDF — produced by a typesetter or layout tool, not scanned from paper. Its text layer is perfect: every character is exactly correct, including Unicode symbols (✔), typographic punctuation, and special characters.

When `--force-ocr` is applied, that perfect text layer is discarded in favour of OCR output from pixel recognition. OCR working from bitmaps introduces recognition errors that did not exist in the source:

- Visually similar characters confused: `l` / `I` / `1`, `O` / `0`
- Unicode symbols may be garbled or dropped: `✔` may become `v` or be lost
- Small text (footnotes, fine print) falls below reliable OCR resolution
- Typographic ligatures (`fi`, `fl`) common in typeset PDFs may be split or misread

#### What `--profiling` Shows

The `--profiling` flag does not change processing — it instruments it, printing a timing breakdown per pipeline stage after the run completes. This provides hard numbers rather than subjective impressions.

On a CPU-only machine (no GPU), the OCR stage under `--force-ocr` is dramatically slower than under default settings:

- **Default run**: OCR runs on a handful of image-only regions — fast
- **Force-OCR run**: OCR runs on every region of every page — the OCR stage time increases by an order of magnitude

The profiling output distinguishes between:
- PDF parsing time
- Layout analysis time
- Table structure recognition time
- OCR time ← this is where `--force-ocr` shows its cost
- Assembly time
- Export time

Running `--profiling` on the default run alongside the force-OCR run provides a direct quantitative comparison of the OCR stage cost.

#### The Core Lesson

`--force-ocr` is not "more thorough" on a native PDF — it is actively harmful. It replaces known-good text with potentially erroneous OCR output and costs significant processing time for zero quality benefit.

`--force-ocr` is the right choice only when:
- The PDF is a **scanned document** with no native text layer
- The native text layer is **corrupt or empty** (some PDFs have garbled or missing text layers)
- You need to override a broken text extraction with OCR as a fallback

For Ramalama processing user-provided PDFs, this means: default OCR settings are correct for typical digitally-produced documents. Reserve `--force-ocr` for documents where you have verified the native text layer is absent or unreadable.

---

### Summary of All Experiments

| Command | Output file | Size | Key characteristic |
|---|---|---|---|
| `docling pytorch-conference.pdf` | `default/pytorch-conference.md` | 1.2 MB | Baseline: standard pipeline, accurate tables, embedded images |
| `--to html` | `html/pytorch-conference.html` | 1.2 MB | Same content, browser-renderable |
| `--to html --image-export-mode referenced` | `html-referenced-image-export-mode/pytorch-conference.html` | 22 KB + 900 KB PNGs | HTML/image separation |
| `--to json` | `json/pytorch-conference.json` | 6.5 MB | Full document model with all metadata |
| `--to text` | `text/pytorch-conference.txt` | 26 KB | Plain text only, all structure lost |
| `--to yaml` | `yaml/pytorch-conference.yaml` | 6.4 MB | Same as JSON, different syntax |
| `--pipeline legacy` | `legacy/pytorch-conference.md` | 28 KB | No image data, text/tables identical |
| `--table-mode fast` | `table-fast/pytorch-conference.md` | 1.2 MB | Visible table errors (cell overflow, row merging) |
| `--force-ocr --profiling` | `force-ocr/pytorch-conference.md` | 1.2 MB | OCR over native text — slower, lower quality |

---

#### Finding 1: Output Format Determines What Information Survives

The most important insight from the format comparison is that each output format represents a different **information retention level**:

```
JSON / YAML          ← Maximum: full document model, bounding boxes, labels, metadata
     ↓
HTML (referenced)    ← High: structure + text, images separated
     ↓
HTML (embedded)      ← High: structure + text + images, self-contained
     ↓
Markdown             ← High: structure + text + images, human-readable
     ↓
Text                 ← Minimal: text content only, all structure discarded
```

The choice of format is not aesthetic — it determines what a downstream RAG system can work with. Markdown and JSON are the two practically useful formats for RAG. Text is useful only for prose-only documents.

---

#### Finding 2: The Legacy Pipeline Does Not Extract Images

The legacy pipeline produces a 28 KB Markdown file where the standard pipeline produces a 1.2 MB file from the same source. The entire difference is image handling:

- **Standard**: 25 images embedded as base64 data → 1.2 MB
- **Legacy**: 25 image placeholders (`<!-- 🖼️❌ Image not available... -->`) → 28 KB

Text content and table structure are **identical** between the two pipelines on this document.

This finding has a nuanced implication: the legacy pipeline is not simply "worse" — it is different in a specific, predictable way. For documents where images carry no answerable knowledge (logos, decorative graphics, charts that are summarised in adjacent text), the legacy pipeline produces smaller, faster-to-process output with no meaningful quality loss.

---

#### Finding 3: `--table-mode fast` Produces Verifiable Errors on Complex Tables

This is the most significant quality finding in the entire experiment set.

The sponsorship comparison table in `pytorch-conference.pdf` is a 7-column, 20-row table with long cell descriptions and no simple single-line cells. The `accurate` mode (using full TableFormer) handles it correctly. The `fast` mode fails in three specific ways:

**Error 1 — Cell text overflow causing row splitting:**
A long cell description that fits in one row under `accurate` mode is split across two rows under `fast` mode. The overflow text becomes an erroneous second row.

**Error 2 — Row merging:**
Two adjacent rows ("Attendee Registration Contact List" and "Social Media Promotion") were collapsed into a single row. Their data values were concatenated inside shared cells. The `Social Media Promotion` row effectively disappeared as a distinct entry.

**Error 3 — Column content bleeding:**
In the last two columns, truncated cell content from one row bled into the cell of the adjacent row: the text `"wifi App Only No physical device provided."` appeared as a cell value in the wrong row.

These errors are not theoretical — they are directly observable by diffing the two output files. An LLM answering questions about this table from the `fast` mode output would produce wrong answers for at least three rows.

---

#### Finding 4: `--force-ocr` Is Harmful on Native PDFs

`--force-ocr` is the only setting tested that **degrades quality on a document it should handle well**. A digitally typeset PDF has a perfect text layer. Forcing OCR over it introduces recognition errors (character confusion, symbol loss, small text degradation) and costs significantly more processing time.

The `--profiling` flag makes this cost quantifiable: the OCR stage time under `--force-ocr` is dramatically higher than under default settings on a CPU-only machine. This is the direct hardware cost of running image recognition over every region of every page when the text was already available for free.

---

#### Finding 5: File Size Is Not Proportional to Information

The largest files (JSON at 6.5 MB, YAML at 6.4 MB) contain the most information. But the second-largest files (Markdown and embedded HTML at 1.2 MB) contain far less structural information than JSON — most of their size is image data.

The text file (26 KB) contains the least information despite being the only format without any image data overhead. It is small because structure was discarded, not because it is efficient.

| Format | Size | Information density |
|---|---|---|
| JSON | 6.5 MB | Highest — full model + images |
| YAML | 6.4 MB | Highest — same as JSON |
| Markdown | 1.2 MB | Medium — structure + images (images dominate size) |
| HTML (embedded) | 1.2 MB | Medium — same as Markdown |
| HTML (referenced) HTML only | 22 KB | Medium — structure only, images external |
| Legacy Markdown | 28 KB | Medium — structure, no images |
| Text | 26 KB | Lowest — content only, no structure |

---

#### How Docling Fits into the RAG Pipeline

Understanding where Docling sits in the larger system makes the trade-offs above concrete:

```
PDF (raw)
    ↓
Docling (document processing)
    ↓
Structured output (Markdown / JSON)
    ↓
Chunker (splits document into pieces at semantic boundaries)
    ↓
Embedding model (converts chunks to vectors)
    ↓
Vector store (stores vectors for similarity search)
    ↓
LLM (answers questions using retrieved chunks)
```

Docling is the first step. Every error it makes propagates through every subsequent step:

- A corrupted table (as seen with `--table-mode fast`) produces bad chunks
- Bad chunks produce bad embeddings
- Bad embeddings surface irrelevant context during retrieval
- Irrelevant context produces wrong answers from the LLM

This is why the task exists: building intuition about Docling's behaviour is directly building intuition about RAG quality.

---

### My Research On How Docling Connects To Ramalama

Based on the findings from all experiments, the following settings are recommended for Ramalama's RAG pipeline:

#### For Typical Digitally-Typeset PDFs (Reports, Brochures, Papers)

```bash
docling <file.pdf> --pipeline standard --table-mode accurate --output ./output/
```

Rationale: Standard pipeline with accurate table mode provides the best quality. On CPU-only hardware, processing time is higher, but the output is correct. For a RAG system where wrong answers have real consequences, quality takes priority over speed.

#### For High-Volume Processing on Constrained Hardware

```bash
docling <file.pdf> --pipeline legacy --output ./output/
```

Rationale: If the documents are text-and-table-heavy with decorative images, the legacy pipeline provides equivalent text/table quality at ~43x smaller output files and faster processing. The trade-off (no image data) is acceptable when images carry no answerable knowledge.

#### For Table-Heavy Technical Documents

```bash
docling <file.pdf> --table-mode accurate --output ./output/
```

Never use `--table-mode fast` for documents where tables are the primary knowledge source. The visible errors in this experiment are exactly the kind of silent corruption that produces wrong RAG answers.

#### For Scanned PDFs Only

```bash
docling <file.pdf> --force-ocr --output ./output/
```

Reserve `--force-ocr` for documents confirmed to have no native text layer. Do not apply it to digitally typeset documents.

#### For Pipeline Integration

```bash
docling <file.pdf> --to json --output ./output/
```

Use JSON output when the processed document will be consumed programmatically — by a framework, a chunker, or a custom RAG implementation. JSON preserves all structural metadata needed for intelligent chunking and source attribution.

---

### Usage Of Artificial Intelligence

For this task, I made sure to play around and read the documentation of Ramalama and Docling. But I used three AI tools as a form of support in deepening my understanding for the completion of this task, and trying out multiple conversion methods:

- **[Docling Dosu](https://app.dosu.dev/)**: I used this AI tool to play around and familiarize myself with the syntax of the Docling tool, and this AI tool is referenced as a means for quicker understanding, of the Docling docs.

- **Claude**: I approached the task iteratively and in a structured way, with Claude as a support tool to help guide my exploration and debugging. I broke the work into smaller steps, starting with installation, then command execution, then output validation, and finally trying different CLI options to compare results. After each step, I checked the output before moving on to the next one. That made it easier to catch issues early, understand the errors more clearly, and improve my approach step by step.

---

### Conclusion

Docling is not a PDF-to-text converter. It is a document understanding system: it uses machine learning models to identify layout regions, reconstruct table structure, determine reading order, and produce output that reflects the semantic structure of the original document rather than just its raw character content.

The experiments in this task demonstrate that:

1. Output format is a first-order decision — it determines what information survives into the RAG pipeline
2. Pipeline and model choices have measurable, concrete effects on output quality
3. More aggressive settings (`--force-ocr`) are not always better — knowing when not to use a feature is as important as knowing how
4. The legacy pipeline is not simply inferior — it represents a valid operating point for constrained environments
5. Table structure quality is the highest-risk dimension for technical documents — errors there corrupt the most knowledge-dense content

These are not abstract observations. Each finding maps directly to a scenario where a RAG system would give a wrong answer if the wrong Docling setting was chosen. That connection between document processing quality and downstream AI quality is the fundamental lesson this task teaches.
