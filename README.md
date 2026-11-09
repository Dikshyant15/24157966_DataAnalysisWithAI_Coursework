# IMF Maldives 2024 Article IV --- LLM-Assisted Financial Report Analysis

An academic research project analysing the **IMF Maldives 2024 Article
IV Consultation** (IMF Country Report No. 24/106, May 2024) using large
language models (LLMs), retrieval-augmented generation (RAG), structured
financial data extraction, and visual analytics.

The project combines **PyMuPDF**, **Google Gemini**, **Sentence
Transformers**, **FAISS**, **Pandas**, **Matplotlib**, and **Seaborn**
in a Google Colab notebook.

> **Research disclaimer:** This project is an educational analysis of a
> public IMF report. LLM outputs, automated evaluation scores, and
> economic projections require verification against the original report
> and should not be treated as financial advice.

## Project objectives

1.  Extract and clean text from the IMF PDF, create token-aware chunks,
    and generate executive and section summaries.
2.  Identify seven financial-stability themes and extract quantitative
    indicators and qualitative findings.
3.  Build a local embedding-based FAISS index for source-grounded
    question answering.
4.  Compare single-query and multi-query retrieval, and visualise
    historical, estimated, and projected economic indicators.

## Repository contents

  ---------------------------------------------------------------------------------------------------
  File or directory                                               Purpose
  --------------------------------------------------------------- -----------------------------------
  `24157966_Maldives_IMF_2024_Article_IV_PEP8_Documented.ipynb`   Main documented Colab notebook

  `README.md`                                                     Setup, methodology, outputs, and
                                                                  limitations

  `requirements.txt`                                              Optional dependency file; create
                                                                  using the installation command
                                                                  below

  `IMF_2024_Outputs/`                                             Generated JSON results and figures;
                                                                  created when the notebook runs
  ---------------------------------------------------------------------------------------------------

The IMF PDF is an external input and is **not assumed to be included**
in the repository.

## Data source

**International Monetary Fund (2024).** *Maldives: 2024 Article IV
Consultation*, IMF Country Report No. 24/106.

Expected input filename:

``` text
null-002.2024.issue-106-en.pdf
```

Download the report from the IMF's official publication pages and place
it in your Google Drive `MyDrive` root, or update `PDF_PATH` to its
actual location.

## Requirements

-   Python 3 in Google Colab
-   Google Drive access for persistent input/output files
-   A Gemini API key stored in Colab Secrets as `GEMINI_API_KEY`
-   Internet access for model calls and the initial download of the
    embedding model

Install notebook dependencies:

``` python
%pip install pymupdf tiktoken sentence-transformers faiss-cpu google-genai seaborn pydantic
```

The notebook also uses NumPy, Pandas, Matplotlib, and IPython.

## Google Colab setup

**Important:** Mount Google Drive **before** creating any directory
under `/content/drive`. Otherwise, Colab can raise
`ValueError: Mountpoint must not already contain files`.

Use the following as the initial configuration cell (replacing the
original input/output setup cell):

``` python
from pathlib import Path
import hashlib
from google.colab import drive

drive_root = Path("/content/drive")
if not (drive_root / "MyDrive").is_dir():
    drive.mount(str(drive_root))

my_drive = drive_root / "MyDrive"
PDF_PATH = my_drive / "null-002.2024.issue-106-en.pdf"
OUT = my_drive / "IMF_2024_Outputs"
OUT.mkdir(parents=True, exist_ok=True)

if not PDF_PATH.is_file():
    raise FileNotFoundError(
        f"IMF PDF not found: {PDF_PATH}. Update PDF_PATH to its Drive location."
    )

print("PDF:", PDF_PATH)
print("SHA256:", hashlib.sha256(PDF_PATH.read_bytes()).hexdigest())
print("Outputs:", OUT)
```

If an earlier run created local files in `/content/drive` before
mounting, back them up and reset the Colab runtime before running the
corrected setup.

Configure `GEMINI_API_KEY` using **Colab → Secrets**. Do not commit API
keys or credentials to GitHub.

The notebook's default model identifiers are configurable with
environment variables `GEN_MODEL` and `JUDGE_MODEL`. Confirm that the
configured Gemini model names are available in your API account.

## Methodology

### Task 1 --- PDF processing and LLM summarisation

-   Extract page-labelled text with **PyMuPDF (`fitz`)**.
-   Clean ligatures, hyphenated line breaks, repetitive headers, and
    whitespace.
-   Create **400-token chunks** with **50-token overlap**, using the
    `cl100k_base` tokenizer.
-   Map report sections to approximate PDF page ranges.
-   Generate section summaries and a map-reduce executive synthesis.
-   Evaluate consistency, completeness, and factual alignment using a
    Gemini-based judge.
-   Use retries and checkpoint files to handle transient API failures.

### Task 2 --- Financial-stability themes and indicators

The notebook counts matches for seven themes:

1.  Inflation & Prices
2.  Interest Rates & Credit
3.  Banking Capital & Liquidity
4.  Sovereign Debt
5.  Systemic Financial Risk
6.  Housing & Real Estate
7.  Labor Market Dynamics

Theme counting operates on cleaned pages rather than overlapping chunks
to avoid overlap-induced double counting.

The structured indicators include 2024 GDP growth and inflation
projections and 2023 Q3 banking soundness measures. Missing or
unverified policy-rate and unemployment values are represented as
`null`, rather than fabricated estimates. The notebook also records
qualitative strengths, vulnerabilities, and source provenance.

### Task 3 --- Sentence Transformers and FAISS RAG

-   Encode PDF chunks using a local Sentence Transformers model.
-   Normalise embeddings and index them with **FAISS `IndexFlatL2`**.
-   Retrieve the nearest passages for each question.
-   Prompt Gemini to answer **only from retrieved excerpts**, with PDF
    page references.
-   Compare single-query retrieval against deterministic
    **three-variation multi-query retrieval**.

A lower embedding distance does not necessarily mean that a retrieved
passage answers the question correctly.

### Task 4 --- Visual analysis

The notebook produces high-resolution figures covering:

-   Relative frequency of the seven financial themes
-   Real GDP growth and inflation
-   Public debt as a share of GDP
-   Money and domestic credit growth
-   Tier 1 capital and non-performing loans (NPLs)
-   Theme frequency by PDF page
-   Supplementary credit-growth versus NPL comparison

Charts distinguish **historical observations**, **2023 estimates**, and
**2024--2029 IMF projections** where applicable.

## Generated outputs

Outputs are written to the directory configured by `OUT`, for example:

``` text
MyDrive/
├── null-002.2024.issue-106-en.pdf
└── IMF_2024_Outputs/
    ├── pages.json
    ├── chunks.json
    ├── section_audits.json
    ├── core_indicators.json
    ├── ... additional summaries, audits and retrieval results
    └── ... generated figures
```

The exact output filenames depend on the executed notebook cells. Run
all relevant cells to produce the complete set.

## Findings and critical evaluation

The thematic analysis emphasises **sovereign debt**, **systemic
financial risk**, and **banking-sector resilience**. The structured
results include a **5.2% real GDP growth projection for 2024**, **4.8%
end-of-period CPI-Malé inflation projection for 2024**, **5.2% NPL ratio
in 2023 Q3**, and **43.5% Tier 1 capital ratio in 2023 Q3**.

Automated section audits awarded **5/5** across consistency,
completeness, and factual alignment, but these scores are not
independently verified accuracy measures. Observed limitations include:

-   Different inflation and banking-capital figures across outputs,
    potentially reflecting distinct definitions or reporting periods
-   RAG failures to retrieve some indicators already available in the
    structured extraction
-   Multi-query retrieval requiring additional time without consistently
    improving answer completeness
-   Potential hallucinated or weakly supported citations
-   Source-excerpt truncation, PDF parsing errors, and possible
    LLM-judge bias

**Critical distinction:** A missing retrieval result or a numerical
discrepancy is not automatically proof of hallucination. Claims should
be checked against the IMF's original tables, including their units,
periods, and definitions.

## Reproducibility and limitations

-   Store the original PDF and record its SHA256 checksum.
-   Keep the notebook's input paths, model identifiers, chunk
    parameters, and source-table assumptions documented.
-   API responses may change across model versions and executions.
-   `temperature=0` reduces sampling variability but does not guarantee
    correctness.
-   Keyword counts indicate textual emphasis, **not** measured financial
    risk severity.
-   PDF page numbers may differ from printed report page numbers.
-   IMF projections are conditional scenarios, **not** realised
    outcomes.
-   Manual verification is required before treating extracted numerical
    claims as authoritative.

## Suggested GitHub `.gitignore`

``` gitignore
.env
.env.*
!.env.example
__pycache__/
.ipynb_checkpoints/
*.pyc
*.faiss
*.index
IMF_2024_Outputs/
imf_2024_outputs/
```

Avoid uploading private API credentials, unnecessary model caches, or
large intermediate files. Only publish the source PDF if its
distribution terms permit it.

## How to run

1.  Open the notebook in Google Colab.
2.  Install dependencies.
3.  Run the **corrected Google Drive setup cell**.
4.  Ensure the IMF PDF exists at `PDF_PATH`.
5.  Configure `GEMINI_API_KEY` in Colab Secrets.
6.  Execute the PDF extraction, summarisation, theme analysis,
    structured extraction, RAG, and visualisation cells in order.
7.  Review generated files under `OUT` and manually verify important
    claims against the source report.

## Reference

International Monetary Fund. (2024). *Maldives: 2024 Article IV
Consultation*. IMF Country Report No. 24/106.

------------------------------------------------------------------------

**Purpose:** Academic demonstration of reproducible, critically
evaluated LLM-assisted analysis of a financial-policy report.
