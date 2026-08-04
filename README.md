# Multiplying flow and pressure — data and interactive review tool

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21786466.svg)](https://doi.org/10.5281/zenodo.21786466)

![Graphical abstract. Neither flow nor airway pressure alone allows detection of ventilation
cycle timing, because artefacts introduced by chest compression and recoil complicate
measurement of intra-arrest ventilation. An inspiratory product (flow × pressure) and an
expiratory product (flow × pressure slope) allow precise detection: F1 score 0.971 for
detection of respiratory phase onset during intra-arrest ventilation, in a porcine model
(n = 13) with 20 intra-arrest ventilations each.](graphical-abstract.svg)

<sub>Graphical abstract designed by [David Purkarthofer](https://github.com/dpurkarthofer).</sub>

Data and an interactive review interface accompanying:

> Orlob S, Purkarthofer D, Grobbel M, Holler M, Furtmüller M, Wittig J, Kern WJ, Martini J,
> Gräsner JT, Putzer G, Wnent J, Hackl B.
> **Multiplying flow and pressure: detecting respiratory phases in intra-arrest ventilation.**
> *Resuscitation*, 2026;222:111050.
> [doi:10.1016/j.resuscitation.2026.111050](https://doi.org/10.1016/j.resuscitation.2026.111050)

## The study in brief

Detecting when a ventilation begins and ends is straightforward while a patient is
ventilated normally, and hard once chest compressions are running: compression and recoil
push air in and out of the lungs, so neither airflow nor airway pressure alone identifies
respiratory phases. The article proposes an algorithm built on two products — **flow ×
airway pressure** for inspiration, and **flow × the slope of airway pressure** for
expiration — which separates airflow caused by ventilation from airflow caused by chest
compressions, and yields explicit onsets rather than an approximate timing.

The algorithm was validated against investigator-established ground truth on ventilatory
recordings from **13 pigs**. For each animal, 20 ventilations before induction of cardiac
arrest (**regular ventilation**) and 20 ventilations delivered asynchronously during
ongoing mechanical chest compressions (**intra-arrest ventilation**) were annotated —
606 s of regular and 1420 s of intra-arrest ventilation in total.

Classification of inspiratory and expiratory phases was perfect under both conditions. For
the exact timestamp of phase onset, the algorithm reached an F1 score of 1 during regular
ventilation and 0.971 during intra-arrest ventilation.

Full methods and results are in the article; this repository holds the underlying records
so the annotations can be inspected breath by breath.

## How the data were produced

The recordings come from experiments conducted between **November 2022 and January 2023**
at the experimental anaesthesia laboratory of the Medical University of Innsbruck, one
animal per experiment day — which is why each file is named by its date.

Seven animals were excluded: five for ventilator failure, one for ventilation with PEEP,
and one for a missing airway pressure recording. The **13 cases** published here are the
remainder.

**Ethical approval.** The Austrian Ministry of Education, Science and Research authorised
the experiments (BMBWF-V/3b GZ:2021-0.895.386). The experiments complied with Austrian and
European regulations concerning the use of laboratory animals.

**Measurements.** Airflow was recorded at 200 Hz with a mass flow meter (SFM3000, Sensirion
AG, Staefa, Switzerland) and airway pressure at 500 Hz with a differential pressure sensor
(DLVR-L60D, All Sensors Corporation, Morgan Hill, California), each connected to a
Raspberry Pi 3 B+. Ventilations were delivered mechanically with an *Evita XL* (Dräger,
Lübeck, Germany); mechanical chest compressions with a *Lucas 3* (Stryker, Kalamazoo,
Michigan). Volume-controlled ventilation before arrest was adapted to maintain a
p<sub>a</sub>CO<sub>2</sub> of 35–40 mmHg. Intra-arrest ventilation was protocolised at a
tidal volume of 10 mL/kg body weight and a rate of 10 min⁻¹, with zero PEEP and a variable
I:E ratio.

**Ground truth.** Two investigators (DP and SO) reviewed automatically generated prelabels
of inspiration and expiration onsets, corrected them where necessary and supplemented
missing events. Annotation and visualisation were carried out with the Python library
[`vitabel`](https://github.com/UniGrazMath/vitabel)
([doi:10.5281/zenodo.15771826](https://doi.org/10.5281/zenodo.15771826)).

## What is here

```
data/                        13 cases, one JSON per experiment day (CC BY 4.0)
review_ground_truth.ipynb    interactive review interface (MIT)
environment.yml              conda environment for Binder / local use
voila.json                   Voilà configuration
```

## Data format

Each file in `data/` is one experiment day, named by date (`YYMMDD`). Files are serialised
[`vitabel`](https://github.com/UniGrazMath/vitabel) `Vitals` objects (written with vitabel
0.1.0) and hold time-series **channels** plus annotation **labels**.

Every case contains **two recording segments** — one for regular ventilation, one for
intra-arrest ventilation — each anchored to its own wall-clock start time. All four signals
therefore appear twice per file, once per segment.

| Channel | Meaning |
|---|---|
| `Flow Interpolated` | airway flow |
| `Pressure Interpolated` | airway pressure |
| `Product Flow Pressure` | inspiratory product, flow × pressure |
| `Product negative Flow Pressures Slope` | expiratory product, flow × pressure slope |

**Labels** — the experimental phase (`baseline`, `cpr`, as interval labels spanning the two
segments), the investigator-established ground truth (`Inspiration Begin gt`,
`Expiration Begin gt` as onsets; `Inspiration gt`, `Expiration gt` as intervals), and the
onsets detected by the algorithm (`Inspiration Begin`, `Expiration Begin`). Each case
carries 21 inspiration onsets and 20 expiration onsets per phase.

Where it occurred, a case additionally carries `pressure limited inspiration`, marking
inspirations that were prematurely terminated by pressure limitation.

## Running the review interface

The notebook renders as a Voilà app: pick a case from the dropdown to see flow, pressure
and the two products in synchronised subplots, with the ground-truth and algorithm
annotations drawn over them and a clickable overview of the full recording beneath for
navigation. Press `+` / `-` to zoom the time axis, use the arrow keys to pan, and scroll
the mouse wheel over a subplot to rescale that trace vertically.

**In your browser, nothing to install:**

[![Launch the review interface](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/zenodo/10.5281/zenodo.21786466/?urlpath=%2Fvoila%2Frender%2Freview_ground_truth.ipynb)

This runs on [Binder](https://mybinder.org/) from the archived Zenodo release rather than
from this branch, so it shows released data whatever happens here later. The first start
takes a few minutes while the environment is built, and Voilà then executes the notebook
before the interface appears — allow about a minute more. To read or modify the code
instead, replace `voila/render` in the URL with `doc/tree`.

The article cites a Binder link that builds from this repository's default branch instead
of from the archive. It continues to work.

Locally:

```bash
conda env create -f environment.yml
conda activate binder-environment
voila review_ground_truth.ipynb
```

The environment pins `vitabel==0.1.1`, the version available when the article was
published.

## Citing

If you use these data, please cite the article above. Attribution is not merely a
courtesy here: the data are licensed under CC BY 4.0, which requires it.
[CITATION.cff](CITATION.cff) carries the citation in machine-readable form.

The dataset itself is archived on Zenodo and carries its own DOI:

- **All versions:** [10.5281/zenodo.21786466](https://doi.org/10.5281/zenodo.21786466) —
  use this one unless you need to pin an exact snapshot
- **v1.0.0:** [10.5281/zenodo.21786467](https://doi.org/10.5281/zenodo.21786467)

## Acknowledgements

[David Purkarthofer](https://github.com/dpurkarthofer) designed the graphical abstract, is
a co-author of the study, and established the ground-truth annotations together with the
first author.

## Licence

Two licences apply, because this repository holds two different kinds of thing.

**The recordings in [`data/`](data/) are licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)** — see
[LICENSE-CC-BY-4.0](LICENSE-CC-BY-4.0). You may share and adapt them for any purpose,
including commercially, provided you give appropriate credit, link to the licence, and
indicate whether you made changes. Crediting the article above satisfies this.

**The review notebook and the configuration files are licensed under the MIT licence** —
see [LICENSE](LICENSE). GitHub reports this repository as MIT because it detects only the
top-level licence file; the data are nonetheless CC BY 4.0 as stated here and in
[`data/LICENSE`](data/LICENSE).

The graphical abstract (`graphical-abstract.svg`) is reproduced from the article, which is
published open access under CC BY 4.0, and is covered by that licence.
