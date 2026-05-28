# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

Japanese-language teaching materials for an introductory bioinformatics / machine-learning course. The repo is **not** an application — it is a collection of curated biological datasets plus four self-contained Jupyter tutorial notebooks. There is no build system, no test suite, and no package manifest. Each notebook installs its own dependencies via a top-cell `%pip install …`.

All prose (markdown cells, plot titles, print messages, discussion questions) is in **Japanese**; code, identifiers, and column names are English. When editing notebook content, match this convention — keep code English, narration Japanese.

## How to run

Open any notebook under `ml/` in Jupyter / VS Code and run cells top-to-bottom. The first `%pip install …` cell is the dependency manifest for that notebook — there is no central `requirements.txt`.

The two `ProtBERT`-based notebooks (`protein_classification_tutorial.ipynb`, `gpcr_ligand_prediction_tutorial.ipynb`) download `Rostlab/prot_bert` from Hugging Face on first run (~1.6 GB) and will use CUDA automatically if available.

`wine_quality_ml_tutorial.ipynb` pulls CSVs over HTTP from `archive.ics.uci.edu`; `penguins_ml_tutorial.ipynb` gets data via the `palmerpenguins` Python package. Neither uses the FASTA data shipped in this repo.

## Japanese font rendering — critical ordering

Notebooks use `matplotlib-fontja` to render Japanese in plots. The setup cell in `penguins_ml_tutorial.ipynb` documents a non-obvious ordering constraint:

```python
sns.set_style("whitegrid")          # MUST come first
import matplotlib_fontja            # then this
plt.rcParams["font.sans-serif"] = ["IPAexGothic"] + _sans  # belt-and-suspenders
```

Reason: any `sns.set_*` call resets `rcParams`, which would undo the font registration if done after. If you add a new notebook with Japanese plots, replicate this ordering or expect tofu (□□□) instead of characters.

## Data layout

```
birds/   31 bird species   — cytb mitochondrial sequences + metadata + PhyloPic silhouettes
mammals/ 37 mammal species — cytb mitochondrial sequences + metadata + PhyloPic silhouettes
gpcr/    25 human GPCRs    — sequences + UniProt index + AlphaFold-predicted PDB structures
ml/      4 tutorial notebooks
```

FASTA header formats are custom and parsed positionally in any downstream tooling:

- `birds/`, `mammals/`: `>Scientific_name|RefSeq_accession|CYTB`
- `gpcr/all_GPCRs.fasta`: `>GENE|UniProt|ClassA|Description|RefSeq:NP_xxxxx`

PDB filenames follow `gpcr/pdb/{GENE}_AF-{UniProt}.pdb` (AlphaFold predictions). PhyloPic image directories ship an `phylopic_attribution.tsv` that records artist, license URL, and `matched_level` (whether the silhouette is for the exact species, genus, or higher) — preserve that file when adding or removing images, downstream notebooks may use it for attribution.

The four datasets in `birds/mammals/gpcr/` are intended as raw material for the tutorials but the tutorials currently embed their own example sequences inline rather than loading from these files. Wiring a notebook to the repo's FASTAs is a reasonable extension.

## Notebook map

| Notebook | Domain | Supervised | Unsupervised |
| --- | --- | --- | --- |
| `penguins_ml_tutorial.ipynb` | Palmer Penguins morphology | logistic-regression GLM (sex), Random Forest (species) | hierarchical / K-means / k-NN+Louvain clustering, PCA/MDS/t-SNE/UMAP |
| `wine_quality_ml_tutorial.ipynb` | UCI wine chemistry | Poisson GLM, Random Forest (quality score) | hierarchical clustering (red vs white) |
| `protein_classification_tutorial.ipynb` | Transmembrane vs soluble | ProtBERT embedding + LR / RF | t-SNE of embeddings |
| `gpcr_ligand_prediction_tutorial.ipynb` | GPCR-peptide binding | ProtBERT (paired embeddings) + RF / XGBoost | — |

Each tutorial deliberately pairs an interpretable model (GLM with `statsmodels` summary) against a black-box model (Random Forest / XGBoost) so students can compare coefficients vs. feature importances. When extending a notebook, keep this pedagogical contrast intact.
