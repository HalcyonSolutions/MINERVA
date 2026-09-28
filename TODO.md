# TODO

Completed items are kept here for project history rather than removed.

## Core Implementation

- [ ] PyTorch implementation
- [x] Add weighted sampling over hop counts during training only
- [ ] Add an inference method that outputs JSON with predicted paths

## Data Loading & Dataset Support

- [x] Add `Path-Key` support to data loading
- [x] Detect multi-answer QA implicitly from the data instead of manually triggering it through options
- [ ] Add PathQuestion (PQ) and PathQuestionLarge (PQL) processing scripts

## Evaluation & Metrics

- [x] Calculate model/action entropy as an evaluation metric
- [x] Add per-hop answer accuracy
- [x] Rename metric functions and logged/W&B metric names to match the paper terminology
- [ ] Verify that moving semantic path expansion/navigation logic from the environment to the grapher preserves the same multi-answer results

## Documentation & Public Release

- [x] Move excess README information into the `docs/` folder
- [x] Update the README to reflect the current codebase
- [x] Update `docs/metrics.md` to reflect the current implementation and the *Theseus in the Graph* paper
- [x] Rename functions and variables to match the paper
- [x] Update README and docs
- [ ] Update the blind-evaluation repository

## Validation & Reproducibility

- [ ] Test the repository from scratch
