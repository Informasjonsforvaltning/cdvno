# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not a software project** — it is the source for **CDV-NO** (Content Description Vocabulary - Norwegian specification), an RDF/semantic-web specification maintained by Digitaliseringsdirektoratet (Norwegian Digitalisation Agency). The deliverable is an HTML (and PDF) document built from AsciiDoc sources and published to GitHub Pages.

The specification is written in **Norwegian Bokmål**, with most normative text mirrored in English (italicised). Preserve this bilingual convention when editing — table rows typically pair a Norwegian label/value with its English equivalent.

## Building the document

The build uses [Asciidoctor](https://asciidoctor.org/). There is no local toolchain config; the canonical commands (run from repo root) mirror the CI workflow:

```bash
# HTML — output goes to docs/index.html
asciidoctor -D docs -o index.html -a lang=nb docs/main.adoc

# PDF — output goes to docs/cdvno.pdf
asciidoctor-pdf -D docs -o cdvno.pdf -a lang=nb docs/main.adoc
```

`-a lang=nb` selects Norwegian Bokmål labels (see `docs/locale/`). There are no tests or linters.

## Deployment

`.github/workflows/deploy-cdvno-Github-Pages.yml` builds and deploys automatically. It triggers on **push to `develop` that touches `docs/**`** (or manual `workflow_dispatch`), builds HTML + PDF with the commands above, and publishes `./docs` to the `gh-pages` branch. The default branch is `develop`. Changes outside `docs/` do not trigger a rebuild.

## Document structure

Everything is included transitively from `docs/main.adoc`. The include tree is the architecture:

- `main.adoc` — root: front matter, doc attributes (`:toc:`, `:doctype: book`, etc.), includes `om_denne_versjon.adoc`, `om_denne_spesifikasjon.adoc`, then the two normative parts.
- `Prosatekst-krav.adoc` — **Part 1 (`[[del1]]`)**: prose/conceptual requirements for content descriptions, implementation-independent. Metadata fields described as sorted tables (mandatory / recommended / optional).
- `RDF-krav.adoc` — **Part 2 (`[[del2]]`)**: how Part 1 maps to RDF. Includes `navnerom.adoc` (namespaces/prefixes), `leserveiledning.adoc` (reading guide), `forenklet-modell.adoc`, and one `klassen*.adoc` file per RDF class.

### The `klassen*.adoc` class files

Each RDF class lives in its own file (`klassenBlokk.adoc`, `klassenDatatjeneste.adoc`, `klassenDistribusjon.adoc`, `klassenOrganisasjon.adoc`, `klassenRegel.adoc`, `klassenRegulativRessurs.adoc`, `klassenRessurs.adoc`). They follow a strict, repeated template — match it exactly when adding a class or property:

1. Section header with anchor, e.g. `== Klassen Blokk (cdvno:Block) [[Block]]`.
2. A class diagram (`image::images/cdvno-*.png[]`) with a figure anchor.
3. A class-summary table (`[cols="30s,70d"]`) with rows: _English name_, Anvendelse/_Usage note_, URI, Subklasse av/_Subclass of_, Kravnivå/_Requirement level_, Merknad/_Note_.
4. Property subsections grouped by requirement level (`=== Obligatoriske egenskaper`, recommended, optional), each property as a `[cols="30s,70d"]` table: _English name_, URI, Verdiområde/_Range_, Anvendelse/_Usage note_, Multiplisitet/_Multiplicity_, Kravnivå/_Requirement level_, Merknad/_Note_.
5. A cumulative RDF Turtle example after each property, fenced with `-----`.

### Conventions

- **Anchors & cross-refs**: Use `[[anchor]]` on headings/tables and `<<anchor>>` to reference them. Class anchors use the English class name (`[[Block]]`); property anchors use Norwegian kebab-case (`[[Blokk-dato-sist-oppdatert]]`).
- **Prefixes/namespaces**: All RDF prefixes are declared once in `navnerom.adoc`. The spec's own namespace is `cdvno:` → `https://informasjonsforvaltning.github.io/cdvno#` (a placeholder; the final official URI is still TBD). Reference external vocabularies (dcat, dct, adms, etc.) via these prefixes consistently.
- **Editorial markers**: Text wrapped in `##...##` (highlight) marks open questions / unresolved decisions. `#text#` is also used for status highlighting. Don't silently resolve these — they are intentional.
- **External links** use the role `role="ext-link"` and `window="_blank"` with the `&#x29C9;` glyph, e.g. `https://...[Label &#x29C9;, window="_blank", role="ext-link"]`.
- Escaped literal URIs in prose/Turtle are prefixed with a backslash (`` `\https://...` ``) to stop Asciidoctor auto-linking.

## Examples

`docs/examples/` holds runnable Turtle examples (`example-minimum.ttl`, `example-dummy.ttl`) and their rendered HTML. These are linked from the prose and should stay consistent with the property definitions in the `klassen*.adoc` files.

## Shared / locale

- `docs/locale/attributes.adoc` + `attributes-nb.adoc` — Asciidoctor built-in label translations (Bokmål). Activated by `-a lang=nb`. Keep `attributes-nb.adoc` free of blank lines (Asciidoctor requirement).
- `docs/shared/` — `docinfo`-injected fragments: `download.adoc` (PDF download tip), `stats.adoc` (Google Analytics, HTML backend only). `docs/docinfo.html` is injected via `:docinfo: shared`.