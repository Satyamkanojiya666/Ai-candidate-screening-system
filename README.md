# AI Candidate Screening System

[![CI](https://github.com/Satyamkanojiya666/ai-candidate-screening-system/actions/workflows/ci.yml/badge.svg)](https://github.com/Satyamkanojiya666/ai-candidate-screening-system/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Python](https://img.shields.io/badge/python-3.10%2B-blue)

An explainable resume-screening engine. It parses resumes (`.txt` / `.pdf`), matches them to a job, scores each candidate out of 100, ranks them, and tells the recruiter **why**, including skill gaps, red flags and interview questions.

**Task:** AI-RH-001 · AI Candidate Screening System · Data Alcott Systems, Free Online AI & Data Science Internship
**Student:** Satyam Sadanand Kanojiya · **Student Code:** DAS014162
**Task page:** https://www.freeinternships.in/ (Task ID AI-RH-001)

![Dashboard](docs/screenshots/01_dashboard.png)

## What makes it different

Most keyword screeners count how many skills appear. This one asks whether the resume actually *shows* them.

| Feature | What it does |
|---|---|
| **Evidence-weighted skills** | A skill shown in EXPERIENCE/PROJECTS earns full credit; one that is only *listed* under SKILLS earns 60%. Keyword-stuffed resumes stop ranking high. |
| **Related-skill credit** | Has PyTorch but not TensorFlow? Gets 40% credit for TensorFlow via a skill-similarity graph (also MySQL/PostgreSQL, AWS/Azure, Flask/Django, ...). |
| **Relevance-adjusted experience** | Six years of Java experience counts for little in an ML job: years are scaled by how well the candidate's skills cover the role. |
| **Interval-merged experience** | Dates are read only from the EXPERIENCE section and overlapping jobs are merged, so degree years and parallel roles are not double counted. |
| **Typo-tolerant matching** | "Tensorflw" or "Kubernates" is still recognised; "java" never matches "javascript", "sql" never matches "mysql". |
| **Blind screening mode** | Hides names, emails, phones and file names to reduce bias. Scores are provably identical (tested); identities stay recoverable after shortlisting. |
| **Explainability** | Every score comes with a plain-language reason, a component breakdown, red flags and tailored interview questions. |
| **Role recommender** | Scores every candidate against several jobs and shows their best fit. |
| **Web app + dashboard** | Flask upload interface and a self-contained HTML dashboard with charts. |
| **ATS check** | Flags resumes that automated tracking systems may parse badly. |
| **Evaluation** | Precision / recall / F1 against a manual shortlist. |

## Scoring (100 points, configurable)

| Component | Points | Rule |
|---|---|---|
| Required skills | 40 | average credit: demonstrated 1.0 · listed only 0.6 · related skill 0.4 |
| Preferred skills | 10 | same credit rules |
| Experience | 25 | relevant years ÷ (minimum + 2), capped at 1 |
| Education | 10 | PhD 100% · Master 80% · Bachelor 60% · Diploma 30% |
| Text similarity | 15 | TF-IDF (1-2 word phrases) cosine similarity between resume and job description |

Verdicts: **75+** Strong match · **60-74** Shortlist · **40-59** Maybe · **<40** Reject. Weights and thresholds live in `screener/config.py` and can be overridden with a JSON file (`--config`).

## NLP pipeline

```
resume (.txt/.pdf) -> read text -> split sections -> extract contact, skills, experience, education
                                         |
job description ---------> preprocess (lowercase, stop words, lemmatise) -> TF-IDF + cosine similarity
                                         |
                      score (skills + experience + education + similarity) -> rank -> explain -> report
```

## Quick start

```bash
git clone https://github.com/Satyamkanojiya666/ai-candidate-screening-system.git
cd ai-candidate-screening-system
pip install -r requirements.txt

python -m screener screen                 # rank the sample resumes
python -m screener screen --blind         # same, with identities hidden
python -m screener roles                  # best-fit job for each candidate
python -m screener web                    # web interface at http://127.0.0.1:5000
python -m unittest discover -s tests -t . # 33 tests
```

`screen` writes `output/dashboard.html`, `screening_report.txt`, `screening_results.csv` and `screening_results.json`.

Screen your own resumes:

```bash
python -m screener screen --job data/jobs/data_analyst.json --resumes path/to/resumes --out my_output
```

Resumes work best with headings such as `SKILLS`, `EXPERIENCE`, `EDUCATION`. PDF samples are in `data/pdf_samples/`.

## Sample results (AI Engineer job, 8 sample resumes)

| # | Candidate | Score | Why |
|---|---|---|---|
| 1 | Carol Davis | 89.6 | all 5 required skills demonstrated, PhD, 4 years |
| 2 | Alice Johnson | 86.6 | all 5 demonstrated, Master's, 3 years |
| 3 | Dev Patel | 70.3 | no TensorFlow but strong PyTorch: partial credit |
| 4 | Farah Khan | 57.4 | lists every skill but her experience is in sales; flagged |
| 5 | Hari Om | 47.6 | good projects, no work experience yet |
| 6 | Gita Rao | 45.4 | data analyst; 3-year employment gap flagged |
| 7 | Bob Smith | 27.7 | 1 year, mostly web development |
| 8 | Eve Martin | 20.4 | strong Java backend engineer, wrong role |

Against the demo manual shortlist: precision 1.0, recall 1.0. `roles` correctly routes Eve to Backend Developer and Gita to Data Analyst. These are 8 hand-written resumes, so treat the metrics as a demo, not proof of real-world accuracy.

| Blind mode | Web interface |
|---|---|
| ![Blind mode](docs/screenshots/02_blind_mode.png) | ![Web form](docs/screenshots/03_web_form.png) |

## Project structure

```
screener/
  config.py     weights and thresholds
  skills.py     skill taxonomy, fuzzy matching, related-skill groups
  parser.py     PDF/TXT reading, sections, contact, experience, education
  nlp.py        preprocessing (NLTK lemmatizer if installed, fallback otherwise)
  scoring.py    engine, ranking, role recommender, evaluation
  insights.py   ATS check, red flags, interview questions, explanations
  report.py     text report, CSV/JSON export, HTML dashboard
  app.py        Flask web interface
  cli.py        command line
data/           jobs, sample resumes (txt + pdf), manual labels
tests/          33 unit tests
docs/           project report, blog post, video script, screenshots, sample output
```

## Docker

```bash
docker build -t candidate-screening .
docker run -p 8000:8000 candidate-screening
```

## Limitations and responsible use

- Skills come from a fixed taxonomy (`screener/skills.py`); add new ones there. Custom job skills are matched literally.
- Date ranges are read in whole years. Scanned (image-only) PDFs need OCR, which is not included.
- No semantic embeddings (BERT etc.): similarity is TF-IDF, which cannot tell that "built neural networks" means "deep learning" unless the words overlap.
- Scores support human decisions and must not replace them. Red flags are questions to ask, not grounds for rejection. Blind mode reduces, but does not remove, bias risk. Check local hiring and data-protection laws before processing real applicants' resumes.

## License

MIT, see [LICENSE](LICENSE). Copyright (c) 2026 Satyam Sadanand Kanojiya.
