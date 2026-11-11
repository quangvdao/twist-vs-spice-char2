Benchmarks for issue #1 on an Apple M4 Max (12 performance and 4 efficiency
cores, macOS), shared with other jobs. Every binary was built with
RUSTFLAGS='-C target-cpu=native'. The baseline binary is ram-compare built at
632855e (upstream master).

final/
    The current results, from ram-compare built at 4e2ae07
    (environment.git_commit in each JSON). This binary adds dense bit scans
    to Mosaic's region sums and the ring switch (shared vendor code that
    both methods use, with unchanged proofs) on top of e896b64. final/run.sh
    is the script that produced them:
    - interleave-n20: baseline (632855e) against the branch, 10 rounds;
    - interleave-pre-n20: PRE (0e9aa9c, the round-2 head with a committed
      read value and no interface claims) against the branch, 10 rounds;
    - interleave-prev-n20: PREV (e896b64, before the dense bit scans)
      against the branch, 10 rounds. It measures the dense bit scans alone;
    - branch-n20: all four methods in one process, 10 rounds;
    - private-n20, cycle-order-n20, chunk-sweep-n20, chunk-sweep-a20-n20,
      sweep-na and sweep-threads, as named in run.sh.
    Every sample of twist, spice-typed and spice-typed-2fp passed the
    harness check of the interface claims against the trace. The one-minute
    load before an invocation was 15.6 to 113.8 during interleave-n20, 26.7
    to 46.9 during interleave-pre-n20, 17.1 to 25.6 during
    interleave-prev-n20 and about 17 to 25 for the rest, so the intervals in
    interleave-n20 and interleave-pre-n20 are wide. The load before each run
    is in run.log and the .txt logs. All four binaries were built after
    `cargo clean --release` in one target directory.

final-e896b64/
    The previous third-round results (virtual read value and the CPU
    interface claims, before the dense bit scans), from ram-compare built at
    e896b64. final-e896b64/run.sh produced them (it wrote to final/ at the
    time). The one-minute load was 13 to 34 during that run (25 to 34 for
    interleave-n20 and 24 to 30 for interleave-pre-n20).

final-committed-rv/
    The first-round results, when Twist v2 committed its read value, from
    ram-compare built at ed1c284. final-committed-rv/run.sh produced them.
    The one-minute load was 17 to 48 during this run, so the
    branch/baseline intervals in interleave-n20.txt are wide.

final-192c417/
    An earlier baseline/branch interleaved run (10 rounds, 3 samples per
    invocation) from ram-compare built at 192c417, at one-minute load 7 to 14.
    Between 192c417 and ed1c284 the prover code differs only in that typed
    Spice moves or copies the final memory into ProveResult::final_memory and
    Twist v2 marks an internal helper unsafe; the rest of the difference is
    harness labelling, provenance fields, tests and documentation.

round2/
    The second round: the Twist v2 cycle prover changes, from ram-compare
    built at a186534 (the BRANCH binary; environment.git_commit in each JSON).
    round2/run.sh is the script that produced them:
    - interleave-n20: baseline (632855e) against the branch, 10 rounds;
    - branch-n20: all four methods in one process, 10 rounds;
    - ablation-n20: Twist v2 only, 6 interleaved rounds of five binaries that
      add the cycle prover changes one at a time: first-round (ed1c284),
      split-eq (c7df2c5), split-lt (eec558c), lookups (acdf49a) and skip-inc
      (the branch binary; its prover code equals 1f73411).
    The load before each run is in the .txt logs and, for the ablation, in
    ablation-n20/summary.txt.
    Every binary was built after `cargo clean --release` in one target
    directory (see docs/methodology.md, "Building binaries to compare"). An
    earlier round-2 run used a binary whose ram-argument code came from a
    stale worktree build. Its results were discarded and are not here.
