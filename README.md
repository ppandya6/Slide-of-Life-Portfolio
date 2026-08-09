# Slide-of-Life

**A local, deterministic-first audit for hidden relationships between the train
and test partitions of computational-pathology datasets.**

Slide-of-Life finds biological linkages in train/test splits, exact image
duplication, and image-similarity candidates before hidden overlap can skew a
research evaluation. It evaluates the identified factual evidence against a
named split policy and can create a review-required repair proposal.

## Project origin and repository relationship

Slide-of-Life began as my project for the **OpenAI DevDay hackathon**, developed
with OpenAI Codex. This repository is an ongoing extension of that work so the
project can continue to evolve beyond the submitted hackathon version.

The original hackathon repository is preserved at
[`ppandya6/Slide-of-Life`](https://github.com/ppandya6/Slide-of-Life). The
published `slide-of-life` package, GitHub releases, release tags, and publication
infrastructure remain associated with that original repository. This continuation
repository does not replace the original publication source.

Codex assisted with the engineering workflow, including package and CLI
implementation, tests, CI, release infrastructure, and documentation. Optional
GPT-5.6 schema assistance is available, but the core audit pipeline remains local
and deterministic. No OpenAI API key is required for the demo or for normal
deterministic audits.

## Quick start

The demo is entirely synthetic, deterministic, and small. Allow about two
minutes. **Python 3.11 or newer is required.**

### Windows PowerShell

First confirm that the Windows Python launcher can find Python 3.11:

```powershell
py -0p
py -3.11 --version
```

If the second command fails, install Python 3.11 or newer from
[python.org](https://www.python.org/downloads/windows/) and enable the Python
launcher. Then open a new PowerShell window.

Clone this continuation repository and run the published package against the
included synthetic demo:

```powershell
git clone https://github.com/ppandya6/Slide-of-Life-Portfolio.git
Set-Location Slide-of-Life-Portfolio
py -3.11 -m venv .venv
& .\.venv\Scripts\python.exe -m pip install --upgrade pip
& .\.venv\Scripts\python.exe -m pip install "slide-of-life==0.1.0a1"
& .\.venv\Scripts\python.exe scripts\generate_demo.py --force
& .\.venv\Scripts\slide-of-life.exe audit --train examples\demo\generated\train_manifest.csv --test examples\demo\generated\test_manifest.csv --images examples\demo\generated\images --schema-map examples\demo\schema-map.yaml --output artifacts\demo-audit --repair --force
```

The final command intentionally returns **exit code 2** because the synthetic
data contains policy violations. That means the audit succeeded and wrote its
reports; it is not a crash. Confirm the result and open the report:

```powershell
$LASTEXITCODE                 # expected: 2
Get-ChildItem artifacts\demo-audit
Invoke-Item artifacts\demo-audit\report.html
```

> If `py -3.11` is unavailable but `py -3.12` is listed by `py -0p`, substitute
> `py -3.12` in the commands that create/check the environment. If `git` is
> unavailable, download the repository ZIP from GitHub, extract it, and begin
> with the virtual-environment command.

### macOS or Linux

Use a Python 3.11+ interpreter. On systems where it is named `python3`, run:

```bash
git clone https://github.com/ppandya6/Slide-of-Life-Portfolio.git
cd Slide-of-Life-Portfolio
python3 --version
python3 -m venv .venv
.venv/bin/python -m pip install --upgrade pip
.venv/bin/python -m pip install "slide-of-life==0.1.0a1"
.venv/bin/python scripts/generate_demo.py --force
.venv/bin/slide-of-life audit \
  --train examples/demo/generated/train_manifest.csv \
  --test examples/demo/generated/test_manifest.csv \
  --images examples/demo/generated/images \
  --schema-map examples/demo/schema-map.yaml \
  --output artifacts/demo-audit \
  --repair \
  --force
```

Exit code **2 is expected**. Open the standalone report with:

```bash
python3 -m webbrowser artifacts/demo-audit/report.html
```

If the browser command is unavailable, open
`artifacts/demo-audit/report.html` from Finder or your file manager.

## What the demo does

The generator creates two six-row manifests and eleven tiny generated images;
one twelfth image path is intentionally missing. The audit compares train and
test records, evaluates the evidence under the default policy, proposes a
replacement partition, and writes four files.

| Path | What it contains |
| --- | --- |
| `examples/demo/generated/train_manifest.csv` | Six synthetic training records |
| `examples/demo/generated/test_manifest.csv` | Six synthetic test records |
| `examples/demo/generated/images/` | Eleven deterministic 64×64 generated images |
| `artifacts/demo-audit/report.html` | Standalone, human-readable report |
| `artifacts/demo-audit/report.json` | Full typed, machine-readable audit report |
| `artifacts/demo-audit/findings.csv` | Policy-evaluated findings table |
| `artifacts/demo-audit/repair_proposal.csv` | Proposed assignment for every input record; researcher review required |

The HTML is self-contained and does not need a server or internet connection.
Rerunning with `--force` replaces the known generated demo/output files.

## Expected demo findings

The report contains **eight deliberately planted relationship categories**:

| Relationship | Expected policy result | Meaning |
| --- | --- | --- |
| Confirmed patient overlap | Violation | A canonical patient identifier crosses partitions |
| Confirmed specimen overlap | Violation | A canonical specimen identifier crosses partitions |
| Confirmed slide overlap | Violation | A canonical slide identifier crosses partitions |
| Confirmed byte-content duplicate | Violation | Differently named files have identical bytes |
| Confirmed pixel-content duplicate | Violation | PNG and BMP files decode to identical pixels |
| Institution overlap | Allowed | Shared institution provenance is recorded but permitted by the default policy |
| Image-similarity candidate | Review | Two generated images differ by one synthetic pixel; this is not identity evidence |
| Missing-image input quality | Review | One manifest path intentionally has no image |

The five confirmed cross-partition overlaps/duplicates are independently planted
and are repair-eligible under the default policy. Similarity is kept separate
from lineage identity, and the repair CSV is only a proposal, not proof that a
split is correct.

## Why contamination matters

If related patients, specimens, slides, crops, re-encodings, or derived images
occur in both training and testing data, a model can effectively be evaluated on
information it has already seen. This can inflate reported performance and
undermine benchmarks, papers, and product claims. Filenames and ordinary file
hashes alone are insufficient: identifiers can express lineage, and the same
decoded pixels can be stored in different file formats.

Slide-of-Life makes that evidence inspectable without making biological or
clinical claims. It does not certify that a dataset is contamination-free.

## How the pipeline stays scientifically clear

Slide-of-Life keeps three concepts separate:

1. **Factual detection:** deterministic detectors emit evidence with source and
   row provenance.
2. **Policy evaluation:** an explicit `SplitPolicy` marks each factual finding as
   allowed, a violation, or requiring review, with a deterministic reason.
3. **Repair proposal:** an optional deterministic assignment responds only to
   eligible confirmed findings and always requires researcher review.

Exact byte/pixel relationships are distinct from perceptual similarity.
Similarity never establishes patient, specimen, slide, or institution identity.

## Codex, GPT-5.6, and deterministic code

### How Codex was used

Codex assisted the engineering workflow: architecture and implementation of the
Python package and CLI, tests, cross-platform CI, GitHub Action and Agent Skill,
release debugging, packaging, and documentation. Codex helped build the tool; it
is not an inference service used by the deterministic audit pipeline.

### How GPT-5.6 is used

GPT-5.6 is an **optional, explicitly enabled schema assistant**. For unresolved
manifest columns, it can propose semantic column mappings using headers and
privacy-bounded aggregate statistics. Slide-of-Life records model/provider
provenance, validates proposals deterministically, and requires explicit user
acceptance before applying a validated mapping. It sends no raw rows or images.

### What remains deterministic

In both normal use and the demo, local deterministic code performs ingestion,
canonicalization, factual relationship detection, image fingerprinting, policy
evaluation, repair proposal generation, and report serialization. GPT-5.6 never
creates findings, decides policy outcomes, or makes repair decisions. The demo
does not contact OpenAI and needs no credential.

## Install the published package

The published alpha is released from the original
[`ppandya6/Slide-of-Life`](https://github.com/ppandya6/Slide-of-Life) repository
and requires Python 3.11+:

```bash
python3 -m pip install "slide-of-life==0.1.0a1"
slide-of-life --version
slide-of-life --help
```

On Windows:

```powershell
py -3.11 -m pip install "slide-of-life==0.1.0a1"
slide-of-life --help
```

The installed interface is the `slide-of-life` executable, not a
`python -m slide_of_life` module. The deprecated `slidelineage` executable
remains only as a compatibility alias.

## Optional AI schema assistance

AI is unnecessary for explicit schema maps such as the demo's. To make the
optional feature available, install:

```bash
python3 -m pip install "slide-of-life[ai]==0.1.0a1"
```

Provider access happens only when AI schema mapping is explicitly requested.
Never paste or commit an API key; use the `OPENAI_API_KEY` environment variable
or a CI secret. See the
[scientific method](docs/scientific-method.md#task-10-optional-ai-schema-interpretation)
for the privacy boundary and acceptance workflow.

## Other ways to run it

- The [GitHub Action](action.yml) invokes the same CLI for policy-aware CI, writes
  reports before returning a policy-violation status, and keeps AI off by default.
- The [Slide-of-Life Agent Skill](skills/slide-of-life/SKILL.md) wraps the CLI for
  safe preflight, execution, and artifact interpretation.
- The [detailed demo guide](examples/demo/README.md) documents fixture semantics,
  cleanup, and custom generation destinations.

The published GitHub Action release is still sourced from the original repository:

```yaml
- uses: actions/checkout@v4
- uses: ppandya6/Slide-of-Life@v0.1.0a1
  with:
    train-manifest: data/train.csv
    test-manifest: data/test.csv
    output-dir: slide-of-life-artifacts
```

## Limitations and safety

Slide-of-Life `0.1.0a1` is an alpha prerelease, not a clinical device or
regulatory compliance product. Audit quality depends on the supplied manifests,
accepted identifiers, images, and schema mapping. Missing or ambiguous data
limits what can be detected. Every finding and repair proposal requires human
review.

The demo contains only fictional identifiers and generated pixels. Slide-of-Life
makes no diagnosis, prognosis, treatment, biological interpretation, or clinical
claim. Do not commit raw clinical data or credentials to this repository.

## Project links

- **Continuation repository:** [ppandya6/Slide-of-Life-Portfolio](https://github.com/ppandya6/Slide-of-Life-Portfolio)
- **Original OpenAI DevDay hackathon repository:** [ppandya6/Slide-of-Life](https://github.com/ppandya6/Slide-of-Life)
- **Published package:** [PyPI `slide-of-life` 0.1.0a1](https://pypi.org/project/slide-of-life/0.1.0a1/)
- **Original repository release:** [`v0.1.0a1`](https://github.com/ppandya6/Slide-of-Life/releases/tag/v0.1.0a1)
- [Changelog](CHANGELOG.md)
- [Scientific method](docs/scientific-method.md)
- [Product specification](docs/product-spec.md)

## Development

```bash
python -m pip install -e ".[dev]"
ruff format --check .
ruff check .
mypy src
pytest -q
```

This repository can continue development independently, but package publication,
release tags, and the canonical published release remain associated with the
original `ppandya6/Slide-of-Life` repository unless that relationship is
explicitly changed in the future.

## License

Slide-of-Life is distributed under the [MIT License](LICENSE).
