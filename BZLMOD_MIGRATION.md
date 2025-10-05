# Bzlmod Migration Guide

This repository has been migrated to use Bzlmod (Bazel's new external dependency management system) with Bazel 7.6.1.

## What Changed

### Files Added
- `MODULE.bazel` - New module definition file that declares dependencies using Bzlmod
- `WORKSPACE.bzlmod` - Contains dependencies not yet available in the Bazel Central Registry (BCR)
- `.bazelignore` - Excludes sub-workspaces that conflict with Bzlmod

### Files Modified
- `.bazelversion` - Updated from 7.4.1 to 7.6.1
- `.bazelrc` - Enabled Bzlmod with `--enable_bzlmod`
- `WORKSPACE` - Marked as deprecated, now only used for backwards compatibility

## Key Features

### MODULE.bazel Structure
The MODULE.bazel file declares:
- **Core Bazel rules**: bazel_skylib, platforms, rules_license, rules_pkg, rules_proto
- **Language rules**: rules_cc, rules_python, rules_go
- **Platform rules**: rules_apple, rules_swift, rules_android
- **Dependencies with patches**: Uses `single_version_override` for custom patches on:
  - rules_cc (protobuf compatibility)
  - rules_python (pip version, freethreading, versions)
  - protobuf (custom patches)
  - abseil-cpp (btree, build_dll, endian, rules_cc, check_op patches)
  - grpc (custom patches)
- **Python toolchains**: Configured for Python 3.11 (default) and 3.12
- **Pip dependencies**: Separate hubs for each Python version

### WORKSPACE.bzlmod Structure
Contains dependencies that cannot be migrated to MODULE.bazel yet:
- **rules_ml_toolchain**: Custom ML toolchain not in BCR
- **rules_closure**: Legacy rules (2019 version)
- **workspace0-4.bzl**: Existing workspace initialization files
- **Python setup**: Hermetic Python configuration
- **Hardware support**: CUDA, NCCL, NVSHMEM configurations

## Building with Bzlmod

### Basic Build
```bash
bazel build //...
```

### Common Issues

#### 1. Module not found in BCR
If you see errors about modules not being found in the Bazel Central Registry:
- Check if the module exists at https://registry.bazel.build/
- If not available, add it to `WORKSPACE.bzlmod` as an `http_archive`

#### 2. Version conflicts
If you see version conflict errors:
- Check the versions in `MODULE.bazel`
- Use `bazel mod graph` to visualize the dependency graph
- Use `single_version_override` or `multiple_version_override` to resolve conflicts

#### 3. Repo mapping issues
For old-style `@repo//target` references:
- Most are automatically mapped in Bzlmod
- Use `repo_name` parameter in `bazel_dep` for explicit mappings
- Example: `bazel_dep(name = "protobuf", repo_name = "com_google_protobuf")`

### Useful Commands

```bash
# Show dependency graph
bazel mod graph

# Show outdated dependencies  
bazel mod tidy

# Query modules
bazel query --output=build @<module_name>//...

# Disable bzlmod temporarily (not recommended)
bazel build --noenable_bzlmod //...
```

## Migration Notes

### Custom Patches
Many dependencies require custom patches for XLA compatibility:
- These are applied using `single_version_override` in MODULE.bazel
- Patches are located in `third_party/` subdirectories
- When updating versions, verify patches still apply

### Python Dependencies
- Python dependencies are managed via rules_python pip extension
- Requirements files: `requirements_lock_3_11.txt` and `requirements_lock_3_12.txt`
- Each Python version has its own pip hub repository

### CUDA/ROCm/TPU Support
Hardware-specific dependencies remain in WORKSPACE.bzlmod:
- CUDA: `@local_config_cuda`
- NCCL: `@local_config_nccl`
- NVSHMEM: Configured via rules_ml_toolchain
- TensorRT: `@local_config_tensorrt`

### Third-Party Dependencies
Many XLA-specific third-party dependencies are loaded from:
- `third_party/*/workspace.bzl` files
- These are called from `workspace2.bzl`
- Not migrated to MODULE.bazel as they are custom/vendored

## Compatibility

### Minimum Requirements
- Bazel 7.6.1 or higher
- All platforms supported (Linux, macOS, Windows)

### Backwards Compatibility
The old WORKSPACE file is kept for reference but is deprecated:
- When bzlmod is enabled, WORKSPACE is only used as a fallback
- WORKSPACE.bzlmod takes precedence for non-module dependencies
- Consider using MODULE.bazel going forward

## Contributing

When adding new dependencies:
1. Check if available in BCR at https://registry.bazel.build/
2. If in BCR, add to MODULE.bazel using `bazel_dep`
3. If not in BCR, add to WORKSPACE.bzlmod using `http_archive`
4. Document any custom patches or configurations

## Resources

- [Bzlmod documentation](https://bazel.build/external/overview#bzlmod)
- [Bazel Central Registry](https://registry.bazel.build/)
- [Migration guide](https://bazel.build/external/migration)
- [Module extensions](https://bazel.build/external/extension)
