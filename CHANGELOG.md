# Changelog

All notable changes to this project are documented here. Format loosely
follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Fixed

- `mypy` failed on Windows: the system collector imported the POSIX-only
  `resource` module inside a `try/except ImportError`, which the type checker
  cannot see through. It now checks `sys.platform` first, which `mypy`
  understands, and CI runs the whole pipeline on Windows too (Python 3.9 and
  3.14) alongside Linux (3.9 through 3.14).
- The experiment safety tests spawned a hard-coded `python3`, which is not on
  `PATH` on Windows, so three of them failed there. They now use
  `sys.executable`, the interpreter already running the suite.

## [0.1.0] - release candidate

Initial MVP. See `docs/ARCHITECTURE.md#roadmap` for what's deliberately not
in this release yet.

### Added

- Versioned snapshot schema (`schema_version` 1.0) with deterministic JSON
  serialization.
- Collectors: system (OS/kernel/arch/CPU/memory/filesystem/limits), runtime
  (Python fully, Node/Java/Go/.NET/Ruby/PHP probed via `--version`),
  dependencies (Python via `importlib.metadata`), network (local
  IPv4/IPv6 loopback + resolver capability only), containers (Docker/K8s
  detection), git (commit/branch/dirty state).
- Redaction policy (`ghoststate/redaction.py`): environment variables
  collected as presence/absence only, never values.
- Deterministic diff engine with `UNCHANGED`/`CHANGED`/`ADDED`/`REMOVED`
  classification and a documented `LOW`/`MEDIUM`/`HIGH`/`CRITICAL`
  relevance rule table.
- Evidence engine: hypothesis construction with a documented, bounded
  (<100%) heuristic confidence score, and strict epistemic language that
  only reaches "confirmed by experiment" after a real experiment runs.
- Investigation engine: ranks hypotheses, and honestly reports
  "insufficient evidence" when no property reaches MEDIUM relevance or the
  two snapshots are identical.
- Experiment engine: proposes a minimal experiment per hypothesis; only
  executes a caller-supplied command with explicit `approved=True`
  (mapped to CLI `--yes`), never mutates the real process environment.
- CLI: `init`, `capture`, `compare`, `investigate`, `experiment`,
  `explain`, `export`, `doctor`, `demo` — with `--json` output on the
  commands where it matters.
- Reproducible end-to-end demo (`ghoststate demo`) using real subprocess
  runs, a real temporary git repository, and real captured snapshots — no
  fixture data.
- Test suite: 70 tests across unit, integration, and security categories,
  including path-traversal, secret-leakage, unsafe-subprocess, and
  malformed-input regression tests.
- Documentation: README, ARCHITECTURE, PRIOR_ART, THREAT_MODEL, PRIVACY,
  SECURITY, CONTRIBUTING, CODE_OF_CONDUCT.

### Known limitations (see README "Status")

- No dashboard/web UI yet (CLI + JSON only).
- No LLM-assisted narrative (the core works, and is tested, fully without one).
- Dependency collection is Python-only; other ecosystems report
  `not_supported_in_v0.1` rather than guessing.
- The experiment engine re-runs a supplied command with env overrides; it
  does not mutate live kernel/network/container state.
