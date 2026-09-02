# Knowledge Contract — adempiere-dashboard-improvements

## 1. Identity

| Field | Value |
|---|---|
| Name | adempiere-dashboard-improvements |
| Repository Type | Library |
| Classification basis | Declared as `type: Library` in `.ai/repository.yml`. |
| Standards | knowledge-contract-v1, repository-classification-v1 |
| Component type | ADempiere dashboard/chart library with application dictionary migrations |
| Language and target | Java 17 (`sourceCompatibility = 1.17`, `targetCompatibility = 1.17`) |
| Build / runtime | Gradle wrapper 7.3.3; `./gradlew build` |
| Published artifact | `io.github.adempiere:adempiere-dashboard-improvements`; this fork's default publish URL is `https://maven.pkg.github.com/erpcya/adempiere-dashboard-improvements` |
| Version | Tags `1.0.0`–`1.0.8` and `adempiere-3.9.4-1.0.8`; build version from `ADEMPIERE_LIBRARY_VERSION`, fallback `local-1.0.0` |
| License | GNU General Public License version 2 |
| Root package or module | `org.spin.eca50` |
| Declared entityType | `D` in `.ai/repository.yml` and `build.gradle`; migrations create and use entity type `ECA50` |
| Owner | ERP Consultores y Asociados |
| Upstream | https://github.com/adempiere/adempiere-dashboard-improvements |

## 2. Responsibility

This repository owns a reusable ADempiere library that improves dashboard and chart capabilities for the ADempiere new UI. It provides chart data retrieval, chart-related application dictionary windows, and setup that registers the library in an ADempiere instance.

Its main responsibilities are:

- Building chart data from `AD_Chart`/`AD_ChartDatasource` definitions through `org.spin.eca50.controller.ChartBuilder`.
- Supplying chart value data classes: `ChartValue`, `ChartSeriesValue`, `ChartDataValue`.
- Deploying and registering the model validator through `org.spin.eca50.setup.DeployChart`.
- Declaring and applying dictionary migrations under `xml/migration/` for chart definition, chart access, chart parameters, parameter translations, dashboard examples, and additional chart types (`Gauge`, `Portlet`).
- Publishing a reusable Java library artifact.

It does not own:

- The base ADempiere platform, its core chart model, or the core dictionary records it only references or extends.
- UI-only presentation components; this is a library with dictionary support, not a Swing/Web/Mobile UI repository.
- Customer-specific customizations; it is classified as a reusable `Library`, not `PatchCustomer`.

## 3. Architecture

The repository is a small Java library with Gradle build files and ADempiere XML migrations.

```text
.
├── .ai/repository.yml
├── .github/workflows/
│   ├── build.yml
│   ├── knowledge-on-release.yml
│   ├── publish.yml
│   ├── pull-request-review.yml
│   └── release-candidate.yml
├── build.gradle
├── settings.gradle
├── gradle/wrapper/
├── src/main/java/org/spin/eca50/
│   ├── controller/ChartBuilder.java
│   ├── data/
│   │   ├── ChartDataValue.java
│   │   ├── ChartSeriesValue.java
│   │   └── ChartValue.java
│   ├── model/validator/Chart.java
│   ├── setup/DeployChart.java
│   └── util/
│       ├── Changes.java
│       └── ChartQueryDefinition.java
└── xml/migration/
    ├── 10210_Add_Entity_Type_for_ECA50.xml
    ├── 10220_ECA50_Add_Windows_for_Chart_Definition.xml
    ├── 10230_ECA50_Add_Parameter_translation.xml
    ├── 10240_ECA50_Chart_Transformation.xml
    ├── 10250_ECA50_WindowChart_Dictionary_Changes.xml
    ├── 10260_ECA50_Fix_Chart_Transformation.xml
    ├── 10270_ECA50_Gauge_Chart_Type.xml
    └── 10280_ECA50_Portled_Chart_Type.xml
```

Main components:

- `ChartBuilder`: builds and executes chart data-source queries, applies time-series/category aggregation, custom parameters, role-based access SQL, and returns `ChartValue` objects.
- Data model classes: hold chart series and data point structures.
- `Chart` model validator: registers document validation for `I_C_Order` and model change validation for `I_C_OrderLine`.
- `DeployChart`: implements `ISetupDefinition`, creates/retrieves an `AD_ModelValidator` record for `org.spin.eca50.model.validator.Chart`.
- `Changes`: contains a static column-name constant used by the validator.
- Migrations: install entity type `ECA50`, chart-definition windows, tabs, fields, sequences, tables, validations, menu entries, example chart data, and new chart types.

### Pre-existing records this repository modifies

| Record | Table | Columns changed | Effect | Migration |
|---|---|---|---|---|
| 105 | AD_Tab | `IsSingleRow` | Sets the existing tab to single-row behavior | `xml/migration/10220_ECA50_Add_Windows_for_Chart_Definition.xml` |
| 53284 | AD_Table | `AccessLevel` | Sets the existing table access level to `6` | `xml/migration/10220_ECA50_Add_Windows_for_Chart_Definition.xml` |

## 4. Dependencies

| Dependency | Scope | Purpose |
|---|---|---|
| `io.github.adempiere:base:3.9.4` | compile/runtime (`api`) | ADempiere core model, DB, query, context, and chart classes used by this library |
| `lib/*.jar` local file tree | compile/runtime (`api`) | Local jars appended to the API classpath; no `lib/` files are present in the tracked evidence |
| Gradle wrapper 7.3.3 | build | Build system version pinned by the repository |
| Java 17 | build/runtime | Source and target compatibility |

## 5. Consumers

Consumers use this repository as a published Maven artifact:

- Maven/Gradle/SBT builds depending on `io.github.adempiere:adempiere-dashboard-improvements`.
- ADempiere installations applying the migrations under `xml/migration/`.
- ADempiere setup processes invoking `org.spin.eca50.setup.DeployChart`.

Public or integration surfaces that can break consumers:

- Artifact coordinates `io.github.adempiere:adempiere-dashboard-improvements`.
- Public class and method contract `org.spin.eca50.controller.ChartBuilder.getChartData`.
- Public data classes `ChartValue`, `ChartSeriesValue`, `ChartDataValue`.
- Model validator class name `org.spin.eca50.model.validator.Chart`.
- Migration sequence names, IDs, and applied entity type `ECA50`.
- Pre-existing record updates in `AD_Tab` 105 and `AD_Table` 53284, which affect installations that already have those records.

## 6. Allowed changes

- Add new dictionary migrations under `xml/migration/`, following the existing `SeqNo` progression and `ECA50` entity type.
- Add or extend chart data-building logic in `org.spin.eca50.controller.ChartBuilder` while preserving the existing public `getChartData` contract.
- Extend data model classes or add related chart metadata classes under `org.spin.eca50.data`.
- Update build and CI workflows, including publication configuration, as long as published coordinates and artifact behavior remain explicit.
- Add tests or verification steps without changing the public API.
- Add new chart types through ADempiere dictionary records, as was done for `Gauge` and `Portlet`.

## 7. Prohibited changes

- Do not alter the published artifact coordinate `io.github.adempiere:adempiere-dashboard-improvements` without an explicit versioning/coordination decision; downstream Maven/Gradle/SBT snippets reference it.
- Do not break or remove `ChartBuilder.getChartData`, `ChartValue`, `ChartSeriesValue`, or `ChartDataValue` without a compatibility plan; they are the published API surface.
- Do not introduce customer-specific flows or make another project depend on this repository as a `PatchCustomer` customization; it is a reusable library.
- Do not modify the upstream-fork reference branch `master`; work belongs on `erpya` or feature branches. The repository's default line is `erpya`.
- Do not edit pre-existing ADempiere records without a migration that records the change and without updating section 3 of this contract.
- Do not commit actual secrets or credential values; credential names and references belong in CI secret configuration, not in tracked files.

## 8. Architectural rules

1. Java source remains under `org.spin.eca50.*`.
2. Dictionary changes are applied only through `xml/migration/*.xml` migrations; no direct database changes are represented here.
3. Chart data access goes through `org.spin.eca50.controller.ChartBuilder`.
4. The model validator class must remain loadable as `org.spin.eca50.model.validator.Chart`, because `DeployChart` registers that class name.
5. `master` stays untouched as the upstream reference line; default and development line is `erpya`.
6. Any modification to a pre-existing ADempiere record must be reflected in this contract's section 3.

## 9. Risks

| Check | Finding | Impact | Precaution |
|---|---|---|---|
| Identifiers outside the allowed allocation range | None found against the observed ranges in this repository's migrations; no declared allocation range is present in the evidence. Observed inserted IDs fall in 5/6-digit ranges (e.g. 50150, 54864–54867, 54921–54924, 102654–102795). | No allocation violation is demonstrable from the evidence. | Confirm the official allocation range with the platform owner before allocating new dictionary identifiers. |
| Build output or IDE metadata under version control | `.classpath`, `.project`, and `.settings/` are tracked; `.vscode/settings.json` is also tracked. | Environment-specific IDE files create noisy diffs and can misrepresent repository state. | Remove them from version control with `git rm --cached` and ignore them; track only shared settings when justified. |
| Secrets in the tree or recoverable from history | None found. Workflow files and `build.gradle` name secret-bearing variables (`signingKey`, `signingPassword`, `sonatypePassword`, `sonatypeUsername`, `GITHUB_DEPLOY_TOKEN`), but the provided evidence shows references, not values. | No actual secret exposure documented. | Keep all real credential values in GitHub Actions secrets; never add them to tracked files. |
| Absent verification mechanism | No `src/test` directory, no test dependencies, and no test task in evidence; CI build runs `./gradlew build`. | Changes are compile-checked only; chart SQL/aggregation regressions would not be caught automatically. | Add focused tests for query construction and data mapping; use the `erp-ai:verified` release-candidate workflow for human verification. |
| Pre-existing records modified (cross-reference section 3) | Two pre-existing records are modified: `AD_Tab` 105 and `AD_Table` 53284. | Installations that already have these core dictionary records are affected by this library's migrations. | Keep section 3 current and evaluate releases that include those migrations with cross-installation impact in mind. |

| Risk | Impact | Precaution |
|---|---|---|
| Entity type mismatch between repository declaration and migrations | `.ai/repository.yml` and `build.gradle` declare `entityType: D`, while migrations create and use `ECA50`. Consumers filtering dictionary records by entity type may see disagreement. | Owner should confirm the authoritative entity type and reconcile `.ai/repository.yml`, `build.gradle`, and migrations if `D` is not intentional. |
| Publication repository mismatch | README documents Maven Central artifact `io.github.adempiere:adempiere-dashboard-improvements`, while this fork's `build.gradle` defaults to GitHub Packages at `maven.pkg.github.com/erpcya/adempiere-dashboard-improvements`. | Document the fork's real publication repository and ensure releases publish to the intended destination. |
| Time-series data handling incomplete | `ChartBuilder` contains `TODO: Define it with dates` in the time-series branch. | Time-series chart categories are not proven complete; do not rely on that path until implemented and verified. |
| Local jar directory absent from tracked files | `build.gradle` adds `lib/*.jar` to the API classpath, but no `lib/` files are tracked. | Keep the local build environment in sync with any intended local jars; do not assume consumers have them unless published elsewhere. |

## 10. Current state

This is a fork of `https://github.com/adempiere/adempiere-dashboard-improvements`, with default branch `erpya` and an untouched upstream line available as `master`. `.ai/repository.yml` declares it as a `Library` owned by ERP Consultores y Asociados.

The repository builds with Java 17 and Gradle 7.3.3, depends on `io.github.adempiere:base:3.9.4`, and publishes an artifact named `adempiere-dashboard-improvements` under group `io.github.adempiere` by default.

The migrations create dictionary entity type `ECA50`, chart-definition windows and fields, parameter translation support, example chart data, a dashboard allocation sequence, chart transformation fixes, and additional chart types `Gauge` and `Portlet`. The migration list runs from sequence `10210` to `10280`.

The Java layer exposes `ChartBuilder.getChartData` and data model classes for chart values and series. `DeployChart` registers `org.spin.eca50.model.validator.Chart` as a model validator.

Workflows exist for CI build, Maven publishing, knowledge contract evaluation on release, AI pull-request review handling, and release-candidate publishing.

Known gaps:

- No automated tests are present.
- IDE metadata is tracked.
- Entity type declaration (`D`) differs from the migration entity type (`ECA50`).
- Time-series category handling contains an unresolved `TODO`.
- The default publication URL differs from the Maven Central coordinates documented in README.

## 11. UNKNOWN

- The official dictionary identifier allocation range for this repository is not declared in the evidence; verify with the platform owner or allocation records.
- Whether `entityType: D` in `.ai/repository.yml`/`build.gradle` versus `ECA50` in migrations is intentional or a defect has not been established; verify with the owner.
- The exact content and intent of the `lib/` directory referenced by `build.gradle` could not be confirmed; verify the build environment.
- Whether Maven Central or GitHub Packages is the actual publication destination for this fork's releases could not be determined from the evidence; verify release artifacts or repository settings.
- The real existence and semantics of `Changes.COLUMNNAME_ColumAddedToCore` (`ColumAddedToCore`) against the ADempiere base model could not be confirmed; verify with `io.github.adempiere:base`.