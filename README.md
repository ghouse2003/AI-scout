# AI-Scout 🤖

> **The HR-friendly AI agent that ranks job candidates by matching their résumé against a job description and measuring their expressed interest level.**

AI-Scout automates the first round of candidate screening. It reads résumés (PDF or plain text), compares them to a target job description using an LLM, and produces a ranked shortlist with a **match score** (how well the résumé fits the role) and an **interest score** (how enthusiastically the candidate wrote about the role). HR teams get an instant, bias-reduced shortlist without reading every application manually.

---

## Table of Contents

1. [Features](#features)
2. [How It Works](#how-it-works)
3. [Prerequisites](#prerequisites)
4. [Installation & Setup](#installation--setup)
5. [Configuration](#configuration)
6. [Running the Agent](#running-the-agent)
7. [Expected Output](#expected-output)
8. [Project Structure](#project-structure)
9. [Contributing](#contributing)
10. [License](#license)

---

## Features

- 📄 **Multi-format résumé ingestion** – supports PDF and plain-text (`.txt`) résumé files.
- 🧠 **LLM-powered analysis** – uses a large language model to evaluate skills, experience, and language tone.
- 📊 **Dual scoring**
  - *Match Score* – how closely the candidate's background aligns with the job requirements (0 – 100).
  - *Interest Score* – how strongly the candidate has expressed enthusiasm for the role (0 – 100).
- 🏆 **Automatic ranking** – outputs a sorted shortlist so the best-fit candidates always appear first.
- 💾 **Structured output** – results are saved as a CSV file for easy review in Excel or Google Sheets.

---

## How It Works

```
Résumé Files  ──►  Text Extraction  ──►  LLM Evaluation  ──►  Score Calculation  ──►  Ranked CSV Output
Job Description ──►┘
```

1. **Text extraction** – résumés are parsed from PDF/TXT format into raw text.
2. **Prompt construction** – each résumé is combined with the job description into a structured prompt.
3. **LLM evaluation** – the model returns a match score, an interest score, and a short justification for each candidate.
4. **Ranking** – candidates are sorted by a weighted combined score (configurable weights).
5. **Export** – the ranked list is written to `output/results.csv`.

---

## Prerequisites

| Requirement | Minimum Version | Notes |
|---|---|---|
| Python | 3.9 | [python.org](https://www.python.org/downloads/) |
| pip | 22.0 | Bundled with Python 3.9+ |
| OpenAI API key **or** Ollama | — | See [Configuration](#configuration) |
| Git | 2.x | For cloning the repository |

> **Optional:** A virtual-environment tool such as `venv` (built-in) or `conda` is strongly recommended.

---

## Installation & Setup

### 1 – Clone the repository

```bash
git clone https://github.com/ghouse2003/AI-scout.git
cd AI-scout
```

### 2 – Create and activate a virtual environment

**macOS / Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows (PowerShell)**
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 3 – Install dependencies

```bash
pip install -r requirements.txt
```

> If `requirements.txt` is not yet present, install the core libraries manually:
> ```bash
> pip install openai pypdf2 pandas python-dotenv
> ```

### 4 – Set up the environment file

Copy the provided example and fill in your credentials:

```bash
cp .env.example .env
```

Then open `.env` in your editor and add your API key (see [Configuration](#configuration) for all available options).

---

## Configuration

All runtime settings live in the `.env` file at the project root.

| Variable | Required | Default | Description |
|---|---|---|---|
| `OPENAI_API_KEY` | Yes* | — | Your OpenAI API key. Get one at [platform.openai.com](https://platform.openai.com/api-keys). |
| `MODEL_NAME` | No | `gpt-4o-mini` | Any OpenAI chat model, e.g. `gpt-4o`, `gpt-3.5-turbo`. |
| `MATCH_WEIGHT` | No | `0.7` | Weight given to the match score in the combined ranking (0–1). |
| `INTEREST_WEIGHT` | No | `0.3` | Weight given to the interest score (should sum to 1 with `MATCH_WEIGHT`). |
| `RESUME_DIR` | No | `resumes/` | Folder that contains candidate résumé files (PDF or TXT). |
| `JOB_DESC_FILE` | No | `job_description.txt` | Path to the plain-text file describing the open position. |
| `OUTPUT_DIR` | No | `output/` | Folder where the ranked CSV will be saved. |

> \* `OPENAI_API_KEY` can be replaced with a local [Ollama](https://ollama.com/) endpoint by setting `OLLAMA_BASE_URL` instead. See the project wiki for details.

**Example `.env`**

```dotenv
OPENAI_API_KEY=sk-...your-key-here...
MODEL_NAME=gpt-4o-mini
MATCH_WEIGHT=0.7
INTEREST_WEIGHT=0.3
RESUME_DIR=resumes/
JOB_DESC_FILE=job_description.txt
OUTPUT_DIR=output/
```

---

## Running the Agent

### 1 – Prepare your input files

**Job description** (`job_description.txt`)

```
Software Engineer – Backend
Requirements:
- 3+ years experience with Python
- Familiarity with REST APIs and SQL databases
- Strong written communication skills
```

**Résumé folder** (`resumes/`)

Place one résumé per file inside the `resumes/` directory. Supported formats: `.pdf`, `.txt`.

```
resumes/
├── alice_smith.pdf
├── bob_jones.txt
└── carol_white.pdf
```

### 2 – Run the agent

```bash
python main.py
```

Optional flags:

```
--resume-dir   PATH   Override the résumé directory (default: resumes/)
--job-desc     FILE   Override the job description file (default: job_description.txt)
--output       FILE   Override the output CSV path (default: output/results.csv)
--top-n        INT    Print only the top N candidates to the console (default: all)
```

**Example with flags**

```bash
python main.py --resume-dir ./applicants --job-desc ./roles/backend_eng.txt --top-n 5
```

---

## Expected Output

### Console

After processing, AI-Scout prints a summary table to the terminal:

```
AI-Scout – Candidate Ranking
=============================================================
Rank  Candidate          Match   Interest  Combined  Verdict
----  -----------------  ------  --------  --------  -------
1     alice_smith        88/100  76/100    84.8      ✅ Strong fit
2     carol_white        74/100  82/100    76.4      ✅ Good fit
3     bob_jones          61/100  55/100    59.2      ⚠️  Partial fit
=============================================================
Results saved to: output/results.csv
```

### CSV File (`output/results.csv`)

| filename | candidate_name | match_score | interest_score | combined_score | summary |
|---|---|---|---|---|---|
| alice_smith.pdf | Alice Smith | 88 | 76 | 84.8 | Strong Python skills, relevant API experience… |
| carol_white.pdf | Carol White | 74 | 82 | 76.4 | Enthusiastic cover note, solid SQL background… |
| bob_jones.txt | Bob Jones | 61 | 55 | 59.2 | Limited backend experience, good communication… |

---

## Project Structure

```
AI-scout/
├── main.py                  # Entry point – orchestrates the full pipeline
├── agent/
│   ├── extractor.py         # Résumé text extraction (PDF + TXT)
│   ├── scorer.py            # LLM prompt construction and score parsing
│   └── ranker.py            # Weighted ranking and CSV export
├── resumes/                 # Place candidate résumé files here
├── job_description.txt      # Target job description
├── output/                  # Generated results are saved here
├── .env.example             # Template for environment variables
├── requirements.txt         # Python dependencies
└── README.md                # This file
```

---

## Contributing

Contributions are welcome! Here is how to get started:

1. Fork the repository and create a feature branch:
   ```bash
   git checkout -b feature/my-improvement
   ```
2. Make your changes and add tests where appropriate.
3. Ensure all tests pass:
   ```bash
   python -m pytest
   ```
4. Submit a pull request describing what you changed and why.

Please follow the existing code style and keep PRs focused on a single concern.

---

## License

This project is licensed under the **MIT License** – see the [LICENSE](LICENSE) file for details.

---

*Built with ❤️ to make hiring faster and fairer.*
