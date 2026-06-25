# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a collection of GitHub Actions workflows. The workflows manage CI/CD, versioning, tagging, dependency updates, and documentation deployment.

## Workflow Architecture

The workflows implement a branching strategy with three main flows:

### Development and Testing

- **validation-rust.yml**: Main CI pipeline for develop branch pushes and master PRs
  - Runs: formatting checks, Clippy linting, builds, tests, and coverage
  - Format/lint/coverage are non-blocking to allow PRs to merge while fixing style issues
  - Uploads coverage to Codecov

- **validate-cpp.yml**: C++ CI pipeline for develop and master branches (pushes and PRs)
  - Runs: clang-tidy linting (PRs only), CMake configuration, builds, tests, and coverage
  - Generates coverage with lcov
  - Uploads coverage to Codecov

### Versioning and Release

- **validate-release-rust.yml**: PR gate for master branch
  - Ensures Cargo.toml version is bumped in every PR to master
  - Fails the check if version hasn't changed

- **auto-tag-rust.yml** and **release-rust.yml**: Version tagging on master
  - Extract version from Cargo.toml using `cargo get package.version`
  - Check if git tag already exists to prevent duplicates
  - Create and push git tag matching the version
  - (release-rust.yml also runs build/test before tagging)

### Branch Synchronization

- **sync-develop.yml**: Keeps develop in sync with master
  - Automatically merges master into develop after every master push
  - Uses no-ff merges to preserve commit history

- **release-rust.yml**: Also merges master into develop after a new release tag

### Documentation

- **deploy-doxygen-docs.yml**: Builds and deploys Doxygen documentation
  - Triggers on master pushes or manual workflow dispatch
  - Configures CMake with documentation build enabled
  - Builds doxygen target with Graphviz support
  - Deploys HTML output to GitHub Pages

### Dependency Management

- **dependabot-rust.yaml**: Weekly checks for updates
  - Ignores semver patch updates for both Cargo and rust-toolchain
  - Targets only minor and major version updates

## Key Implementation Details

- All Rust workflows use `actions-rust-lang/setup-rust-toolchain@v1` for Rust setup
- `cargo install cargo-get` is used to extract version from Cargo.toml files
- Git operations (tagging, merging) are done via the `github-actions[bot]` user
- Full fetch history (fetch-depth: 0) is required for tag checking
- Environment variable `CARGO_TERM_COLOR: always` is set for colored output
- Tarpaulin 0.34.1 is used for code coverage generation

## Common Git Workflow

1. Create a feature branch from develop
2. Make changes and push
3. Verify CI passes in validation-rust.yml
4. Create PR to master with bumped Cargo.toml version
5. validate-release-rust.yml ensures version bump
6. After merge: auto-tag-rust.yml creates the release tag
7. sync-develop.yml merges master back into develop

## Testing Workflows Locally

Test workflow syntax with:
```bash
# Install act: https://github.com/nektos/act
act --list  # Show available workflows
act push -b  # Simulate a push event on current branch
act pull_request  # Simulate a PR event
```

Note: `cargo get` must be installed in your local environment to test version extraction.
