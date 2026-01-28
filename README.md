# github-workflows

A collection of workflows for my GitHub repos.

## auto-tag-rust.yml

Creates and pushes a Git tag based on the version in Cargo.toml whenever changes land on master. It prevents duplicate tags by checking if the version tag already exists.

## validation-rust.yml

Runs Rust CI on pushes and PRs for develop, master, and feature branches. It checks formatting, runs Clippy, builds, tests, and publishes coverage to Codecov (non-blocking for formatting, lint, and coverage).

## release-rust.yml

Builds and tests on master updates to Cargo.toml, then tags the release based on Cargo.toml and merges master into develop after a successful release tag (skips if the tag already exists).

## validate-release-rust.yml

Validates that a PR targeting master updates the Cargo.toml version by comparing the PR version to master’s version.

## sync-develop.yml

Automatically merges master into develop on every push to master to keep develop up to date.

## dependabot-rust.yaml

Configures Dependabot to check Cargo and rust-toolchain updates weekly while ignoring semver patch updates.
