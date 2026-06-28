# Semantic Graph Project

We model how people associate words as a big graph (words = dots, "this word reminds me
of that word" = arrows) and poke at it three different ways. The data comes from the
[Small World of Words](https://smallworldofwords.org) (SWOW) word-association project.

The whole project boils down to one finding: **the useful signal lives in the graph's
structure (how words are wired together), not in distances or random movement through
it.** Short version — *structure wins, dynamics lose.*

The project is **done**. All phases have been run and written up.

## What We Found (TL;DR)

| Phase | Notebook | Question | Answer |
|-------|----------|----------|--------|
| 1 | `01`, `02` | Build the English word-graph | 24,657 words, 279,410 connections |
| 2 | `03` | Does distance in the graph predict real priming behaviour? | **No** (r = -0.022). But word *popularity* (fan-in) does. |
| 3 | `04` | Do English and Dutch graphs differ? | Same overall shape, but Dutch is more clustered and hub-heavy. |
| 4 | `05` | Can we fake a non-native speaker by adding noise? | **No** — clean walk fits best for everyone. But native vs non-native graphs *are* genuinely different (the 2x2 test). |

Two of the headline results are clean "no" answers — that's intentional, not a failure.
Well-checked negatives are real findings.

## Setup

The project uses Python 3.13.

```bash
# Clone the repo
git clone https://github.com/strontium-rn/semantic_graph.git
cd semantic_graph

# Create a virtual environment
python -m venv .venv
source .venv/bin/activate        # macOS / Linux
# .venv\Scripts\activate         # Windows

# Install dependencies
pip install -r requirements.txt
```

## Acquiring the Data

The raw datasets must be downloaded separately and placed in `data/raw/`.

**SWOW (English).** Register at <https://smallworldofwords.org/en/project/research>
and download the English 2018 release (`SWOW-EN18`). Place the entire folder at
`data/raw/SWOW-EN18/`. You need both the strength file
(`strength.SWOW-EN.R123.20180827.csv`, used to build the graph) and the complete
per-participant file (`SWOW-EN.complete.20180827.csv`, used in Phase 4 — it has the
`nativeLanguage` field for the native/non-native split).

**SWOW (Dutch).** From the same site, download the Dutch 2012 release. Place the
folder at `data/raw/DutchWordAssociationData_April2012C.../`. Note: it ships as raw
trials (no strength file, so Phase 3 rebuilds the weights) and has no native/non-native
field — that's why Phase 4 only uses English.

**SPP.** From <https://www.montana.edu/attmemlab/spp.html>, download:
- `items_spreadsheet.xls` — item characteristics
- `ldt word item analysis.xlsx` — z-scored priming effects
- `all ldt subs_all trials3.xlsx` — trial-level data

Place all three at `data/raw/`.

> **Gotcha:** some SWOW files are tab-separated despite the `.csv` extension (the
> strength and responseStats files). If you load one by hand, use `sep="\t"` or
> `sep=None, engine="python"`.

## Running the Notebooks

Run notebooks in numbered order. Each phase generates processed files that the next
phase depends on. **Do not skip or reorder.**

| Notebook | Phase | What it does |
|----------|-------|-------------|
| `01_data_exploration.ipynb` | Phase 1 | Explores raw SWOW data |
| `02_graph_construction.ipynb` | Phase 1 | Builds and saves `swow_en_graph.pkl` |
| `03_spp_validation.ipynb` | Phase 2 | Loads graph, runs priming correlation + diagnostics |
| `04_dutch_exploration.ipynb` | Phase 3 | Builds Dutch graph, cross-linguistic comparison |
| `05_L2validation.ipynb` | Phase 4 | Native/non-native split, noise walk, 2x2 structural test |

> If you have received the processed data files via the shared Drive link, you can
> skip earlier notebooks and open a later one directly to see that phase's state.
> Phase 4 (`05`) also needs `swow_en_graph.pkl` and the complete per-participant file.

**The five cleaning rules** (applied identically in every phase, in order):
1. Lowercase the cue and response.
2. Drop single-character responses.
3. Drop multi-word responses.
4. Keep only purely alphabetic responses.
5. Drop connections supported by fewer than 2 participants.

Connection strength = the fraction of people who gave that response to a cue. Expensive
steps are cached to disk with a validation stamp, so re-running skips work but redoes it
automatically if the inputs change.

## Phases at a Glance

**Phase 1 — build the graph.** From the SWOW-EN strength file. Cleaning trims ~979k raw
rows to ~279k connections. Result: 24,657 words in one connected component, ~8,659 of
them cue words with outgoing arrows, the rest response-only.

**Phase 2 — does distance predict behaviour?** We tested whether words *close* in the
graph prime faster in the lab (SPP). They don't — basically zero correlation, robust
across four diagnostics. So the graph isn't a validated model of priming *by distance*.
But fan-in (how many words point to a word) *does* carry signal. First sign of "structure
beats dynamics."

**Phase 3 — English vs Dutch.** Built the Dutch graph the same way and compared, with
controls for size and collection protocol. Similar density, compactness, and modularity,
but Dutch is more clustered and more hub-concentrated. Differences survived every control.

**Phase 4 — can noise fake an L2 speaker?** Split English speakers into native (72,429)
and non-native (16,293), matched data sizes so the smaller group wasn't unfairly
"noisier," and tuned a noise dial (epsilon) on a random walk. It failed — zero noise fit
best for both groups, under two noise models. But the 2x2 test (a graph from each group,
checked against both groups) showed each graph fits its *own* group best, symmetrically.
The native vs non-native difference is real, and it's structural.

## Implementation Notes

The graph is built and analyzed in Python using **NetworkX**. Markov walks and matrix
operations use **NumPy** and **SciPy**. Community detection uses **python-louvain**
(Louvain modularity maximization). Statistical testing uses **SciPy** (Pearson
correlation, Mann–Whitney U) and **statsmodels** (regression). Phase 4 compares answer
distributions with **Jensen–Shannon divergence** (0 = identical, 1 = totally different;
lower = better match). Visualization uses **matplotlib** and **seaborn** for
distributions, and **Gephi** for full network layouts.

## Theoretical Background

The project sits at the intersection of three traditions. **Spreading Activation**
theory (Collins & Loftus, 1975) provides the cognitive grounding: activating one
concept propagates excitation along associative links to neighboring concepts. The
**Sapir–Whorf hypothesis** (linguistic relativity) provides the cross-linguistic
motivation: if language shapes thought, language communities should organize conceptual
space measurably differently. **Network science** provides the mathematical machinery —
shortest paths, centrality, community structure, and random walks — through which both
ideas can be made quantitatively testable on real psycholinguistic data.

One commitment runs throughout: report carefully controlled structural differences and
well-defended negative results rather than overstated dynamic claims. This is a study of
word-association data, not a model of the brain.