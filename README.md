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
  artifacts.oml      Document, Table, Matrix, Slide
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

| Davinci construct | OML construct | File |
|---|---|---|
| Base fields (Name, Shortname, Documentation) | `base:Element` + `name`/`shortname`/`documentation` scalar properties | base.oml |
| Ownership tree (parent/children) | `base:Container`, `base:Contained`, `base:Contains` relation entity | base.oml |
| Tags | `base:Taggable`, `base:Tag` concept, `hasTag` relation | base.oml |
| References (source-grounding) | `base:Referenceable`, `reference:Reference` concept, `reference:References` relation entity | base.oml, reference.oml |
| Base relationship: Uses | `base:Uses` relation entity (Element→Element) | base.oml |
| Base relationship: Realized | `base:Realizes` relation entity | base.oml |
| Base relationship: Impact | `base:Impacts` relation entity | base.oml |
| Base relationship: Dependency | `base:DependsOn` relation entity | base.oml |
| Base relationship: Source | `base:HasSource` relation entity | base.oml |
| Base relationship: Result | `base:ResultsIn` relation entity | base.oml |
| Base relationship: Validation | `base:Validates` relation entity | base.oml |
| Base relationship: Reference | `reference:References` relation entity (`base:Referenceable`→`Reference`) | reference.oml |
| Base relationship: Connect | `interface:connectsPort` relation (`Interface`→`Port`) | interface.oml |
| Base relationship: Performs | `behavior:Performs` relation entity (`base:Element`→`Action`) | behavior.oml |
| Base relationship: Subject | `structure:HasSubject` relation entity (`Requirement`→`base:Element`) | structure.oml |
| Base relationship: Mitigation | `values:Mitigates` relation entity (`base:Element`→`Risk`) | values.oml |
| Base relationship: Resource | `values:UsesResource` relation entity (`base:Element`→`Resource`) | values.oml |
| Test/Requirement "verify" relationship | `structure:Verifies` relation entity (`Test`→`Requirement`) | structure.oml |
| Part | `structure:Part` concept | structure.oml |
| Package (+ Access private/public) | `structure:Package` concept + `AccessKind` scalar/`access` property | structure.oml |
| Entity | `structure:ExternalEntity` concept (`entity` is an OML keyword; `@rdfs:label "Entity"` preserves the Davinci name) | structure.oml |
| Requirement | `structure:Requirement` concept (status is evaluation-time, not modeled as a property — see Not captured) | structure.oml |
| Test (status, verdict, category, weight, resultsValue) | `structure:Test` concept + matching scalar properties; `TestStatus` scalar for the closed status lifecycle | structure.oml |
| Capability (startDate/endDate) | `structure:Capability` concept + `startDate`/`endDate` string properties | structure.oml |
| Linkage | `structure:Linkage` concept | structure.oml |
| Attribute (expression, unit, kind) | `values:Attribute` concept < `Valued` < `Expressible`; `AttributeKind` scalar for Number/Boolean/String/Date | values.oml |
| Constraint (expression) | `values:Constraint` concept < `Expressible` | values.oml |
| Resource (isConsumable, useEffectiveLaborHours) | `values:Resource` concept < `Valued`, boolean properties | values.oml |
| Risk (probability, impact, detectability, combinator, criticality) | `values:Risk` concept + properties; `RiskCombinator` scalar; `RiskCriticality` OML rule (probability × impact) | values.oml |
| Document (owns Tables) | `artifacts:Document` concept, `restricts all base:contains to Table` | artifacts.oml |
| Table | `artifacts:Table` concept | artifacts.oml |
| Matrix | `artifacts:Matrix` concept | artifacts.oml |
| Slide | `artifacts:Slide` concept | artifacts.oml |
| Action (sequence, calls) | `behavior:Action` concept + `sequence` property + `Calls` relation entity | behavior.oml |
| State (entry/do/exit actions, interval, Initial/Final) | `behavior:State` concept + `HasEntryAction`/`HasDoAction`/`HasExitAction` relation entities; `Initial`/`Final` specializations with `transitionsFrom`/`transitionsTo` max-0 restrictions | behavior.oml |
| Transition (kind, conditional, probability, multi-target) | `behavior:Transition` relation entity (State→State, non-functional `to`) + `TransitionKind` scalar + `conditional`/`probability` properties | behavior.oml |
| Task (dates, duration, prerequisites) | `behavior:Task` concept + date/duration properties + `Prerequisite` relation entity | behavior.oml |
| Port (direction) | `interface:Port` concept + `PortKind` scalar (`in`/`out`/`inout`) | interface.oml |
| Interface (Connect/Topology) | `interface:Interface` concept + `connectsPort` relation (topology ordering not modeled) | interface.oml |
| Item | `interface:Item` concept + `transfers` relation (Interface→Item) | interface.oml |
| Code (language, imports) | `analysis:Code` concept < `CodeBearing`; `imports` (`Imports` relation entity) | analysis.oml |
| Toolbox | `analysis:Toolbox` concept < `CodeBearing` | analysis.oml |
| Reference (citation metadata) | `reference:Reference` concept + `authors`/`title`/`publication`/... scalar properties | reference.oml |

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
