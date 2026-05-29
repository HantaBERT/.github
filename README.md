---
title: HantaBERT
emoji: 🧬
colorFrom: green
colorTo: yellow
sdk: static
pinned: true
---

<p align="center">
  <img src="assets/HantaBERT_logo_wide_transparent.png" alt="HantaBERT" width="560">
</p>

<p align="center">
  <b>Multi-Task <i>Orthohantavirus</i> classification by fine-tuning DNABERT-2.</b><br>
  One forward pass &rarr; <b>species</b>, <b>host</b>, and <b>geographic origin</b>, plus a 768-d phylogenetic embedding.
</p>

<p align="center">
  <a href="https://hantabert.faizath.com/">🌐 Web App</a> &nbsp;·&nbsp;
  <a href="https://hantabert-api.faizath.com/docs">⚡ API Docs</a> &nbsp;·&nbsp;
  <a href="https://huggingface.co/HantaBERT/HantaBERT">🤗 Model</a> &nbsp;·&nbsp;
  <a href="https://huggingface.co/datasets/HantaBERT/Orthohantavirus-Genome-Atlas">📊 Dataset</a> &nbsp;·&nbsp;
  <a href="https://github.com/HantaBERT">💻 GitHub</a>
</p>

---

## What is HantaBERT?

Hantaviruses (genus *Orthohantavirus*) are segmented negative-sense ssRNA viruses that cause hemorrhagic fever with renal syndrome (HFRS) in Eurasia and cardiopulmonary syndrome (HCPS) in the Americas, with mortality reaching ~40% in HCPS cases. Rapidly identifying the **species**, **reservoir host**, and **geographic origin** of a sequence is essential for surveillance, but BLAST and classical phylogeny are slow and do not integrate across attributes.

**HantaBERT** fine-tunes [DNABERT-2 (117M)](https://github.com/Zhihan1996/DNABERT_2) into a *multi-task* model that emits probabilities for all three tasks in a **single forward pass**. A shared 768-d bottleneck feeds three independent classification heads, trained with a weighted combined loss, balanced classes, AMP fp16, gradient accumulation, and a differential learning rate between encoder and heads.

## 📈 Headline results

Held-out test set (883 sequences), neural classification heads:

| Task | Classes | Test accuracy |
|------|:-------:|:-------------:|
| 🧬 **Species / lineage** | 23 | **96.7%** |
| 🐀 **Host** (Rodent / Human / Others) | 3 | **91.4%** |
| 🌍 **Geographic origin** | 7 | **80.5%** |

<table>
  <tr>
    <td width="50%"><img src="assets/training_curves.png" alt="Training curves" width="100%"></td>
    <td width="50%"><img src="assets/cm_species.png" alt="Species confusion matrix" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><sub><b>Training progression</b>: val accuracy climbs to 96.7% / 91.4% / 80.5% over 10 epochs.</sub></td>
    <td align="center"><sub><b>Species confusion matrix</b>: clean diagonal across 23 lineages.</sub></td>
  </tr>
</table>

### Emergent phylogenetic structure

A UMAP projection of all **8,822** bottleneck embeddings reveals clean per-lineage clusters (*and* substructure per genome segment S, M, L) with **no explicit supervision of the segment**. The S/M/L separation tracks differences in selective pressure (conserved N protein on S, antigenic positive selection on Gn/Gc in M, active RdRp motifs on L).

<table>
  <tr>
    <td width="50%"><img src="assets/umap_all_species.png" alt="UMAP of all species" width="100%"></td>
    <td width="25%"><img src="assets/umap_Orthohantavirus_seoulense.png" alt="UMAP Seoul virus" width="100%"></td>
    <td width="25%"><img src="assets/umap_Orthohantavirus_puumalaense.png" alt="UMAP Puumala virus" width="100%"></td>
  </tr>
  <tr>
    <td align="center"><sub><b>All 8,822 sequences</b> by lineage</sub></td>
    <td align="center"><sub><b>Seoul virus</b> (1,391)</sub></td>
    <td align="center"><sub><b>Puumala virus</b> (2,709)</sub></td>
  </tr>
</table>

---

## 🗂️ Project components

The HantaBERT stack spans data collection, modeling, a public API, and a web interface, each in its own repository.

### 📊 Data pipeline & dataset
Automated extraction, cleaning, multi-task labeling, and geocoding of *Orthohantavirus* genomic records from NCBI GenBank (Biopython + Nominatim). Produces the ready-to-train dataset of S/M/L RNA segments with standardized host, species, and geography labels.

- 🤗 Dataset: **[HantaBERT/Orthohantavirus-Genome-Atlas](https://huggingface.co/datasets/HantaBERT/Orthohantavirus-Genome-Atlas)**: `raw` (9,950), `interim` (9,846), `default` processed (9,846)
- 💻 Code: **[github.com/HantaBERT/data-pipeline](https://github.com/HantaBERT/data-pipeline)**

### 🧠 Model: training & fine-tuning
The multi-task fine-tuning code: `MultiTaskHantaBERT` (DNABERT-2 encoder → shared bottleneck → 3 heads), weighted loss `1.0·L_species + 0.5·L_host + 0.3·L_geo`, full train / evaluate / visualize scripts, and an SVM-on-embeddings baseline.

- 🤗 Model: **[HantaBERT/HantaBERT](https://huggingface.co/HantaBERT/HantaBERT)**
- 💻 Code: **[github.com/HantaBERT/HantaBERT](https://github.com/HantaBERT/HantaBERT)**

### ⚡ Inference API
FastAPI + Uvicorn service, packaged with Docker. Accepts raw DNA/RNA or FASTA, auto-converts U→T, and returns top-N probabilistic predictions per task.

- 🚀 Live: **[hantabert-api.faizath.com/docs](https://hantabert-api.faizath.com/docs)**
- 💻 Code: **[github.com/HantaBERT/HantaBERT-API](https://github.com/HantaBERT/HantaBERT-API)**

### 🌐 Web interface
Pure static HTML/CSS/JS frontend with an interactive world map (D3 + TopoJSON). Paste a sequence or upload a FASTA file and explore ranked predictions across all three tasks.

- 🌍 Live: **[hantabert.faizath.com](https://hantabert.faizath.com/)**
- 💻 Code: **[github.com/HantaBERT/HantaBERT-Web](https://github.com/HantaBERT/HantaBERT-Web)**

<p align="center">
  <img src="assets/website-interface.png" alt="HantaBERT web interface" width="80%">
</p>

### 📄 Paper
*HantaBERT: Multi-Task Hantavirus Classification with DNABERT-2 Fine-Tuning*, an IEEE-style conference paper (English & Indonesian), written for the **IF3211 Domain-Specific Computation (Bioinformatics)** course at STEI ITB.

- 💻 Source & PDFs: **[github.com/HantaBERT/paper](https://github.com/HantaBERT/paper)**

---

## 🚀 Quick links

| | Web / Hub | Source |
|---|---|---|
| **Model** | [🤗 HantaBERT/HantaBERT](https://huggingface.co/HantaBERT/HantaBERT) | [github.com/HantaBERT/HantaBERT](https://github.com/HantaBERT/HantaBERT) |
| **Dataset** | [🤗 Orthohantavirus-Genome-Atlas](https://huggingface.co/datasets/HantaBERT/Orthohantavirus-Genome-Atlas) | [github.com/HantaBERT/data-pipeline](https://github.com/HantaBERT/data-pipeline) |
| **API** | [hantabert-api.faizath.com/docs](https://hantabert-api.faizath.com/docs) | [github.com/HantaBERT/HantaBERT-API](https://github.com/HantaBERT/HantaBERT-API) |
| **Web** | [hantabert.faizath.com](https://hantabert.faizath.com/) | [github.com/HantaBERT/HantaBERT-Web](https://github.com/HantaBERT/HantaBERT-Web) |
| **Paper** | n/a | [github.com/HantaBERT/paper](https://github.com/HantaBERT/paper) |

---

## 👥 Authors

Muhammad Faiz Atharrahman · Muhammad Rafi Dhiyaulhaq · Lydia Gracia, School of Electrical Engineering and Informatics (STEI), Institut Teknologi Bandung.

Developed as the final project for the **IF3211 Domain-Specific Computation (Bioinformatics)** course at STEI ITB.

Released under the **Apache-2.0** license, consistent with the DNABERT-2 backbone. If you use HantaBERT, please also cite [DNABERT-2 (Zhou et al., 2023)](https://arxiv.org/abs/2306.15006).
