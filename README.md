# DrugNet: Faster Drug Discovery Using Generative Deep Learning

DrugNet is a computational drug-discovery system that generates drug-like molecules for a chosen
target protein and ranks them without leaving the computer. It couples a generative molecular
representation to a learned target-affinity model, so that a search algorithm can propose molecules
and score them against a protein in a single loop.

A full technical write-up is in [`drugnet.pdf`](drugnet.pdf) — a self-contained preprint covering
the mathematics, architecture, experiments and reproducibility caveats.

## How it works

DrugNet has four components, each supplying something the previous one lacks.

- **MAE — Molecular Autoencoder.** A SELFIES sequence autoencoder (bidirectional GRU encoder,
  GRU decoder) that maps molecules into an 84-dimensional continuous latent space and back. This is
  the interface between discrete chemistry and continuous search.
- **BGMM — Bayesian Gaussian Mixture Model.** Because the autoencoder is deterministic rather than
  variational, a 12-component mixture is fitted afterwards to the encoded training set, giving a
  distribution to sample plausible latent vectors from.
- **AESS — Advanced Evolutionary Spread Search.** A population-based metaheuristic, extending Virus
  Spread Optimization, that searches the latent space. Its contribution is an adaptive
  exploration/exploitation controller driven by how long the best solution has survived, plus a
  redesigned exploratory update rule.
- **DeepADTP — Deep Attention Drug–Target Prediction.** A drug–target affinity model built from
  dilated causal convolutions with residual connections and **bidirectional attention** between the
  molecule and protein sequences. It provides the objective the search optimises.

In the integrated loop, AESS proposes a latent vector, the MAE decoder turns it into a molecule,
and DeepADTP scores it against the target protein (the original work used the SARS-CoV-2 3CL
protease); the score is fed back to steer the search.

## Results

DrugNet was evaluated on four fronts: molecular generation (MOSES metrics), drug–target affinity
(KIBA), a COVID-19 3CL protease target search, and Penalized LogP / QED property optimization.
Across these benchmarks its components compared favourably with the reported baselines, and the
AESS modifications improved on unmodified VSO under matched conditions.

The exact figures — and, importantly, the caveats that qualify them — are in the thesis. These are
in-silico results: the generated molecules score highly under a learned surrogate objective, not
under any experimental measurement of binding or activity. The thesis states each result at the
strength the evidence supports and is explicit about where the search dynamics themselves shape how
a headline number should be read.

## Repository layout

- `drugnet.pdf` — the technical thesis / preprint
- `drugnet.tex`, `references.bib`, `figures/` — LaTeX source; build with `latexmk -pdf drugnet.tex`
- `supplementary/verify_parameter_counts.py` — recomputes every reported model size analytically
  (standard library only)
- `supplementary/aess_reference.py` — a corrected, runnable reference implementation of the search
  algorithm, with a self-test covering its key invariants (requires NumPy)
- `DrugNet_Main_Document.ipynb` — the original implementation notebook

## Reproducibility

The original notebook establishes the architecture but is not a one-click reproduction: it loads
preprocessed data and pretrained weights from external paths, and several cells carry the kind of
syntax, argument-order, and stale-output issues typical of an edited notebook. Appendix C of the
thesis documents each of these individually, and the corrected reference implementation in
`supplementary/` shows how the search operators are intended to behave. Anyone reproducing the work
should start there rather than from the notebook as written.

## Authors

Ali Kılıç and Arda Cigizoglu.

DrugNet (originally *İlaçNet: Derin Öğrenmeyle Hızlı İlaç Üretimi*) was developed as an independent
research project in 2020–2021 and placed in the top five of its category at Teknofest. 
