# Davinci Schema OML

An [OML](https://www.opencaesar.io/oml) vocabulary expressing the [Davinci](https://docs.davinci-app.com) MBSE tool's object schema, structured as an [OML Code](https://www.modelware.io/) project modeled on [modelware/sierra-method](https://github.com/modelware/sierra-method).

## 🚀 Already in OML Code?

Open this folder in VS Code with the OML Code extension installed.

## Structure

```
src/method/oml/davinci-app.com/schema/   the Davinci method (vocabularies)
  base.oml         shared aspects, base fields (name/shortname/documentation),
                    Container/Contained/Contains, Taggable/Tag, Referenceable,
                    and the generic base:Relationships types
  structure.oml     Part, Package, ExternalEntity, Requirement, Test,
                    Capability, Linkage
  values.oml        Attribute, Constraint, Resource, Risk
  artifacts.oml      Document, Table, Matrix, Figure, Slide
  behavior.oml       Action, State, Transition, Task
  interface.oml       Port, Interface, Item
  analysis.oml       Code, Toolbox
  reference.oml      Reference
  bundle.oml         vocabulary bundle including all of the above

src/model/oml/anhkos.github.io/example/  a small example instantiating the method
  model.oml          the instance data
  bundle.oml          description bundle wrapping model.oml
```

Each Davinci "visual category" (Structure, Values, Artifacts, Behavior, Interface, Analysis, Reference) is its own vocabulary file; relations live in the file where their source (or best-fit) concept is defined, following the Sierra convention.

## Mapping table: Davinci construct → OML construct

See [docs/MAPPING.md](docs/MAPPING.md) for the full construct-by-construct
mapping table, cross-referenced with each member's exact `@dc:description`
annotation so the two stay in sync.

## Not captured, and why

- **Requirement status (TRUE/FALSE/INCOMPLETE), Test verdict rollup, Resource rollup** — all are evaluation-time behaviors computed from expression/equation evaluation or category-configured rollup modes (All/Minimum/Weighted, sum/max). OML/SWRL cannot evaluate arbitrary Math.js expressions, so only the structural ownership (`base:contains`) and, for Test, the `Verifies` relation are modeled; the computed fields themselves are left as comments. Risk `criticality` is the one exception: since it's simple arithmetic (`probability * impact`), it's captured as a `RiskCriticality` OML rule using `swrlb:multiply`.
- **Attribute array dimensionality** (scalar vs. 1D/2D/3D arrays for Number/Boolean attributes) — not modeled; `expression` is a single string.
- **Attribute state-scoped values** (`[attribute-uuid].state([state-uuid])` syntax) — an expression-language feature, not a structural relation.
- **Interface Topology** (ordered connection groups of ports) — only the flat `connectsPort` membership is modeled, not the grouping/ordering.
- **"Standard Custom Relationships"** (Data, Power, Mechanical, Optical, Chemical, Electromagnetic, Thermal, Fluid) — the docs describe these as project-configurable presets layered on the generic Connect mechanism, with no distinguishing fields given, so they are not modeled as separate relation entities.
- **The "Transition" base relationship type** (common/relationships.md) — overlaps in name with the full `Transition` object type (behavior.oml). The docs don't clarify the relationship between the two, so only the richer object type is modeled.
- **Reference relationship locator** — the docs say a `References` link "describes where in the reference object it found the information" but name no specific field for it, so no locator property is added.
- **Capability start/end date as a Task reference with `useEnd` flag** — only the plain date-string form is modeled, to avoid a structure.oml → behavior.oml dependency for a secondary form.
- **Test Procedure/Results/Prerequisites as references to documents/code/other objects** — the docs don't name a dedicated relation; closest existing base relations (`Uses`, `DependsOn`) are noted in a comment rather than adding new bespoke ones.
- **Package's reserved `LIBRARY`/`MODEL` top-level packages** — a project convention (instance-level), not a schema constraint.
- **Table/Matrix cell- and query-level structure** (formulas, cell references, scope/depth/filters) — UI/expression-language concerns out of scope for a structural vocabulary.

Every concept, property, and relation not listed above traces directly to a statement in the fetched Davinci docs (`https://docs.davinci-app.com/pages/modeling/objects/*` and `common/{relationships,equations,units}`).

## Known tooling limitation

`npx @oml/cli lint`/`reason` in this environment consistently resolved to a stale, unrelated workspace (even after that workspace's files were deleted from disk), so the automated `oml-cli reason` consistency check described in the project instructions could not be completed from the command line in this session. All files were instead validated incrementally via the OML Code VS Code extension's live diagnostics (which flagged and helped fix several real issues: the `uses`, `unit`, and `language` keyword collisions). Before treating the model as final, run `oml reason` (or the VS Code extension's reasoning command) directly.
