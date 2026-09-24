# AI Intern Exercise — Predicting Gene-Editor Activity

**Time limit: five hours. AI tools allowed.**

Evaluate this proposal: combine components from pretrained protein sequence models and diffusion-based structure prediction models, replace the structure-prediction trunk with an activity-prediction component, and predict gene-editor activity.

Decide whether to pursue, modify or reject the proposal. Support your decision with a runnable experiment and a technically concrete architecture.

## Dataset

Use `CAS9_STRP1_Spencer_2017_positive` from the supplied data pack.

[Original study](https://www.nature.com/articles/s41598-017-17081-y) · [ProteinGym](https://github.com/OATML-Markslab/ProteinGym)

## Requirements

1. Determine what the measurements support predicting. Identify the dataset's most consequential limitations.
2. Design your own evaluation protocol. Justify the split, baselines and metrics against the generalization claim you intend to make.
3. Implement a predictor of the supplied scores and one substantive extension. State which part of your proposal the experiment tests. Produce held-out predictions, run an ablation, and run a separate experiment intended to challenge your preferred interpretation.
4. Specify the pretrained model versions, named modules, representation shapes, interfaces and training objective for your proposed architecture. Identify what you would retain, replace, freeze or train, and justify those decisions.
5. Make a recommendation based on the evidence. State what your experiment establishes, what remains untested and the next experiment that could change your recommendation.

Choose an implementation that fits the time limit and your available compute. Full foundation-model training is not required. Architectural proposals must be distinguished from executed experiments; conclusions about structural or diffusion features require evidence using those features.

## Deliverables

- A runnable repository or notebook, including dependencies, preprocessing, split assignments, held-out predictions and reproduction instructions.
- A report of no more than three pages containing the architecture, experimental results, failure analysis and recommendation.
- A brief record of time spent, compute used, AI tools used and unfinished work.

Stop at five hours. Be prepared to explain, debug or modify your implementation live.
