# Contributing

Bug reports, benchmark results from other hardware, and PRs are all welcome.

## Building and testing

```bash
cargo build --release        # needs bubblewrap (`bwrap`) on the host
cargo test                   # Rust unit tests (pure logic)
pip install pytest
systemd-run --user --scope -q -p OOMPolicy=continue -- python3 -m pytest -v
```

The pytest suite is the real test: it drives `target/release/runbox` through
its CLI/JSON contract, adversarial cases included (fork bombs, memory bombs,
output floods). The `systemd-run` wrapper gives it a delegated cgroup scope
and keeps systemd from killing the session when the OOM tests fire; plain
`pytest -v` also works, with cgroup-dependent asserts skipping.

Full-fidelity runs need what the README's "Host requirements" section
describes: `kernel.perf_event_paranoid` ≤ 2 and a real PMU. Without them the
instruction-count asserts skip — that's expected (GitHub runners have no
PMU), but please run the full suite on real hardware for changes touching
measurement.

## CI gates

PRs must pass `cargo fmt --check`, `cargo clippy --all-targets -- -D
warnings`, `cargo test`, and the pytest suite (as far as runner hardware
allows). Run them locally before pushing.

## Ground rules

- **The JSON contract is stable.** Never rename, remove, or retype a field
  in [docs/CONTRACT.md](docs/CONTRACT.md); add fields instead, and document
  them there in the same PR.
- **Measurement claims need measurements.** Anything affecting variance or
  load-invariance should come with numbers from
  [`bench/measure.py`](bench/measure.py) (methodology in
  [docs/BENCHMARK.md](docs/BENCHMARK.md)).
- Keep the child-side code between `fork` and `exec` async-signal-safe —
  no allocation, no locks.
- Dependencies are deliberately minimal (currently just `libc`); adding one
  needs a strong reason.

## Security issues

Not here — see [SECURITY.md](SECURITY.md) for private reporting.
