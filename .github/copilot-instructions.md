# Copilot Instructions For Netplan Repository

When analyzing a Netplan configuration field data flow from input YAML to rendered backend output (when the user asks to trace a Netplan field), follow the workflow in:

- `.github/netplan-field-dataflow.md`

Expectations:

- Use source-backed conclusions with precise file references.
- Distinguish stored defaults from effective/runtime defaults.
- Cover normal parse mode and `IGNORE_ERRORS` behavior where relevant.
- Include merge/layering behavior across files and patch/nullification behavior when applicable.
- Identify backend rendering effects (`networkd`, `NetworkManager`, `Open vSwitch`) and unsupported combinations.
- Label conclusions by confidence: `code-proven`, `test-proven`, or `inferred`.

Output format for field investigations should include:

1. Field path and scope
2. Required vs optional
3. Defaults (stored vs effective)
4. Parse-time validations
5. Internal representation
6. Merge/layering rules
7. Final validation or late defaults
8. Backend output mapping
9. Evidence references
10. Minimal reproducible example
