# Slide-of-Life

**A tool for finding hidden overlap between the training and test sets of computational-pathology datasets.**

Slide-of-Life looks for relationships that can make a model evaluation less trustworthy, such as the same patient, specimen, slide, or image appearing across both training and test data. It can also flag very similar images for review and suggest a cleaner split for a researcher to inspect.

## Project origin

Slide-of-Life began as my project for the **OpenAI DevDay hackathon**, developed with OpenAI Codex. This repository is a continuation of that project so I can keep improving and extending it without changing the original hackathon submission.

The original submission is preserved at [`ppandya6/Slide-of-Life`](https://github.com/ppandya6/Slide-of-Life). The published `slide-of-life` Python package, GitHub releases, release tags, and package-publishing setup remain tied to that original repository.

Codex helped with the engineering work behind the project, including the Python package and command-line interface, tests, CI, packaging, release setup, and documentation. GPT-5.6 can optionally help interpret unclear dataset column names, but the actual checks for overlap and duplication are performed locally by normal Python code. No OpenAI API key is required for the demo or for normal use.

## Quick start

The included demo uses only synthetic data and takes about two minutes. **Python 3.11 or newer is required.**

### Windows PowerShell

```powershell
py -0p
py -3.11 --version
```

Then clone this repository and run the demo:

```powershell
git clone https://github.com/ppandya6/Slide-of-Life-Portfolio.git
Set-Location Slide-of-Life-Portfolio
py -3.11 -m venv .venv
& .\.venv\Scripts\python.exe -m pip install --upgrade pip
& .\.venv\Scripts\python.exe -m pip install "slide-of-life==0.1.0a1"
& .\.venv\Scripts\python.exe scripts\generate_demo.py --force
& .\.venv\Scripts\slide-of-life.exe audit --train examples\demo\generated\train_manifest.csv --test examples\demo\generated\test_manifest.csv --images examples\demo\generated\images --schema-map examples\demo\schema-map.yaml --output artifacts\demo-audit --repair --force
```

The final command intentionally returns **exit code 2** because the demo contains problems that Slide-of-Life is supposed to find. This means the audit completed successfully and found policy violations; it does not mean the program crashed.

Open the generated report with:

```powershell
Invoke-Item artifacts\demo-audit\report.html
```

### macOS or Linux

```bash
git clone https://github.com/ppandya6/Slide-of-Life-Portfolio.git
cd Slide-of-Life-Portfolio
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

Then open:

```bash
python3 -m webbrowser artifacts/demo-audit/report.html
```

## What the demo does

The demo creates two small dataset manifests and a set of generated images. Slide-of-Life compares the training and test data, looks for overlap, applies the chosen split rules, and writes several files for review.

| Path | What it contains |
| --- | --- |
| `examples/demo/generated/train_manifest.csv` | Synthetic training records |
| `examples/demo/generated/test_manifest.csv` | Synthetic test records |
| `examples/demo/generated/images/` | Generated demo images |
| `artifacts/demo-audit/report.html` | Human-readable report |
| `artifacts/demo-audit/report.json` | Machine-readable report |
| `artifacts/demo-audit/findings.csv` | Table of detected issues |
| `artifacts/demo-audit/repair_proposal.csv` | Suggested reassignment of records for researcher review |

## What Slide-of-Life looks for

The demo includes examples of the main kinds of issues Slide-of-Life can detect:

| Relationship | Result | Meaning |
| --- | --- | --- |
| Patient overlap | Violation | The same patient appears in both train and test data |
| Specimen overlap | Violation | The same specimen appears in both sets |
| Slide overlap | Violation | The same slide appears in both sets |
| Exact file duplicate | Violation | Two differently named files contain the same bytes |
| Exact image duplicate | Violation | Two files store the same image pixels, even in different formats |
| Institution overlap | Allowed | Both sets contain data from the same institution, which the default rules allow |
| Similar-image candidate | Review | Two images are very similar and should be checked manually |
| Missing image | Review | A manifest points to an image that is not present |

## Why this matters

If related patients, specimens, slides, or images appear in both training and test data, a machine-learning model may be tested on information that is too similar to what it already saw during training. That can make performance look better than it really is.

Slide-of-Life is meant to make those relationships easier to find and inspect before a dataset is used for evaluation. It does not claim that a dataset is automatically safe or contamination-free.

## How it works

Slide-of-Life separates the process into three steps:

1. **Find relationships.** It checks identifiers and images for overlaps, duplicates, and other suspicious similarities.
2. **Apply split rules.** Each finding is marked as allowed, a violation, or something that needs human review.
3. **Suggest a repair.** If requested, Slide-of-Life can propose a new assignment of records, but the final decision is left to the researcher.

Exact duplicates are treated differently from images that are only similar. A similarity match is a reason to review the data, not proof that two records come from the same patient or specimen.

## Codex and GPT-5.6

### Codex

Codex assisted with the development of the project, including architecture, Python implementation, the command-line interface, tests, CI, packaging, release debugging, and documentation.

Codex helped build Slide-of-Life, but it is not required when the tool runs.

### GPT-5.6

GPT-5.6 is optional. It can help suggest what unclear dataset columns may represent when their names are ambiguous. Only column names and limited summary information are used for this feature; raw rows and images are not sent.

Any suggested mapping is checked by the program and must be accepted by the user before it is used.

### What runs locally

The actual work of reading the dataset, comparing identifiers, checking images, applying split rules, suggesting repairs, and creating reports runs locally in Python. GPT-5.6 does not decide whether an overlap exists or whether a record should be moved.

## Install the published package

The published alpha is released from the original [`ppandya6/Slide-of-Life`](https://github.com/ppandya6/Slide-of-Life) repository and requires Python 3.11+.

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

## Optional AI schema assistance

To install support for the optional GPT-assisted column-mapping feature:

```bash
python3 -m pip install "slide-of-life[ai]==0.1.0a1"
```

The feature only connects to an AI provider when it is explicitly requested. Store credentials in the `OPENAI_API_KEY` environment variable or a CI secret rather than placing them in code.

More detail is available in the [scientific method documentation](docs/scientific-method.md#task-10-optional-ai-schema-interpretation).

## Other ways to use it

- The [GitHub Action](action.yml) allows Slide-of-Life checks to run in CI.
- The [Slide-of-Life Agent Skill](skills/slide-of-life/SKILL.md) provides a guided way for an agent to run and interpret an audit.
- The [demo guide](examples/demo/README.md) explains the synthetic example in more detail.

The published GitHub Action release still comes from the original repository:

```yaml
- uses: actions/checkout@v4
- uses: ppandya6/Slide-of-Life@v0.1.0a1
  with:
    train-manifest: data/train.csv
    test-manifest: data/test.csv
    output-dir: slide-of-life-artifacts
```

## Limitations and safety

Slide-of-Life `0.1.0a1` is an alpha release. It is not a clinical device and does not make diagnoses, treatment recommendations, or biological conclusions.

Its results depend on the quality of the identifiers, images, and metadata it receives. Missing or unclear data can limit what it finds, so every finding and repair suggestion should still be reviewed by a person.

The included demo contains only fictional identifiers and generated images. Do not commit private clinical data or credentials to this repository.

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

This repository can continue to develop independently, while the published package and official releases remain tied to the original `ppandya6/Slide-of-Life` repository unless that changes later.

## License

Slide-of-Life is distributed under the [MIT License](LICENSE).
