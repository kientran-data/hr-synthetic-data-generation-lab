# 16 — Dataset Release, Documentation, Privacy and Change Governance

## Distinguish deliverable states
- **Specification:** Markdown contracts only, no generated dataset implied.
- **Prototype:** runnable small fixture with recorded failing/passing tests.
- **Candidate:** full generated bundle awaiting final QA, privacy and compatibility checks.
- **Release:** immutable, validated, versioned dataset and manifest.
Never call a design artifact an executed release.

## Version dimensions
| Version | Meaning | When it changes |
|---|---|---|
| generator version | Python code behavior | logic/bug fixes |
| scenario version | distributions, calendar, targets and policy | changed sampling assumptions |
| canonical schema version | hidden model shapes/semantics | internal row structure changes |
| source schema version | observable export columns/grains | public CSV/API contract changes |
| dataset release version | approved packaged run | any published file content changes |
| benchmark version | sampled pairs, predicate and split policy | any evaluation task change |

Use conventional semver or a documented alternative, but never infer equality from a reused filename. Changes to columns, date meanings or source IDs can be breaking even if row counts are unchanged.

## Public vs restricted release boundaries
**Public/student release** includes source observations, optional unlabeled match-pair tasks, source schemas, safe row-count summaries, instructions and manifest. **Instructor release** includes canonical entities/events, source-to-truth mapping, true and source-as-of KPI oracles, injected anomaly reasons and labeled benchmarks. They must be packaged from different allowlists, not by zipping the whole output folder.

## Manifest and provenance
Each release needs: scenario/config hash, code commit/version, pinned dependencies, seed scheme, schema versions, run date/cutoff, inputs, row counts, target attainment, validation summary, intended noise, SHA-256 file checksums, semantic replay fingerprint, release audience and known limitations. Avoid claiming timestamped manifests are byte-identical on replay; compare canonical content fingerprints with volatile fields excluded.

## Privacy and ethical generation
- All characters and employers must be fictional. Use `example.com`/`example.org` domains or locally non-routable test identifiers and never send messages.
- No live national IDs, real phone endpoints, biometric/medical data, or inferred sensitive personal traits.
- If age or gender is intentionally included for a course lab, use explicit synthetic assumptions and discourage causal/fairness claims from artificial correlations.
- Confidentiality is a **dataset access rule**, not a security guarantee from naming a directory; enforce separate storage/access controls in actual deployments.

## Licensing and attribution
The public example GitHub HR analytics repository is a reference. Check its license before reusing any original data, SQL, screenshots or dashboard assets. The generator should make newly simulated records rather than repackage third-party HR data. Document the reference and source-credit decisions in README/release notes.

## Release checklist
- [ ] Config feasibility and root-seed registration completed.
- [ ] Full canonical and operational lifecycle validation completed.
- [ ] Source formats import into the declared SQL environment.
- [ ] Distribution and noise diagnostics included.
- [ ] No truth IDs or restricted labels in public data, metadata, notebooks or filenames.
- [ ] Instructor mappings and benchmark labels stored separately.
- [ ] Manifest contains all file hashes and version identifiers.
- [ ] Same-seed replay demonstrates semantic equivalence.
- [ ] Known limitations, partial-year cutoff and fictional policy assumptions documented.
- [ ] Safe audience packages constructed from allowlists and scanned before publication.

## Change request template
Document request ID, why the change is needed, affected stages/schemas, compatibility impact, before/after scenario outcomes, new tests, migration and release version. For source schema changes, include a data dictionary diff and a working import example. For business-rule changes, rerun KPI/temporal calibration and bump scenario or generator versions as appropriate.

## Future expansion
The domain-neutral parts—configuration, seeded event generation, time-bounded relationships, source projections, observation defects, validation and manifest—can later serve other fictional domains. HR-specific simulators and policies remain isolated plugins. This is an architectural direction, not a claim of currently implemented cross-domain support.
