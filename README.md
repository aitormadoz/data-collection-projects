# Clinical trials in glioblastoma — ClinicalTrials.gov API

Glioblastoma is the most common and aggressive malignant brain tumour in adults, with a standard of care that has barely changed in two decades and a median survival of around 15 months. For a pharmaceutical company considering this field, the first step is to map the clinical trial landscape: who sponsors the trials, which phase they are in, which drugs are tested and how many patients they enrol.

This notebook connects to the public **ClinicalTrials.gov API (v2)** and builds a DataFrame with every registered glioblastoma trial (2,288 studies, 16 columns): status, phase, sponsor and sponsor type, interventions, enrolment, design (randomisation, masking), start and primary completion dates, countries, number of sites and whether results are posted.

## How it works
1. A small test request to check the API and inspect the JSON structure of a study.
2. A function, `extraer_fila`, that extracts the relevant fields from each study and tolerates missing ones.
3. **Pagination**: a loop that follows `nextPageToken` until all studies are downloaded, with a short pause between requests.
4. **Cleaning**: API codes are translated, dates are parsed and trials are sorted by phase and enrolment, so the large late-phase trials appear first.
5. A styled table where each title is coloured by outcome: green (completed), yellow (ongoing) and red (terminated, suspended or withdrawn).

## Problems solved
- Not every study has every field (observational studies have no phase), so all lookups default to null instead of failing.
- There are two completion dates; the primary completion date was chosen because it indicates when results will be available.
- Sorting by start date showed almost only ongoing trials (glioblastoma trials take years), so trials are sorted by phase and size instead.

## Run
```bash
pip install requests pandas
```
Change `ENFERMEDAD` in the parameters cell to build the same table for any other condition.
