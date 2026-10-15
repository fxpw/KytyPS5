# Yōtei: active DLSS performance follow-up, 2026-10-15

Retained result: restore compute-chain in both local DLSS launch scripts.
Matched title FPS increases2.889→3.516/3.611 (+21.68%/+24.95%). Active DLSS,
character and village/HUD are confirmed; stable60FPS is not achieved. Affinity
and the new constant-batching prototype are NOT retained as extra speedups.
Current production matchesccd7b8a7, installed native executableFC3467A8.

Native Windows, branch `yotei-integration-native`. Three independent agents
reviewed CPU dependencies, immutable bound constants, and temporal RGB work.
Root serializes native builds, GPU fixtures and game runs. WSL shutdown was
requested, no Dota/emulator/build overlap. Original game/save/cache/captures and
user untracked prefetch fixture remain untouched. No push or issue publication.

## Matched title measurements

Window1920x1080, actual temporal input1920x1080/output3840x2160, same external
metadataCEDB4ECF, Performance request, neutral input, no menu navigation. New
measurement gate requires BOTH frame>=400 and completedDLSS>=200 before the
20s title-only sample. This avoids comparing opening flyover to the player/HUD
scene. Root viewed captures in A/B/C/D/E: the character and same village location
are visible. Individual captures do not prove full temporal quality/stability.
Pipeline caches were reused, not restored from identical immutable snapshots.

| Run suffix / variant | Mean title FPS | Range | Decision |
|---|---:|---|---|
|203954/5e2960 A: compute-chain0|2.88947|2.8..3.0|Matched control|
|204324/38e710 B: compute-chain1|3.51579|3.3..3.8|+21.68%; retain|
|204603/06eef8 C: B plus GPU_CORE1|3.57368|3.4..3.7|+1.65%; insufficient evidence, omit affinity|
|205226/724144 D: B plus batched constants|3.66111|3.2..3.9|Extra gain not confirmed by repeat control; reverted|
|205500/1d4e7f E: repeat B, original executable|3.61053|3.3..3.9|Confirms retained chain setting; batch delta only1.40%|

A/B/C use EXACT same coreFC3467A8, productionccd7b8a7. Source edits during
later runs do not change that executable; its earlier verified run record and
SHA are explicitly supplied to the launcher. A beginsframe541/completed202,
B frame539/completed201; each has19 title observations in the20s window.
A completed202→259; B201→273. Both exit0, no memory/timeout guards, real DLSS
evaluation/merge/completion continues. Raw runs are under `_Build/runs/` with
full names matching these suffixes; `verified-title-window.json` preserves rows.
Earlier exploratory203719/8db6c8 mean3.6421 started during the flyover and is
not the matched improvement estimate. Earlier2.7667FPS is also a different gate.

Compute-chain is an existing supported setting. The activation launcher had
explicitly forced it OFF. `temporal_boundary` excludes both declared DLSS passes
from chaining; `CommandBuffer::Handle` orders earlier pending compute writes.
The setting restores batching for unrelated compute dispatches. Recorder spin
2000 and label flush2000us were already defaults, so omitting those variables
was not a separate regression. Both `start-yotei-dlss.ps1` and `run-live-fps.ps1`
now set KYTY_COMPUTE_CHAIN=1 by default; both pass PowerShell syntax parsing.
The manual script also defaults to the tested Performance request; caller
AdditionalArguments still follow it, so an explicit later override is possible.
Affinity also changes process thread placement;
its small single-run difference is not treated as useful acceleration.

## Batched actual constant-buffer snapshots

Two synchronous144B native reads each submitted/waited before this change.
Native handle/offset identity is insufficient to reuse producer bytes: the SAME
buffer can contain different consumer data and cache generations are not exact
content revisions. The new ticket records each actual descriptor copy at its
own ordered point, retains its staging until the real GPU tick, then resolves
both snapshots with one bounded wait at accumulation. Exact serial/jitter/dt
checks, actual image roles, color identity/generation and frame guards remain.
Original guest rendering still executes on provider refusal.

Intended native RED204549/79d40b: exact A/B and independent GPU copies pass,
including overwrite of the same descriptor and recording continuation; only
cost2/2 versus expected1/1 fails. Already-submitted ticks, stream0submit,
invalid requests, max-three, lifetime and final Vulkan validation0 also pass.
Unchanged fixture SHA F242364F passes GREEN204934/c112a8 recorder0 and
204935/1fc33b recorder1: cost1/1, exact data, finalvalidation0. Old reader
oracle2891EDD1 passes204936/4de1bf with9 invalid no-submit neighbors and
recordingthread1. CPU contracts205028/3c143b pass5/5.

Core build205028/1dddd4 passes; commitbe078be3 production contents match core
`DFD05FF5CFEFD7297F8121645C99CDB818407F9F50418CDB4C5E13EF35A9B0CA`.
Game D exits0 and logs snapshots2/submits1 before real NGX evaluation/merge;
GPU completed200→272 in the measured player/HUD scene. However, repeat E on
the original exact executable returns3.61053, versus D3.66111: only1.40%,
inside observed short-run variation. Do not claim this as a batching speedup.
Commitbe078be3 was reverted by6f9c7972 per the user's discard-no-gain rule.
The API, fixture and provider draft are preserved as a local commit and patch
in `_Build/analysis/yotei-dlss-performance-20261015/batch-experiment/`.
Reduced native submits is a verified cost property, not proof of game FPS gain.
Exposure/motion/history quality remains experimental.

Installed runtime was restored to exact coreFC3467A8 and checked against the
tested production treeccd7b8a7. `_Build/windows-integration` still contains the
experimental DFD05FF5 build; rebuild before any later install from that directory.
All measured native workers are closed. No game settings, save, caches or
proprietary shader data were deleted. These runs reuse their compatible caches.

## Deferred alternatives

RGB store masking plus current DCE removes only about10.3% of captured IR
definitions and1/32 image reads because control Reference roots remain live.
Larger pruning requires CFG proof and transactional replay of the original
full dispatch on NGX failure, including input/output alias handling. Do not
skip the original alpha/R8 work or modify a globally shared shader by hash.

Private images already use GraphicContext's native image recycler. A new
three-slot pool would primarily save view creation and CPU bookkeeping; there
is no measured100MiB-per-frame allocation cost to claim. Reuse requires both
pending CPU ownership and all actual GPU leases to end.

Next unmeasured shared mechanism: classify optional eager downloads whose
tokens are repeatedly superseded by newer writes. Only proven redundant NEW
speculative admissions may be throttled; required reads, older partial-page
publications, retirement and owner-generation reset must remain correct.
The existing pre-DLSS profile shows many GPU→CPU boundaries but does not prove
how many current eager copies are wasted. First collect bounded local token
and byte counters, then build a finite native regression before production.

Detailed agent proposals are ignored artifacts in
`_Build/analysis/yotei-dlss-performance-20261015/`:
`cpu-dependency-and-eager-plan.md` and `temporal-rgb-pruning.md`.
