  Set up fixture-driven end-to-end CLI tests for this project, following this exact model.

  Goal:
  - Add integration tests under `tests/` using fixtures under `tests/fixtures/synthetic/<case>/`.
  - Each case must contain:
    - `input/` (initial files)
    - `scenario.toml` (commands + assertions)
    - `expected/` (final files after all commands)

  Implement:
  1. Create `tests/cli_synthetic.rs` with:
     - Case discovery from fixture root, sorted for deterministic order.
     - TempDir per case.
     - Copy `input/` into temp dir.
     - Parse scenario TOML with `[[command]]` entries:
       - `args: [String]`
       - `expect_exit` default `0`
       - `stdout_contains`, `stdout_not_contains`, `stderr_contains`, `stderr_not_contains`
     - Run CLI via `std::process::Command::new(env!("CARGO_BIN_EXE_<binary_name>"))`.
     - `current_dir(temp_dir)`.
     - Environment isolation: remove env vars that can change behavior (at least project-specific root/mode/editor vars; e.g. `DJOUR_ROOT`, `DJOUR_MODE`, `EDITOR`, `VISUAL`).
     - Assert command outputs and exit codes.
     - Compare full final tree with `expected/` (missing/extra files + byte/text comparison; normalize `\r\n` vs `\n` for text files).

  2. Add a tiny shared helper in `tests/common/mod.rs` for env-sanitized command execution if useful.

  3. Add at least one sample fixture case.

  Safety/determinism:
  - Do not rely on host editor, cwd, or user env.
  - If a test needs editor behavior, set explicit no-op editor command in that test.
  - Keep outputs stable across OS (newline normalization).

  Validation:
  - Run in order: `cargo build`, `cargo test`, `cargo fmt`, `cargo clippy --all-targets --all-features`.

  Workflow rules:
  - If behavior changes, add/adjust unit tests.
  - If behavior changes file outputs, add/adjust synthetic fixture cases.
  - If behavior only changes stdout/stderr, assert it in scenario and run command once to verify.
  - If CLI args/options change, update README.