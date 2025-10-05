# Quick Reference: Bzlmod Setup

This repository now uses Bazel 7.6.1 with bzlmod enabled.

## File Structure

```
├── MODULE.bazel          # Main module definition (dependencies from BCR)
├── WORKSPACE.bzlmod      # Non-BCR dependencies and custom setup
├── WORKSPACE             # Deprecated, kept for reference
├── .bazelrc              # Bzlmod enabled
├── .bazelversion         # 7.6.1
└── .bazelignore          # Excludes xla/mlir_hlo sub-workspace
```

## Key Dependencies in MODULE.bazel

### From Bazel Central Registry (BCR)
- **Core rules**: bazel_skylib, platforms, rules_license, rules_pkg, rules_proto
- **Language**: rules_cc, rules_python, rules_go, rules_shell
- **Platforms**: rules_apple, rules_swift, rules_android
- **C++ libs**: protobuf, abseil-cpp, grpc, googletest, benchmark, re2, zlib

### Custom Patches Applied
Using `single_version_override`:
- `rules_cc`: protobuf compatibility
- `rules_python`: pip, freethreading, version support
- `protobuf`: custom XLA patches
- `abseil-cpp`: btree, dll, endian, rules_cc, check_op
- `grpc`: custom patches

### Python Setup
- Python 3.11 (default) and 3.12 toolchains
- Separate pip hubs: `@pypi` (3.11), `@pypi_312` (3.12)
- Requirements: `//:requirements_lock_3_11.txt`, `//:requirements_lock_3_12.txt`

## Key Dependencies in WORKSPACE.bzlmod

### Not Yet in BCR
- **rules_ml_toolchain**: Custom ML toolchains for CPU/GPU
- **rules_closure**: Legacy rules (2019 version)

### Custom Initialization
- **Workspace files**: workspace0.bzl through workspace4.bzl
- **Third-party**: ~50 custom dependencies from third_party/
- **Hardware**: CUDA, ROCm, NCCL, NVSHMEM, TensorRT configs
- **Hermetic Python**: Custom Python setup with pip integration

## Common Build Commands

```bash
# Standard build
bazel build //...

# Build with specific config
bazel build --config=cuda //...

# Query dependency graph
bazel mod graph

# Check module information
bazel mod show_repo <module_name>

# Build without bzlmod (emergency fallback)
bazel build --noenable_bzlmod //...
```

## Troubleshooting

### Module Version Conflicts
```bash
# Check dependency tree
bazel mod graph --output text

# Force specific version (in MODULE.bazel)
single_version_override(
    module_name = "...",
    version = "...",
)
```

### Missing Patches
If a patch fails to apply after version update:
1. Check patch file in `third_party/*/`
2. Update patch or remove if no longer needed
3. Test build after changes

### Repo Mapping Issues
- Most handled automatically by bzlmod
- Use `repo_name` in `bazel_dep` for explicit mappings
- Example: `repo_name = "com_google_protobuf"` maps to `@com_google_protobuf`

## Migration Notes

1. **WORKSPACE is deprecated**: Use MODULE.bazel and WORKSPACE.bzlmod
2. **Sub-workspaces ignored**: xla/mlir_hlo is in .bazelignore
3. **Hermetic toolchains**: ML toolchains registered in WORKSPACE.bzlmod
4. **Custom dependencies**: Most third_party/* stay in workspace files

## Version Requirements

- **Bazel**: >= 7.6.1 (enforced in MODULE.bazel)
- **Python**: 3.11 (default), 3.12 (available)
- **OS**: Linux, macOS, Windows (via WSL recommended)

## Documentation

- Full guide: [BZLMOD_MIGRATION.md](BZLMOD_MIGRATION.md)
- Bazel bzlmod: https://bazel.build/external/overview#bzlmod
- BCR: https://registry.bazel.build/

## Support

For issues:
1. Check [BZLMOD_MIGRATION.md](BZLMOD_MIGRATION.md)
2. Review Bazel logs: `bazel build //... --verbose_failures`
3. Contact maintainers: maintainers at openxla.org

## Testing Status

✅ **Tested with Bazel 7.6.1** - Basic configuration working
✅ **Simple builds pass** - `//:license` target builds successfully
⚠️ **Protobuf patch disabled** - Needs update for version 31.x
⚠️ **Full testing pending** - XLA components, Python, CUDA configs

See [BZLMOD_MIGRATION.md](BZLMOD_MIGRATION.md) for detailed testing status.
