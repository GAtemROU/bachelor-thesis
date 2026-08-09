# Understanding Pragmatic Reasoning Through Eye Movement Patterns in Reference Games

See final thesis PDF [main.pdf](tex/thesis/main.pdf) for the full argument, methods, models, and discussion.

This repository contains the experiment, analysis code, trial-generation utilities, and thesis sources for a bachelor thesis on using webcam-based eye tracking to study pragmatic reasoning in reference games.

The project asks whether participants' visual attention patterns reveal how they solve reference games: communicative tasks where a listener must infer which object a speaker intended to refer to from a possibly ambiguous message. The study combines a LingoTurk web experiment, WebGazer-based gaze tracking, participant strategy reports, and statistical analysis of gaze-derived features.

## Project overview

Reference games are used to study pragmatic inference: people often infer meaning by reasoning about what a cooperative speaker could have said but did not say. This project investigates that reasoning process through eye movements.

The experiment presents participants with three colored shapes, one message sent by a previous participant, and a set of messages that were available to the speaker. Participants click the object they think the speaker intended. The trials have three conditions:

- `unambiguous`: the sent message uniquely identifies the target.
- `simple`: the sent message is ambiguous, but first-order pragmatic reasoning is sufficient.
- `complex`: the trial requires deeper reasoning and attention to the distractor structure.

The study used webcam-based eye tracking via WebGazer during the task. The main analysis converts raw gaze predictions into areas of interest and tests whether proportional time spent on the target, competitor, distractor, sent message, available-message bank, and non-AOI regions predicts accuracy and strategy.

## Main findings summarized in the thesis

- 101 participants remained after exclusions.
- Participants solved unambiguous trials most accurately, followed by simple trials, then complex trials.
- Proportional time on the available-message bank was positively associated with accuracy, especially for simple trials.
- Proportional time on the distractor was not a reliable positive predictor of complex-trial accuracy.
- Correct participants tended to use similar attention profiles for simple and complex trials, while unambiguous trials showed a different, simpler profile.
- The results suggest that failures are less about missing the distractor entirely and more about how participants interpret and reason over the available information.

For the full argument, methods, models, and discussion, see `tex/thesis/main.pdf`.

## Repository layout

- `RefGameShapesGazeFeedbackExperiment/`  
  Exported LingoTurk experiment files. This includes the Angular experiment renderer, WebGazer integration, calibration scripts, CSS, stimuli images, LingoTurk question metadata, and the SQL query used to export results.

- `analysis/`  
  Data processing, exploratory notebooks, regression scripts, scanpath experiments, plots, and derived data.
  - `analysis/preprocessing_and_init_analysis.ipynb`: main preprocessing and initial analysis notebook.
  - `analysis/plots.ipynb` and `analysis/plots.r`: plotting and participant-level summaries.
  - `analysis/regressions/MLE/`: maximum-likelihood logistic regression models.
  - `analysis/regressions/Bayesian/`: Bayesian model variants and related output.
  - `analysis/scanpath_classification/`: exploratory scanpath and strategy-classification code.
  - `analysis/data/`: raw, preprocessed, and final analysis datasets.

- `trials/`  
  Python utilities for generating the reference-game trial lists.
  - `gen_trials.py`: generates simple, complex, and unambiguous trial structures.
  - `gen_inputdata.py`: writes LingoTurk input CSV files.
  - `inputdata.csv` and `inputdata_simplified.csv`: generated trial input files.

- `tex/`  
  LaTeX sources and compiled PDFs for the thesis, paper version, and ethics description.
  - `tex/thesis/main.pdf`: compiled thesis.
  - `tex/thesis/sections/`: thesis sections.
  - `tex/paper/`: paper-format version.

- `webgazer_test/`  
  Small local utilities used while testing WebGazer/click behavior.

- `requirements.txt`  
  Python dependencies used by notebooks, analysis utilities, and exploratory modeling.

- `createZip.sh`  
  Rebuilds `RefGameShapesGazeFeedbackExperiment.zip` from the exported experiment directory.

- `importChanges.sh`  
  Copies the current experiment implementation from a sibling LingoTurk checkout into this repository.

## Setup

Clone the repository and initialize submodules:

```bash
git clone <repo-url>
cd Thesis-Project
git submodule update --init --recursive
```

Create a Python environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

The pinned requirements include CUDA builds of PyTorch (`torch==2.6.0+cu118`, `torchvision==0.21.0+cu118`, `torchaudio==2.6.0+cu118`). If pip cannot resolve those wheels from the default index, install the appropriate PyTorch build for your machine from the PyTorch wheel index, or replace them with CPU-only builds if GPU support is not needed.

Some analyses are written in R. The R scripts use packages including:

- `tidyverse`
- `lme4`
- `scales`
- `emmeans`
- `magrittr`
- `GGally`
- `saccades` for the exploratory fixation-detection script

Install them from R as needed:

```r
install.packages(c("tidyverse", "lme4", "scales", "emmeans", "magrittr", "GGally"))
```

## Working with trial lists

Trial generation is controlled by `trials/gen_trials.py` and `trials/gen_inputdata.py`.

To regenerate the input CSV from the repository root:

```bash
python trials/gen_inputdata.py
```

`gen_inputdata.py` currently writes either `trials/inputdata_simplified.csv` or `trials/inputdata.csv` depending on the `gen_simplified` flag in the script.

The generated columns match the LingoTurk question fields in `RefGameShapesGazeFeedbackExperiment/app__models__Questions/RefGameShapesGazeFeedbackExperiment/fields.json`:

- `trialid`
- `type`
- `sentmsg`
- `trgt`
- `comp`
- `dist`
- `msg1` through `msg4`

## Experiment package

The experiment is stored in `RefGameShapesGazeFeedbackExperiment/` in the directory structure expected by the project’s LingoTurk import/export workflow.

Important files:

- `app__views__ExperimentRendering/RefGameShapesGazeFeedbackExperiment/RefGameShapesGazeFeedbackExperiment_render.html`: experiment UI, instructions, calibration screens, trial screens, and strategy questions.
- `public__javascripts__ExperimentRendering/RefGameShapesGazeFeedbackExperiment/RefGameShapesGazeFeedbackExperiment_render.js`: Angular controller, trial preparation, WebGazer data collection, feedback, and result submission.
- `public__javascripts/`: WebGazer and calibration-related scripts.
- `public__images__Experiments/RefGameShapesGazeFeedbackExperiment/`: shape/color stimuli and instruction images.
- `app__models__Questions/RefGameShapesGazeFeedbackExperiment/resultQuery.sql`: SQL query for exporting experiment results from LingoTurk.

To rebuild the distributable ZIP:

```bash
./createZip.sh
```

This creates or replaces `RefGameShapesGazeFeedbackExperiment.zip`.

To import the latest implementation from a sibling LingoTurk checkout:

```bash
./importChanges.sh
```

`importChanges.sh` assumes the following directory layout:

```text
parent-directory/
  Lingoturk/
    lingoturk/
  Thesis-Project/
```

## Analysis workflow

The analysis is organized around datasets in `analysis/data/` and scripts/notebooks in `analysis/`.

Typical workflow:

1. Use `analysis/preprocessing_and_init_analysis.ipynb` to parse exported LingoTurk results, assign gaze points to areas of interest, compute trial-level and gaze-point-level features, and create final datasets.
2. Use `analysis/plots.ipynb` or `analysis/plots.r` for exploratory summaries and thesis figures.
3. Run the regression scripts in `analysis/regressions/MLE/` for the main logistic models:
   - `analysis/regressions/MLE/per_trial/correct_answer.r`: predicts whether a trial was answered correctly.
   - `analysis/regressions/MLE/per_correct_fixation/on_dist.r`: predicts whether a gaze point lands on the distractor for correctly solved trials.
   - `analysis/regressions/MLE/per_correct_fixation/av_msgs.r`: predicts whether a gaze point lands on the available-message bank for correctly solved trials.
4. Use `analysis/scanpath_classification/` for exploratory scanpath and strategy-classification experiments.

The main final datasets referenced by the R scripts are:

- `analysis/data/final_datasets/final_experiment_trials.csv`
- `analysis/data/final_datasets/final_experiment_correct_fixations.csv`
- `analysis/data/final_datasets/final_experiment_participants_extended.csv`

Most R scripts assume they are run from the repository root.

Example:

```bash
Rscript analysis/regressions/MLE/per_trial/correct_answer.r
```

## Thesis sources

The compiled thesis is available at:

- `tex/thesis/main.pdf`

The source sections are under:

- `tex/thesis/sections/`

There is also a paper-format version under `tex/paper/`.

To rebuild the thesis, use your local LaTeX toolchain from `tex/thesis/`. For example:

```bash
cd tex/thesis
latexmk -pdf main.tex
```

## Notes on data and reproducibility

- The experiment collected webcam-derived gaze coordinates and pupil estimates, not webcam video or audio.
- The raw data can be downloaded [here](https://drive.google.com/drive/folders/1Kv3FhWDbCAJJyxSpgwgeYHD4gLgbQZg4?usp=sharing)
- The analysis uses derived gaze features rather than high-confidence lab-grade fixations because WebGazer sampling rates were low and variable.
- Some paths in exploratory scripts are absolute or user-specific; the main regression scripts generally use repository-relative paths and should be run from the repository root.
- Several notebooks are exploratory and may depend on intermediate files in `analysis/data/`.

## Author

Tymur Mykhalievskyi  
Bachelor Thesis, Computer Science (English)   
Saarland University, May 2025

Advisors:

- Prof. Dr. Vera Demberg
- Dr. John Duff

## Citation

If you use this repository or build on the experiment, cite the thesis:

> Mykhalievskyi, T. (2025). *Understanding Pragmatic Reasoning Through Eye Movement Patterns in Reference Games*. Bachelor thesis, Saarland University.
