# Paper Copilot OpenReview Archive

This repository is the OpenReview-derived dataset archive for
[Paper Copilot](https://papercopilot.com/). It contains standard
venue-year records, author/profile enrichment, PDF metadata, and a
timestamped ICLR subset with code and derived temporal reviewer records.

Canonical repository:
[github.com/papercopilot/openreview](https://github.com/papercopilot/openreview)

## Dataset

The general archive is organized as JSON/JSONL data products:

| Path | Contents |
| --- | --- |
| `venues/<venue>/<venue><year>.jsonl` | Normalized per-paper OpenReview records for a venue-year. |
| `authors/<venue>/<venue><year>.jsonl` | Author profile and affiliation enrichment keyed by OpenReview forum ID. |
| `pdf/<venue>/<venue><year>.jsonl` | PDF metadata keyed by OpenReview forum ID. |

Current inventory: 45 venue-year JSONL files, 40 author-enrichment JSONL
files, and 9 PDF metadata JSONL files. Covered venues include ICLR,
NeurIPS/NIPS, ICML, CoRL, COLM, EMNLP, AISTATS, 3DV, WWW, ALT, AI4X,
and ACMMM.

Each `.jsonl` file stores one JSON object per line. Fields vary by venue
and year because OpenReview forms evolve, but venue records commonly
include:

- Paper metadata: `id`, `title`, `track`, `status`, `abstract`,
  `primary_area`, `site`, `bibtex`.
- Authorship/profile metadata: `author`, `authorids`, `or_profile`,
  `aff`, `aff_domain`, `position`, `homepage`, `dblp`, `google_scholar`,
  `orcid`, `linkedin`.
- Review signals: semicolon-separated per-reviewer fields such as
  `rating`, `confidence`, `soundness`, `contribution`, `presentation`,
  `correctness`, or `technical_novelty`.
- Aggregates and activity: fields ending in `_avg`, plus word/reply
  counts such as `wc_review`, `reply_reviewers`, and `reply_authors`.

The processed paper-list release that combines this OpenReview archive
with official conference pages and other sources lives at
[github.com/papercopilot/paperlists](https://github.com/papercopilot/paperlists).
Use this data for aggregate, reproducible research; do not attempt to
de-anonymize reviewers or use parsed profile metadata for high-stakes
individual decisions.

## Timestamped ICLR Subset

The archive also includes timestamped ICLR snapshots for studying how
review scores evolve over time. This subset is the data release behind
the ICLR Daily Score Archive used in
[Paper Copilot: Tracking the Evolution of Peer Review in AI Conferences](https://arxiv.org/abs/2510.13201).
The paper is affiliated with the University of Southern California,
University of Cambridge, Stanford University, and Paper Copilot.

Raw snapshots are stored under:

| Path | Contents |
| --- | --- |
| `venues/iclr/iclr2024/*.jsonl` | ICLR 2024 daily or near-daily review snapshots. |
| `venues/iclr/iclr2025/*.jsonl` | ICLR 2025 daily or near-daily review snapshots. |
| `venues/iclr/iclr2026/*.jsonl` | ICLR 2026 continuing archive snapshots. |

Snapshot filenames encode collection time:

```text
iclrYYYY.MMDDYYYY.jsonl
iclrYYYY.MMDDYYYY.HH.jsonl
```

Each snapshot line uses the same JSONL record format as the general
dataset. Compare records by `id` across snapshots to reconstruct a
paper's review timeline.

Processing code for this subset lives under `code/iclr2026/`:

| Path | Purpose |
| --- | --- |
| `code/iclr2026/temporal.py` | Traces reviewer identities across timestamped snapshots and builds reviewer-level temporal JSON. |
| `code/iclr2026/temporal_visual.py` | Visualizes traced reviewer-score footprints. |

Example visualization generated from traced ICLR reviewer footprints:

![Example reviewer-score footprint](code/iclr2026/reviewer-footprint-example.jpg)

The temporal pipeline builds reviewer-level JSON files named:

```text
iclrYYYY_threshold<k>_<n>_reviewers.json
```

The same derived record can be written in first/last mode or full-sequence
mode. The snippet below shows one reviewer from the same paper in both
modes.

With `--first_last_only`:

```json
{
  "id": "00SnKBGTsz",
  "title": "DataEnvGym: Data Generation Agents in Teacher Environments with Student Feedback",
  "tracing_score": 2,
  "review": {
    "rVo8": {
      "rating": "5;6",
      "confidence": "4;4",
      "soundness": "2;2",
      "contribution": "3;3",
      "presentation": "3;3"
    }
  }
}
```

With `--first_last_only` disabled:

```json
{
  "id": "00SnKBGTsz",
  "title": "DataEnvGym: Data Generation Agents in Teacher Environments with Student Feedback",
  "tracing_score": 2,
  "review": {
    "rVo8": {
      "rating": "5;5;5;5;5;5;5;5;5;5;5;5;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6;6",
      "confidence": "4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4;4",
      "soundness": "2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2;2",
      "contribution": "3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3",
      "presentation": "3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3;3"
    }
  }
}
```

For ICLR 2024/2025, each timestamped paper record stores all reviewer
scores for that paper at one collection time. Fields such as `rating`,
`confidence`, `soundness`, and `contribution` follow the same reviewer
order, and that order is sorted by `rating`. This is enough to describe a
single snapshot, but not enough to directly build per-reviewer score
trajectories: if a reviewer changes their rating, their place in the
rating-sorted list can change in the next snapshot.

The recovery idea is simple: use the non-rating scores as the reviewer's
fingerprint. `rating` controls the sorting, so a rating change can move a
reviewer in the list; `confidence`, `soundness`, `contribution`, and the
other non-rating fields move with that reviewer. The code ignores
`rating`, matches reviewers by the non-rating fields, and records the
minimum allowed difference as `tracing_score`.

The toy examples below use `R = rating`, `C = confidence`, and `S =
soundness`; `[...]` marks the reviewer being traced. The real code uses all
non-rating dimensions available for that year.

The per-reviewer matching cost is:

```math
\mathrm{tracing\_score} =
\begin{cases}
-1, & \text{if no remapping is needed} \\
\sum_{d \in D_{\text{non-rating}}} |d_{\text{before}} - d_{\text{after}}|, & \text{otherwise}
\end{cases}
```

A larger `tracing_score` means the match required more non-rating mismatch,
so the recovered reviewer trajectory is less reliable.

<table>
  <thead>
    <tr>
      <th width="24%">Before</th>
      <th width="24%">After</th>
      <th width="34%"><code>tracing_score</code></th>
      <th width="18%">What changed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><pre><code>R: 4, [6], 7, 8
C: 2, [3], 4, 4
S: 2, [3], 4, 5</code></pre></td>
      <td><pre><code>R: 4, [6], 7, 8
C: 2, [3], 4, 4
S: 2, [3], 4, 5</code></pre></td>
      <td><code>unchanged = -1</code></td>
      <td>Nothing changed; no remapping was needed.</td>
    </tr>
    <tr>
      <td><pre><code>R: 4, [6], 7, 8
C: 2, [3], 4, 4
S: 2, [3], 4, 5</code></pre></td>
      <td><pre><code>R: 4, 7, 8, [9]
C: 2, 4, 4, [3]
S: 2, 4, 5, [3]</code></pre></td>
      <td><code>abs(C: 3-3) + abs(S: 3-3) = 0</code></td>
      <td>Moved by rating; non-rating scores match.</td>
    </tr>
    <tr>
      <td><pre><code>R: 4, [6], 7, 8
C: 2, [3], 4, 4
S: 2, [3], 4, 5</code></pre></td>
      <td><pre><code>R: 4, 7, 8, [9]
C: 2, 4, 4, [4]
S: 2, 4, 5, [3]</code></pre></td>
      <td><code>abs(C: 3-4) + abs(S: 3-3) = 1</code></td>
      <td>Moved; one non-rating score differs by one point.</td>
    </tr>
    <tr>
      <td><pre><code>R: 4, [6], 7, 8
C: 2, [3], 4, 4
S: 2, [3], 4, 5</code></pre></td>
      <td><pre><code>R: 4, 7, 8, [9]
C: 2, 4, 4, [4]
S: 2, 4, 5, [4]</code></pre></td>
      <td><code>abs(C: 3-4) + abs(S: 3-4) = 2</code></td>
      <td>Moved; two non-rating scores differ by one point.</td>
    </tr>
    <tr>
      <td><pre><code>R: 4, [6], 7, 8
C: 2, [3], 4, 4
S: 2, [3], 4, 5</code></pre></td>
      <td><pre><code>R: 4, 7, 8, [9]
C: 2, 4, 4, [6]
S: 2, 4, 5, [6]</code></pre></td>
      <td><code>abs(C: 3-6) + abs(S: 3-6) = 6 (&gt;2)</code></td>
      <td>Larger-difference or unrecovered cases.</td>
    </tr>
  </tbody>
</table>

Recovery summary, cumulative by maximum allowed `tracing_score`:

| Conference | Total records | `<= 0` | `<= 1` | `<= 2` | Final recovered (`<= 5`) |
| --- | ---: | ---: | ---: | ---: | ---: |
| ICLR 2024 | 2,611 | 1,378 (52.78%) | 2,217 (84.91%) | 2,537 (97.17%) | 2,610 (99.96%) |
| ICLR 2025 | 5,659 | 2,344 (41.42%) | 4,205 (74.31%) | 5,224 (92.31%) | 5,657 (99.96%) |

For ICLR 2026, the snapshots also capture the late-November 2025 reviewer
identity leak and score reset
([OpenReview note](https://openreview.net/forum?id=uAkexWJ7dW&noteId=ObG5ao5t4e)).
Score changes across that boundary should be interpreted as the reset
event, not ordinary reviewer-score movement.

ICLR 2024 uses `rating`, `confidence`, `correctness`, and
`technical_novelty`; ICLR 2025/2026 use `rating`, `confidence`,
`soundness`, `contribution`, and `presentation`.

Paper links:

- ICLR 2026 poster:
  [iclr.cc/virtual/2026/poster/10010812](https://iclr.cc/virtual/2026/poster/10010812)
- OpenReview:
  [openreview.net/forum?id=CyKVrhNABo](https://openreview.net/forum?id=CyKVrhNABo)
- arXiv:
  [arxiv.org/abs/2510.13201](https://arxiv.org/abs/2510.13201)

If you use the timestamped ICLR subset, please cite:

```bibtex
@inproceedings{yang2026papercopilot,
  title = {Paper Copilot: Tracking the Evolution of Peer Review in AI Conferences},
  author = {Yang, Jing and Wei, Qiyao and Pei, Jiaxin},
  booktitle = {The Fourteenth International Conference on Learning Representations},
  year = {2026},
  url = {https://openreview.net/forum?id=CyKVrhNABo}
}
```
