# Netplan Field Data Flow Workflow (V2)

Use this checklist to trace any Netplan field from YAML input to backend output with high confidence.

## 1. Define the target field precisely

- Canonical path: `network.<type>.<id>.<field>` or global/type-level equivalent.
- Field context: `global`, `type-level`, or `netdef-level`.
- Expected YAML node kind to investigate: scalar, sequence, mapping.

## 2. Build a field variant matrix before deep tracing

Test all relevant forms:

- Omitted key.
- Valid explicit value(s).
- Invalid value(s).
- Wrong YAML type.
- Empty value.
- `null`.
- Case variants (if token-like field).
- Multi-file overrides (earlier/later layer).
- Patch/nullification variants (if using `set` or patch path).

## 3. Locate parser registration and immediate handler path

- Find handler table entry in `src/parse.c`.
- Record expected YAML node type from table metadata.
- Follow handler function to capture parse-time behavior.

## 4. Determine required vs optional semantics

- Check parser: does omission itself fail?
- Check grammar/backend validation: is field required conditionally?
- Mark requirement scope:
  - always required,
  - required under conditions,
  - optional.

## 5. Determine defaults at all stages

- Parse-time stored default from reset/init (`src/types.c` etc.).
- Merge-time implicit behavior when omitted.
- Final/effective default applied later (`finish_iterator`, backend generation, writer logic).
- Distinguish clearly:
  - stored value,
  - effective behavior.

## 6. Capture parse-time validation rules exactly

- Accepted tokens/ranges/patterns.
- Case sensitivity (`strcmp` or `g_strcmp0` vs `g_ascii_strcasecmp`).
- Error messages produced.
- Failure behavior in:
  - normal mode,
  - `IGNORE_ERRORS` mode.

## 7. Map internal representation and normalization

- Field type in object (`src/abi.h`, `src/types-internal.h`).
- Whether value is stored as:
  - scalar,
  - enum/tristate,
  - split booleans,
  - arrays/maps,
  - link pointers.
- Any normalization/transformation during parsing.

## 8. Document merge rules for this field specifically

- Scalar replace vs map merge vs list append/replace/dedupe.
- Omitted-key behavior across layers.
- Empty mapping behavior.
- Nullification/deletion semantics in patch path (`null_fields`, `null_overrides`).
- Basename shadowing and file order impacts from hierarchy load.

## 9. Capture cross-field dependencies and constraints

- Preconditions and companion fields.
- Mutual exclusions.
- Backend-specific support/unsupported combinations.
- Side effects on other fields or flags.

## 10. Capture final validations and late mutations

- Definition-level grammar checks.
- Backend-rule checks and SR-IOV/route/other final checks.
- Any final pass that mutates effective values.

## 11. Trace rendering to backend outputs

- Which backend(s) consume field.
- Which writer file(s) and function(s) use it.
- Output artifact mapping:
  - file(s) created/updated,
  - section/key emitted,
  - conditions for emission/suppression.
- Unsupported backend behavior and errors.

## 12. Trace write-back/round-trip behavior

- YAML dump/update behavior for this field.
- Whether serialized value is preserved, normalized, or omitted as default.
- Per-file provenance implications (`sources`, `global_renderer`, filepath ownership).

## 13. Verify with tests and fill gaps

- Positive behavior tests.
- Negative/validation tests.
- Merge/layer tests.
- Ignore-errors behavior tests.
- If missing, note test gap explicitly.

## 14. Produce a structured field report

Include this exact structure:

1. Field.
2. Required/optional (with conditions).
3. Defaults: stored vs effective.
4. Parse-time validation and error behavior.
5. Internal representation.
6. Merge rules.
7. Final validation/late defaults.
8. Backend output mapping.
9. Round-trip/provenance notes.
10. Evidence list with `path:line`.
11. Confidence tag per claim:
   - `code-proven`,
   - `test-proven`,
   - `inferred`.

## 15. End with a minimal reproducible example

- Small YAML snippet(s).
- Expected parse outcome.
- Expected backend output fragment(s).
- Optional matrix row IDs linking back to Step 2.

## Recommended evidence files

- Parser and mapping: `src/parse.c`
- Defaults/reset: `src/types.c`
- Internal types: `src/abi.h`, `src/types-internal.h`
- Validation: `src/validation.c`
- Hierarchy/merge order: `src/util.c`
- Backend renderers: `src/networkd.c`, `src/nm.c`, `src/openvswitch.c`, `src/gen-networkd.c`
- YAML emission/update behavior: `src/netplan.c`
- Tests: `tests/generator/*.py`, `tests/test_libnetplan.py`, `tests/ctests/*.c`
