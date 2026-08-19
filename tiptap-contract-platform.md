# Contract Document Platform on TipTap

**Type:** Technical Design
**Status:** Draft for review
**Scope:** Replacing hand-built JSON document templates with TipTap platform primitives for web-native contract authoring

---

## 1. Context

Our contract documents are created, edited, and viewed almost entirely in the web app. PDF is the only export target. Today, each contract type is backed by a hand-authored JSON template that is slow to build and slow to change.

Requirements:

- Documents are web-native. Word is not the system of record.
- Documents have enforced structure (required sections, signature blocks, ordering).
- Every document must record which template, at which version, it was created from.
- Full edit history and auditability per document instance.
- Print-quality PDF export with correct pagination, headers, and footers.

TipTap's "document layouts" (the Pages extension) solve only one slice of this. The full answer uses four platform pieces together: schema extensions, the Document REST API, Snapshots, and Conversion export.

## 2. Architecture

### 2.1 Templates are versioned TipTap JSON seeds

TipTap JSON is the recommended storage and working format, and nodes carry attributes. We exploit that:

- Define a custom `doc` node whose attributes include `templateId`, `templateVersion`, and `contractType`.
- Store each contract form as a semver-versioned TipTap JSON template in a template registry we own.
- On document creation, the backend copies the template JSON into a new document via the Document REST API, stamping the provenance attributes at birth.

The document then permanently self-reports its origin, independent of any external lookup. We also record the mapping in our own database for querying, but the document is the authority.

```json
{
  "type": "doc",
  "attrs": {
    "templateId": "svc-agreement",
    "templateVersion": "2.3.0",
    "contractType": "services"
  },
  "content": [ ... ]
}
```

### 2.2 Structure is enforced by schema, not convention

This is the highest-leverage move for contracts. TipTap supports forced content structure through custom document extensions:

- Define nodes such as `partiesBlock`, `clauseSection`, `signatureBlock`.
- The schema requires them, in order. ProseMirror rejects any transaction that would delete a signature block or reorder required sections. Users cannot break the form.
- Apply the UniqueID extension so every clause node has a stable identifier.

Stable clause IDs also unlock server-side manipulation: the content injection API can target specific nodes while users collaborate in real time, with all changes tracked and compatible with Snapshots. This supports programmatic clause updates and merge-field population.

### 2.3 Instance versioning uses Snapshots

Document history is a platform primitive, not something we build:

- Autoversioning captures versions at configurable intervals; manual versions mark workflow milestones (sent for review, executed).
- Snapshot Compare renders a redline: what changed between versions and who changed it.
- Version metadata is updatable via REST after creation, so versions can be tagged with workflow states.

Constraint: Collaboration, Snapshots, and Snapshot Compare require a TipTap Document server. For our environment, the Enterprise on-premises deployment is the assumed path. Managed features (comments, versions, webhooks) run inside our own infrastructure.

### 2.4 Pages renders the screen, Conversion renders the PDF

- The Pages extension provides the paginated in-app view: page formats, visual page breaks, headers and footers with page numbering.
- There is no browser-print integration. PDF output goes through the Conversion export path.
- The export configuration does not inherit page geometry from the Pages extension. Maintain one shared page-config object that feeds both, so screen and PDF never drift.

Known Pages limitations that apply to contracts:

- No per-page styling or page templates. Every page shares one layout. Distinct cover pages or mid-document margin changes are out of scope today.
- Export produces a single section. Mixed portrait and landscape (for example, a landscape exhibit) cannot be produced.
- Non-splittable blocks taller than a page (large table rows, figures) cause an unstable layout loop. Use PagesTableKit for tables and cap block heights with the provided `--page-max-height` variable.
- Pages is in active development. Pin exact package versions.

## 3. Template migration policy

TipTap will not solve this for us. When template v2.3 becomes v2.4:

- Executed contracts stay on their birth version. This is correct and defensible.
- Drafts either lock to their birth version or migrate. Because documents carry `templateVersion` and clauses carry stable IDs, server-side migrations are possible through the content injection API, but the migration logic is ours to write.

**Recommendation:** lock drafts to their birth version by default. Migration is an explicit, user-initiated action with a Snapshot taken immediately before.

## 4. What this replaces

| Today | Proposed |
|---|---|
| Hand-authored JSON template per contract type | Versioned TipTap JSON seed per contract type |
| Structure by convention | Structure enforced by ProseMirror schema |
| Custom or absent version tracking | Snapshots + Snapshot Compare |
| Template provenance in app database only | Provenance stamped into the document itself |
| Layout logic embedded in each template | One shared page-config feeding Pages and Conversion export |

## 5. Adoption path

1. Prototype with the three most complex contract types, especially anything table-heavy.
2. Build the custom schema (document node attrs, required block nodes, UniqueID).
3. Stand up the on-prem Document server; wire Snapshots and autoversioning.
4. Implement the shared page-config and validate screen-to-PDF fidelity.
5. Define the template registry, semver policy, and draft migration policy.
6. Migrate remaining contract types.

Go/no-go after step 1: if the hardest forms paginate and export cleanly, proceed. If they fight the Pages limitations (multi-section layout, oversized tables), scope the simplification of those forms before platform migration.
