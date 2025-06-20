# Bun Lockfile Test

This repository is designed to test and demonstrate a specific behavior in Bun's package installation process, particularly when using the `--frozen-lockfile` flag with filtered installations.

## Test Scenario

The test demonstrates an inconsistency in how Bun resolves dependencies when using different installation commands.

### Test Commands

1. **Full installation (works as expected):**

   ```bash
   rm -rf node_modules
   bun install --frozen-lockfile
   grep '"version"' node_modules/mime/package.json | sed 's/.*"version": "\(.*\)".*/\1/'
   ```

   **Result:** Returns `1.6.0` (matches bun.lock)

   ```bash
   grep '"version"' node_modules/postcss-url/node_modules/mime/package.json | sed 's/.*"version": "\(.*\)".*/\1/'
   ```

   **Result:** Returns `2.5.2` (matches bun.lock)

2. **Filtered installation with frozen lockfile (unexpected behavior):**
   ```bash
   rm -rf node_modules
   bun install --frozen-lockfile --filter a-widget
   grep '"version"' node_modules/mime/package.json | sed 's/.*"version": "\(.*\)".*/\1/'
   ```
   **Result:** Returns `2.5.2` (different from bun.lock)

## Expected Behavior

When using `--frozen-lockfile --filter a-widget`, we expect `mime@2.5.2` to be installed as a dependency of `postcss-url` in the path:

```
node_modules/postcss-url/node_modules/mime
```

This should match what's specified in the `bun.lock` file.

## Issue Description

The test reveals that Bun's filtered installation with frozen lockfile doesn't properly respect the nested dependency structure defined in the lockfile. Instead of installing `mime@2.5.2` as a nested dependency under `postcss-url`, it installs a different version (`1.6.0`) at the root level.

This behavior suggests a potential issue with how Bun handles dependency resolution during filtered installations with frozen lockfiles.

## Project Structure

```
bun-lock-test/
├── apps/
│   └── a-ui/
│       └── package.json
├── libs/
│   └── a-widget/
│       └── package.json
├── bun.lock
└── package.json
```

## Reproduction Steps

1. Clone this repository
2. Run the test commands above
3. Compare the results with the expected behavior described in this README

This test case can be used to:

- Verify Bun's dependency resolution behavior
- Report potential bugs in Bun's filtered installation feature
- Understand the differences between full and filtered installations with frozen lockfiles
