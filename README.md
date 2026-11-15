# Data Collection Projects

Three data-collection projects developed for the *Data Collection Techniques* course of the Master's in Big Data Science and Artificial Intelligence at the University of Navarra. Each project gathers data from a different kind of source — a live website, unstructured PDF documents and a public REST API — and turns it into structured, usable data.

| Notebook | Source | Techniques |
|---|---|---|
| [clinicaltrials_glioblastoma.ipynb](clinicaltrials_glioblastoma.ipynb) | ClinicalTrials.gov API | REST API, JSON parsing, pagination, data cleaning |
| [llm_pdf_metadata_extraction.ipynb](llm_pdf_metadata_extraction.ipynb) | Scientific articles (PDF) | PDF text extraction, local LLM from Hugging Face, prompt engineering |
| [stackoverflow_error_helper.ipynb](stackoverflow_error_helper.ipynb) | Stack Overflow website | Web scraping with Selenium and BeautifulSoup |

The notebooks are written in Spanish; each one starts with an executive summary.

**Author:** Aitor Madoz González — BSc in Biochemistry (University of Navarra), MSc student in Big Data Science and Artificial Intelligence. Interested in data-driven roles in the pharmaceutical industry.

---

## 1. Clinical trials in glioblastoma — ClinicalTrials.gov API

Glioblastoma is the most common and aggressive malignant brain tumour in adults, with a standard of care that has barely changed in two decades and a median survival of around 15 months. For a pharmaceutical company considering this field, the first step is to map the clinical trial landscape: who sponsors the trials, which phase they are in, which drugs are tested and how many patients they enrol.

The notebook connects to the public **ClinicalTrials.gov API (v2)** and builds a DataFrame with every registered glioblastoma trial (2,288 studies, 16 columns): status, phase, sponsor and sponsor type, interventions, enrolment, design (randomisation, masking), start and primary completion dates, countries, number of sites and whether results are posted.

**How it works:** a small test request to inspect the JSON structure; a function that extracts the relevant fields from each study and tolerates missing ones; pagination following `nextPageToken` until all studies are downloaded; cleaning (API codes translated, dates parsed, trials sorted by phase and enrolment); and a styled table where each title is coloured by outcome — green (completed), yellow (ongoing) and red (terminated, suspended or withdrawn).

**Problems solved:** not every study has every field, so all lookups default to null instead of failing; the primary completion date was chosen among two possible end dates because it indicates when results will be available; and sorting by start date showed almost only ongoing trials, so trials are sorted by phase and size instead.

Change `ENFERMEDAD` in the parameters cell to build the same table for any other condition.

---

## 2. Metadata extraction from scientific PDFs with an LLM

Six scientific articles in PDF are turned into a table with one row per article and one column per variable (null when the value is missing): title, health condition studied, study population, keywords and number of references.

**How it works:** text is extracted with **pypdf**; the number of references is computed in Python by locating the bibliography and counting entries; the remaining fields are extracted by **Qwen2.5-3B-Instruct**, an open LLM downloaded from Hugging Face and run on a GPU (Google Colab, T4), using a prompt that asks for a JSON answer with `null` for missing values. Each article becomes a record appended to a list and converted into a pandas DataFrame, displayed with **Great Tables**.

**Problems solved:** on a Mac the model hung (PyTorch with MPS) or was too slow (MLX), so it runs on Colab; `pypdf` is pinned because different versions changed the reference count; one PDF encoded digits as glyph names (`/one.tnum`), converted back to numbers; JSON keys are in English because the model renamed keys requested in Spanish; and a first version asking for values that require interpreting the whole study (participants, placebo use, number of arms) failed, so the final version asks for information stated explicitly in the text.

**Limitations:** a 3B-parameter model handles clearly stated information well but struggles with fields that require interpretation; results would need human review.

**Data:** the articles are open access (Frontiers) and are not included. Download them and upload them when the notebook asks:
[fpsyg.2017.00308](https://doi.org/10.3389/fpsyg.2017.00308) ·
[fpsyg.2018.01240](https://doi.org/10.3389/fpsyg.2018.01240) ·
[fpsyg.2020.01354](https://doi.org/10.3389/fpsyg.2020.01354) ·
[fpsyg.2021.639236](https://doi.org/10.3389/fpsyg.2021.639236) ·
[fpsyt.2023.1301143](https://doi.org/10.3389/fpsyt.2023.1301143) ·
[fpubh.2022.1045777](https://doi.org/10.3389/fpubh.2022.1045777)

---

## 3. Stack Overflow error helper

A function, `get_SOS_help`, that runs a piece of Python code and, if it fails, searches the error on Stack Overflow and returns the first related question and its most-voted answer.

**How it works:** the code runs inside a `try/except` to capture the error; **Selenium** opens a real Chrome window and searches Stack Overflow (the captcha is solved by hand); the first question is parsed with **BeautifulSoup** to find the highest-scored answer; the result is printed and the link is opened in the browser.

**Problems solved:** Colab cannot show the browser window needed for the captcha, so it runs locally; two Chrome options hide the automation flags that Stack Overflow detected; quoted fragments (the user's own typo, e.g. `'siz'`) are removed from the error before searching; and the answer HTML is cleaned so code blocks and button labels do not get mixed up.

**Limitations:** removing quoted fragments can make the search too generic; the answer returned is the most voted, not necessarily the correct one.

---

## Setup
```bash
pip install -r requirements.txt
```
The Stack Overflow helper must run locally with Chrome installed. The PDF extraction was run on Google Colab with a T4 GPU. The ClinicalTrials.gov notebook runs anywhere.
