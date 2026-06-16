# Semantic Graph Project

A graph-theoretic and psycholinguistic investigation of how human semantic memory
is organized and retrieved. This project builds a directed weighted graph of English
semantic associations from crowdsourced word-association data, validates that graph
against human reaction-time priming experiments, and extends the analysis to
cross-linguistic comparison and second-language (L2) speaker behavior.

## Research Question

To what extent do the topological properties of empirical semantic graphs —
constructed from human word-association data — differ across language communities,
and can a validated Markov Chain Random Walk model of spreading activation detect
and quantify these cross-linguistic structural differences?

Three sub-questions follow from this:

- **RQ1.** Does shortest-path distance in the semantic graph significantly predict
  semantic priming reaction times, validating the graph as a model of human
  semantic memory?
- **RQ2.** How do graph-theoretic properties (clustering, centrality, path length,
  community structure) differ between English and a typologically distinct language?
- **RQ3.** Does a stochastic noise parameter `ε` applied to the Markov transition
  matrix accurately model the associative behavior of L2 speakers as compared to
  native (L1) speakers?

## Approach

The project moves through four analytical phases.

**Phase 1 — Graph construction.** Build a directed weighted graph `G = (V, E, W)`
from the Small World of Words (SWOW) dataset. Words become nodes; population-level
cue-to-response association strengths become directed edge weights. A standard
preprocessing pipeline filters non-content tokens and rare associations before
the graph is materialized.

**Phase 2 — Empirical validation.** Compute Dijkstra shortest-path distances (with
edge weights inverted as `1/Wᵢⱼ`) for prime-target pairs in the Semantic Priming
Project (SPP) dataset. Correlate graph distance against z-scored reaction-time
priming effects. A statistically significant negative correlation establishes that
the graph captures real associative structure in human memory.

**Phase 3 — Cross-linguistic comparison.** Construct a second graph from SWOW data
in another language (Dutch or French, the next-richest SWOW releases). Compare
clustering coefficients, average path lengths, PageRank centralities, and Louvain
community structures across the two graphs. Universal anchor concepts and
language-specific hubs are identified.

**Phase 4 — Markov walks and L2 calibration.** Simulate spreading activation as
a random walk on the graph using the transition matrix
`Pᵢⱼ = Wᵢⱼ / Σₖ Wᵢₖ`. Introduce a noise parameter `ε ∈ [0, 1]` that interpolates
between strong associative transitions and uniform random transitions:

```
P'ᵢⱼ = (1 − ε) · Pᵢⱼ + ε · (1 / |V|)
```

The walk distribution at each `ε` is compared (via KL or Jensen-Shannon divergence)
against the empirical association behavior of non-native English speakers in SWOW.
The value of `ε` that best fits L2 data anchors the noise parameter to a real
psycholinguistic phenomenon rather than leaving it as an abstract construct.

## Datasets

**Small World of Words (SWOW).** A crowdsourced word-association database in which
participants give the first words that come to mind in response to a cue. Provides
population-level cue → response strength values that serve directly as edge weights.
English data is the primary working set; a second language is selected for Phase 3.
Available at <https://smallworldofwords.org>.

**Semantic Priming Project (SPP).** A large lexical-decision priming dataset
containing z-scored reaction-time priming effects for ~1,661 prime-target pairs at
both short (200ms) and long (1200ms) stimulus-onset asynchronies. Serves as the
empirical ground truth for Phase 2 validation. Available at
<https://www.montana.edu/attmemlab/spp.html>.

The raw data is not included in this repository. Download instructions are below.

## Project Structure

```
semantic_graph_project/
├── data/
│   ├── raw/         # raw SWOW and SPP downloads (gitignored)
│   └── processed/   # cleaned data and serialized graphs (gitignored)
├── notebooks/       # numbered Jupyter notebooks, one per analytical step
├── src/             # reusable Python modules
├── notes/           # written observations, methodology decisions, data dictionaries
└── README.md
```

## Setup

The project uses Python 3.13 and standard scientific computing libraries.

```bash
# Clone and enter the repo
git clone https://github.com/strontium-rn/semantic_graph.git
cd semantic_graph_project

# Create a virtual environment
python -m venv .venv
source .venv/bin/activate          # macOS / Linux
# .venv\Scripts\activate           # Windows

# Install dependencies
pip install pandas numpy scipy networkx matplotlib seaborn \
            python-louvain tqdm xlrd openpyxl
```

## Acquiring the Data

The raw datasets must be downloaded separately and placed in `data/raw/`.

**SWOW (English).** Register at <https://smallworldofwords.org/en/project/research>
and download the English 2018 release (`SWOW-EN18`). Place the entire folder at
`data/raw/SWOW-EN18/`.

**SPP.** From <https://www.montana.edu/attmemlab/spp.html>, download:
- `items_spreadsheet.xls` — item characteristics
- `ldt word item analysis.xlsx` — z-scored priming effects (the primary file for
  Phase 2 validation)

Place both at `data/raw/`.

## Implementation Notes

The graph is built and analyzed in Python using **NetworkX**. Markov walks and
matrix operations are handled with **NumPy** and **SciPy**. Community detection
uses the **python-louvain** implementation of the Louvain modularity-maximization
algorithm. Statistical testing uses **SciPy** (correlation and Mann–Whitney U
tests). Visualization is split between **matplotlib** and **seaborn** for
distributions and **Gephi** for full network layouts.

## Theoretical Background

The project sits at the intersection of three traditions. The **Spreading
Activation** theory of semantic memory (Collins & Loftus, 1975) provides the
cognitive grounding: activating one concept propagates excitation along
associative links to neighboring concepts, raising their activation level. The
**Sapir–Whorf** (linguistic relativity) hypothesis provides the cross-linguistic
motivation: if language influences habitual patterns of thought, language
communities should organize conceptual space measurably differently. **Network
science** provides the mathematical machinery — shortest paths, centrality,
community structure, and random walks — through which both ideas can be made
quantitatively testable on real psycholinguistic data.