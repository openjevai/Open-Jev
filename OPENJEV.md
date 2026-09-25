# OpenJEV Support

This fork adds optional [OpenJEV](https://openjev.sh) support alongside the
existing TypeSafe Jev API integration. Jev is built by
[TypeSafe](https://typesafe.ai); OpenJEV is a free community gateway to the
same Jev model. TypeSafe remains the default — anyone with a TypeSafe key sees
zero behaviour change.

## What was added

| File | Change |
|---|---|
| `scripts/benchmark_jev_api_latency.py` | `PROVIDERS` dict + `resolve_provider()` / `configure_provider()`. `--provider` flag (choices: `typesafe`, `openjev`). `JEV_PROVIDER` env override. Model/key/endpoint resolve from the chosen provider. HTTP 503 added to stop statuses. |
| `scripts/evaluate_trec_provider.py` | `openjev` added to `MODELS`. `jev_scores()` now takes the model id instead of hardcoding `jev-1.13.0`. `run()` configures the jev_api host for `openjev`. `main()` resolves `OPENJEV_API_KEY` for `--model openjev`. |
| `scripts/run_jevbench_hosted.py` | `openjev` provider entry in `PROVIDERS` (endpoint `api.openjev.sh`, model `openjev`, price 0). `--openjev-key-file` argument. `TypeSafeAdapter` used for both `jev` and `openjev`. Default providers unchanged (`jev`, `luna`, `astra`). |
| `jev/serving.py` | `openjev` accepted as a wire-compatibility model alias on the local server. |
| `jev/server.py` | `openjev` listed in `/v1/models` aliases. |
| `docs/provider-comparison.md` | Reproduce section documents the OpenJEV option. |
| `README.md` | Short OpenJEV note after the project intro. |

No TypeSafe code, endpoints, model ids, or defaults were removed or renamed.

## Provider selection rule

1. Explicit choice wins: `--provider openjev`, `JEV_PROVIDER=openjev`, or
   `--model openjev` (in `evaluate_trec_provider.py`).
2. Otherwise, if `TYPESAFE_API_KEY` is set → TypeSafe (unchanged default).
3. Otherwise, if only `OPENJEV_API_KEY` is set → OpenJEV.

| | TypeSafe (default) | OpenJEV |
|---|---|---|
| Endpoint | `https://api.typesafe.ai/v1/systemone` | `https://api.openjev.sh/v1/systemone` |
| Model | `jev-1.13.0` | `openjev` |
| Key env | `TYPESAFE_API_KEY` | `OPENJEV_API_KEY` (from https://openjev.sh/dashboard) |
| Overload status | 529 | 503 (also 429) |

## How to configure

```bash
# TypeSafe (unchanged):
export TYPESAFE_API_KEY=ts_...
python -m scripts.benchmark_jev_api_latency --requests … --output …

# OpenJEV (explicit):
export OPENJEV_API_KEY=ojev_...
python -m scripts.benchmark_jev_api_latency --provider openjev --requests … --output …

# OpenJEV (implicit, no TypeSafe key set):
export OPENJEV_API_KEY=ojev_...
python -m scripts.benchmark_jev_api_latency --requests … --output …

# TREC provider comparison with OpenJEV:
python -m scripts.evaluate_trec_provider --model openjev --input … --output …

# JevBench hosted with OpenJEV:
python -m scripts.run_jevbench_hosted --providers openjev --openjev-key-file /path/to/key …
```

## How it was verified

A live POST to `https://api.openjev.sh/v1/systemone` with the OpenJEV key,
model `openjev`, state `ping`, and one noul question returned HTTP 200.
No repository code was executed during this port (read and edit only).

## Upstream

Original project: https://github.com/Zefan-Cai/Open-Jev by @Zefan-Cai.
