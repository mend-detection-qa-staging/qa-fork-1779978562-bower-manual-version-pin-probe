# bower-manual-version-pin-probe

## Patterns covered

This probe bundles two complementary Bower detection patterns:

1. **manual-version-pin** -- Dependencies are declared with bare exact versions (`"3.7.1"`) rather than semver operators (`"^3.7.1"` or `"~3.7.1"`). Mend must report the exact pinned version, not a range. The `source_detail.constraint` field in Mend output should reflect a bare version string, not a range expression.

2. **bower-components-committed** -- The `bower_components/` directory is committed into the repository (no `.gitignore` exclusion). Each `bower_components/<pkg>/.bower.json` contains the resolution metadata written by `bower install`: `_release`, `_resolution.type`, `_resolution.tag`, `_resolution.commit`, `_target`, and `_source`. This is the closest artifact to a lockfile that Bower produces and Mend's UA fallback can read these fields directly without re-resolving from the registry.

## Why these patterns are bundled

Both patterns improve detection fidelity together: exact pins in `bower.json` guarantee there is no version ambiguity, and committing `bower_components/` provides Mend with the resolved-state metadata. Together they give the maximum possible signal for reliable detection.

## Dependencies

| Package | Declared constraint | Expected resolved version | Direct |
|---|---|---|---|
| jquery | `3.7.1` (exact pin) | 3.7.1 | yes |
| backbone | `1.6.1` (exact pin) | 1.6.1 | yes |
| moment | `2.30.1` (exact pin) | 2.30.1 | yes |
| underscore | (resolved by backbone) | 1.13.8 | no |

Total: 4 packages. Direct: 3. Transitive: 1.

## What Mend should detect

- Exact resolved versions matching the pins: `jquery@3.7.1`, `backbone@1.6.1`, `moment@2.30.1`.
- `underscore` as a transitive dependency pulled in by `backbone` (backbone's `bower.json` declares `"underscore": ">=1.8.3"`).
- All four packages attributed to source type `registry`.
- Version constraints in detection output should be bare version strings (`3.7.1`, `1.6.1`, `2.30.1`), not range expressions.
- From `.bower.json` files: `_release` matches `version`, `_resolution.type` is `"version"`, `_target` is the bare version string (exact pin), `_source` points to the GitHub git endpoint.

## Key detection failure modes exercised

- Mend reports a semver range instead of the exact pinned version (treats `3.7.1` as `^3.7.1`).
- `underscore` is missing because Mend did not traverse backbone's transitive dependency.
- Mend ignores `.bower.json` metadata even when it is committed and prefers manifest-only detection.
- `_target` field in `.bower.json` shows exact pin but Mend normalises it to a range.

## Probe metadata

```json
{
  "probe_name": "bower-manual-version-pin-probe",
  "patterns": ["manual-version-pin", "bower-components-committed"],
  "pm": "bower",
  "generated": "2026-05-04",
  "target": "remote",
  "remote_url": "https://github.com/mend-detection-qa/bower-manual-version-pin-probe"
}
```