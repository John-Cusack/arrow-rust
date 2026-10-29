arrow-rust — Rust-acceleration lane for arrow-py/arrow
=====================================================

This repository was created for the rustsmith `rs-port-arrow` lane
(2026-10-28). It contains no ported code: the lane measured the
tier-1 pure-Python memoization first and declined the Rust rewrite on
the numbers (honest no_gain).

Lane result
------------

- Headroom probe (frozen WORKLOAD.md, representative
  parse→shift→format→humanize pipeline over 2000-timestamp batches):
  score **0.72 rewrite**, pure 85%, top frame
  `DateTimeParser._generate_pattern_re` per-parse regex rebuild.
- Tier-1 memoization (pure Python): fmt-parse 44.6 → 7.4 ms
  (**6.03x**); representative 74.4 → 37.4 ms (**1.99x**). A
  lane-tested candidate patch (class-level `(locale, fmt)` memo + one
  parser per locale in the factory) gives 1.91x end-to-end through
  `arrow.get` with a 20-case equivalence corpus byte-identical.
- Rust rewrite declined: against the memoized baseline only
  parse 7.1 + format 9.3 = 16.4 of 37.4 ms is kernel-addressable, so
  the infinite-speed ceiling is **1.78x < 2x** kill bar (realistic
  ≈1.4–1.5x); the rest is `Arrow.shift`/object-model dispatch plus
  locale-aware formatting — dispatch-dominated.
- The pure-Python memoization is written up as an upstream-ready patch
  note (no upstream PR opened per lane rule).

Evidence: rustsmith lane `rs-port-arrow-01` (ADRs + bench rows in the
rustsmith evidence PR); upstream patch note with the lane record.

Attribution
-----------

Any future port here derives its behavior from
https://github.com/arrow-py/arrow (Apache-2.0); see NOTICE.
