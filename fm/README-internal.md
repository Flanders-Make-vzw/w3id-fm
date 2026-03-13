# Internal Notes — `w3id.org/fm` .htaccess

## Overview

This folder provides permanent identifiers under `https://w3id.org/fm/` for Flanders Make ontologies. All requests are redirected to `https://ontology.flandersmake.be/`.

## How the .htaccess works

### Global settings

- **`Options -MultiViews`** — disables Apache's automatic content negotiation so only our explicit rewrite rules apply.
- **`Vary: Accept`** header — tells caches that responses differ by `Accept` header (important for content negotiation).

### Rule A — File requests with extensions (line 18)

`/fm/{ontology}/{path/to/file.ext}` → `https://ontology.flandersmake.be/{ontology}/{path/to/file.ext}`

Examples:
- `w3id.org/fm/aqume/index_en.html` → `ontology.flandersmake.be/aqume/index_en.html`
- `w3id.org/fm/aqume/style.css` → `ontology.flandersmake.be/aqume/style.css`
- `w3id.org/fm/aqume/sections/x.html` → `ontology.flandersmake.be/aqume/sections/x.html`

### Rule B — Root-level RDF file shorthand (line 24)

`/fm/{ontology}.{ttl|rdf|owl|jsonld|json|nt|nq|trig}` → `https://ontology.flandersmake.be/{ontology}/{ontology}.{ext}`

Examples:
- `w3id.org/fm/aqume.ttl` → `ontology.flandersmake.be/aqume/aqume.ttl`
- `w3id.org/fm/aqume.jsonld` → `ontology.flandersmake.be/aqume/aqume.jsonld`

### Rule C — Concept/entity IRIs (line 30)

`/fm/{ontology}/{ConceptName}` → `https://ontology.flandersmake.be/{ontology}/index_en.html#{ConceptName}`

Examples:
- `w3id.org/fm/aqume/Sensor` → `ontology.flandersmake.be/aqume/index_en.html#Sensor`

Note: any deeper path segments after the concept name are silently discarded (only the first segment after the ontology name is used as the fragment).

### Rule D — Content negotiation on the ontology IRI (lines 36–55)

`/fm/{ontology}` or `/fm/{ontology}/` redirects based on the `Accept` header:

| Accept header               | Redirects to                          |
|-----------------------------|---------------------------------------|
| `text/turtle`               | `…/{ontology}/{ontology}.ttl`         |
| `application/rdf+xml`       | `…/{ontology}/{ontology}.rdf`         |
| `application/ld+json`       | `…/{ontology}/{ontology}.jsonld`      |
| `application/n-triples`     | `…/{ontology}/{ontology}.nt`          |
| anything else (HTML fallback) | `…/{ontology}/index_en.html`        |

## Assessment

### What works well

- Content negotiation logic is correct and follows the standard pattern for ontology publishing.
- Rule ordering is proper — specific matches (file paths, extensions, concepts) come before the conneg fallback.
- `NE` (no-escape) flag is correctly used on rules that produce fragment identifiers (`#`).
- All redirects are `302` (temporary), which is appropriate during development and gives flexibility to change targets later.

### Things to be aware of

1. **`text/plain` triggers N-Triples** — the `Accept` condition for N-Triples also matches `text/plain`. A plain `curl` without an explicit `Accept` header sends `*/*` (which won't match), but a client explicitly sending `text/plain` would get N-Triples rather than HTML. This is a deliberate and generally acceptable choice.

2. **No `application/json` in conneg** — Rule B supports `.json` as a file extension, but Rule D has no `application/json` negotiation entry. If you serve plain JSON alongside JSON-LD, consider adding a conneg entry for it.

3. **Consider `303` instead of `302`** — For ontology IRIs (Rules C and D), the Linked Data / httpRange-14 convention recommends `303 See Other` to indicate "this resource is described at another location." Consider switching once the setup is stable.
