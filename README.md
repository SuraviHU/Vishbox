# VishBox

VishBox is a local, text-only research prototype for phishing-awareness training. It follows the paper’s manager-led workflow: configure a fictional scenario and synthetic cohort, run bounded simulated turns, analyze risk cues, and produce prevention guidance. It never calls people, sends messages, requests or stores credentials, initiates payments, or performs real-world actions.

## Paper and scope

The design is based on Yang et al., [“VishBox: An AI-Agent-Based Adaptive Voice Phishing Simulation Framework for Cybersecurity Education,” IEEE Access 14 (2026), 39672–39686](https://doi.org/10.1109/ACCESS.2026.3667823). The paper describes a manager-coordinated agent workflow, profiles based on demographic cohorts, digital financial literacy and OCEAN traits, three scenario families with four stages, composite turn-level risk, and personalized post-hoc prevention. It reports validation work that this student project has **not** reproduced.

The paper’s formula is `R̄ = mean(Vᵢ × Sᵢ)`, with persuasion intensity scaled to 0–1 and a classifier score. This implementation treats `Sᵢ` as the model’s positive phishing-class score. The paper also calls it “detection probability”; that interpretation and calibration must be resolved before treating the result as a validated human-risk measure. The current red-flag rules and cohort values are demonstrations only.

## Included features

- Streamlit configuration page for the three JSON scenario templates and six synthetic age cohorts.
- Manager orchestration across scenario stages with a deterministic offline caller and victim.
- Optional local KLUE-RoBERTa model loading, plus an explicitly unvalidated red-flag heuristic when no trained model exists.
- Turn-level composite score, conservative stop on sensitive-action cues or critical score, decision trace, and learner-side safety guidance.
- Watermarked synthetic content and a post-hoc Markdown prevention checklist.
- Opt-in SQLite storage for synthetic sessions and an aggregate-only dataset profile panel.
- A local JSON-RPC adapter for the app and an optional MCP SDK stdio server exposing scenario/profile discovery and one safe simulation turn.
- Docker Compose setup bound to localhost; raw data is mounted read-only.

The adaptive component changes the **learner’s safety guidance**, not an attacker’s persuasive capability. Live LLM calls are not active in this version. The paper’s attacker-side prompts are intentionally not reproduced as executable instructions.

## Datasets and model limitations

`data/raw/KoBERT_dataset_v3.0.csv` has `Transcript` and `Label` fields. The label mapping, source documentation, and license have not been confirmed. Do not map either class to phishing until you verify the dataset record.

The Korean Post Office fraud-account CSV is handled as potentially sensitive case data. Its profile includes CSV row-width consistency because quoted values can contain delimiters. This project reports only aggregate schema metadata and does not use it to train the text classifier or expose row-level details.

The paper trained KLUE-RoBERTa using an FSS vishing-call dataset together with a conversational-speech dataset. Those training inputs and the paper’s evaluation set are not interchangeable with the two uploaded CSVs. Training/evaluation results from this repository cannot be presented as a reproduction of the paper.

Inspect row counts, label distribution, encoding, and row-width consistency without printing transcript or case rows:

```powershell
.\.venv\Scripts\python.exe -m src.data_tools
```

Profile values in `config/victim_profiles.json` are marked `illustrative_unvalidated`. They must not be used to infer an individual’s susceptibility. A random row split can also leak similar transcripts across train and holdout sets; use source-aware grouping for research-grade evaluation.

## Run locally

Requires Python 3.10 or newer.

```powershell
cd vishbox_project
py -3.11 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\streamlit.exe run app\main.py
```

Open the local Streamlit URL shown in the terminal. The UI runs without API keys and without sending dialogue to a model provider. Save a session only when you select the save button; SQLite records stay under `data/processed/`.

To run the optional MCP SDK server over stdio:

```powershell
.\.venv\Scripts\python.exe -m src.mcp_server.standard_server
```

For a local container:

```powershell
docker compose up --build
```

The UI binds to `127.0.0.1:8501`. Review the Compose volume mounts before starting it on a machine that should not expose the uploaded raw data to the container.

## Train the optional text classifier

Install the larger ML dependencies only after verifying dataset license, label semantics, and approved use:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements-ml.txt
.\.venv\Scripts\python.exe -m src.train_classifier data\raw\KoBERT_dataset_v3.0.csv --negative-label 0 --positive-label 1 --approved-data-use
```

The positive/negative mapping above is an example, **not a claim about the uploaded data**. Use it only if source documentation confirms it. Training downloads the base model and saves local weights under `models/`; the app can load them with `VISHBOX_MODEL_PATH`. Record class mapping, data version, split method, and holdout metrics in any research report. Model output remains unvalidated until evaluated on a suitable independent vishing test set.

## Research roadmap

1. Confirm dataset documentation, license, label mapping, and provenance; clean the account CSV schema and keep sensitive fields excluded.
2. Replace illustrative profiles with documented, aggregate calibration inputs and evaluate demographic fairness without deterministic individual profiling.
3. Evaluate the text classifier on a permitted, independent vishing corpus with group/time-aware splits, calibration, class-wise metrics, and human review.
4. Run a research-ethics-approved human study with consent, debriefing, and outcome measures before claiming educational effectiveness.
5. Explore privacy-reviewed voice/sentiment features, cross-language calibration, and expanded institutional scam patterns, as proposed in the paper. Keep voice generation and real-world delivery disabled in the awareness product.

## Project layout

```text
config/                 scenarios and illustrative cohorts
data/raw/               supplied source datasets
data/processed/         local SQLite and future derived artifacts
src/agents/             manager, simulated caller and victim
src/analytics/          classifier adapter, risk, learner guidance
src/mcp_server/         local adapter and optional MCP stdio server
src/utils/              local database and prevention report
src/train_classifier.py guarded KLUE-RoBERTa fine-tuning entry point
app/                    Streamlit UI
tests/                  local unit tests
```

The existing local MCP adapter check can be run with `.\.venv\Scripts\python.exe -m pytest tests\test_mcp.py`. Risk and classifier evaluation still need a documented, authorized dataset mapping before meaningful research claims can be made.


