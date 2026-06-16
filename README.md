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
`data/raw/SWOW-EN18/`.

**SWOW (Dutch).** From the same site, download the Dutch 2012 release. Place the
folder at `data/raw/DutchWordAssociationData_April2012C.../`.

**SPP.** From <https://www.montana.edu/attmemlab/spp.html>, download:
- `items_spreadsheet.xls` — item characteristics
- `ldt word item analysis.xlsx` — z-scored priming effects
- `all ldt subs_all trials3.xlsx` — trial-level data

Place all three at `data/raw/`.

## Running the Notebooks

Run notebooks in numbered order. Each phase generates processed files that the next
phase depends on. **Do not skip or reorder.**

| Notebook | Phase | What it does |
|----------|-------|-------------|
| `01_data_exploration.ipynb` | Phase 1 | Explores raw SWOW data |
| `02_graph_construction.ipynb` | Phase 1 | Builds and saves `swow_en_graph.pkl` |
| `03_spp_validation.ipynb` | Phase 2 | Loads graph, runs priming correlation |
| `04_dutch_exploration.ipynb` | Phase 3 | Builds Dutch graph, cross-linguistic comparison |

> If you have received the processed data files via the shared Drive link, you can
> skip notebooks 01–03 and open `04_dutch_exploration.ipynb` directly to see the
> current state of the project.

## Implementation Notes

The graph is built and analyzed in Python using **NetworkX**. Markov walks and matrix
operations use **NumPy** and **SciPy**. Community detection uses **python-louvain**
(Louvain modularity maximization). Statistical testing uses **SciPy** (Pearson
correlation, Mann–Whitney U). Visualization uses **matplotlib** and **seaborn** for
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