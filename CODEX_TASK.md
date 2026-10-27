# Codex task: repair and modernize CXBin Converter

Repository: J-W-Smith/CXBin-Converter. Work on this branch; keep master untouched, produce a reviewable pull request, and do not merge automatically.

## Report
The user tried a Creality Cloud Rumi cosplay shoulder-armor `.cxbin` model and the GUI displayed an error. A previous independent analysis reportedly recovered geometry (137,797 vertices / 275,636 triangles) and exported binary STL; **verify these claims when provided the file**. The model is not committed or attached. Ask for the exact GUI error text and a local/private fixture as necessary, without publishing copyrighted model contents.

## Priorities
1. **Reproduce and identify root cause**. Inspect `src-tauri/src/cxbin/reader.rs`, `mesh.rs`, `commands.rs`, `cli.rs`, export modules, UI error/preview paths. Check the official Creality CXBin format and the version/head-code interpretation; do not assume file corruption or parser bug without reproduction.
2. **Parser hardening**. Validate signed lengths/counts before conversion, bound allocations and zlib decompression, use checked math on compressed/decompressed sizes and texture RGBA dimensions, reject truncated blocks, malformed faces/UV indices, invalid zlib, huge declared counts, and prevent OOM or panics on untrusted input. Distinguish unsupported versions from corrupted input; don't silently ignore potentially important sections. Preserve support for legacy and modern valid files.
3. **Diagnostics**. Add actionable parse/export/IO error messages in GUI and CLI (stage, file, cause), with a safe diagnostics option for support that does not disclose private geometry. Test error display, output directory failure, and preview failure.
4. **Export correctness**. Validate STL, 3MF and OBJ first; check triangle indices, normals, optional materials and textures, and format handling. Do not break existing supported formats.
5. **Maintenance**. Review Rust/JS dependency updates and advisories without unnecessary major upgrades; ensure React/Tauri code builds, refresh fork's README installation command (currently points to HellBz), improve CI build/tests/release workflow.
6. **Tests**. Create small synthetic redistributable CXBin test fixtures for valid legacy/new-layout files and malformed inputs; add Rust tests and UI checks where practical. Do NOT check the user's proprietary source model or converted STL into git.

## Definition of done
- Supported valid fixtures reliably load, preview, and convert to STL.
- Invalid/corrupt inputs fail promptly with clear errors and without panic or excessive allocation.
- GUI distinguishes parsing, exporting, filesystem and unexpected failures.
- `cargo test`, `cargo clippy`, frontend lint/typechecks, and build checks pass where supported; list unrun checks.
- PR includes a focused change log, exact reproduction/verification steps, tests, and any remaining unknowns.
- Do not claim the specific user's error is resolved until the original file is tested or matching failure is reproduced.

## Starting point
`src-tauri/src/cxbin/reader.rs` currently casts raw i32 counts to usize before validation, multiplies lengths without checked arithmetic, decodes texture dimensions via multiplication, and decompresses buffers to unbounded `Vec`. Begin with safe resource caps and precise errors, plus tests, and then examine parser format compatibility. Preserve texture/material layout accuracy; consult official reference, not guesswork.
