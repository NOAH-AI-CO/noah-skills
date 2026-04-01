---
name: medical-conference-search
description: "Search medical conference and presentation databases. Use this skill whenever the user asks about medical conferences, academic conferences, session abstracts, posters, oral presentations, or conference-presented drug/trial data. Three scripts are available: search_conferences.py (find conferences), search_presentations.py (find abstracts/presentations), and search_chained.py (find conferences then auto-inject into presentation search). Trigger words include: conference, symposium, congress, ASCO, ESMO, AHA, ACC, session, abstract, poster, oral presentation, data presented at, efficacy data, safety data, congress abstract."
env:
  - name: NOAH_API_TOKEN
    description: "API authentication token. Register for a free account at https://noah.bio to obtain your key."
    required: true
---

# Conference Search Skill

This skill searches a conference and presentation database using three independent scripts. Choose the script that fits the task — each can be used standalone, or combined in sequence.

---

## Scripts Overview

| Script | When to use |
|---|---|
| `scripts/search_conferences.py` | Find conferences by name, date, location, or therapeutic area |
| `scripts/search_presentations.py` | Find abstracts/presentations when you already know the conference name or series |
| `scripts/search_chained.py` | Discover the conference first, then auto-inject its name into a presentation search |

---

## Script 1 — Search Conferences Only

Use when the user asks about **which conferences exist**, **when/where they are held**, or wants to **browse conferences** by area or organization.

```bash
python scripts/search_conferences.py --params '<JSON>'
python scripts/search_conferences.py --params-file query.json
python scripts/search_conferences.py --params '<JSON>' --raw
python scripts/search_conferences.py --params '<JSON>' --output results.json
```

### Query fields

| Field | Type | Description | Example |
|---|---|---|---|
| `conference_name` | `str` | Full or partial conference name | `"ASCO Annual Meeting 2024"` |
| `conference_start_date` | `str` | Start date (YYYY-MM-DD) | `"2024-06-01"` |
| `conference_end_date` | `str` | End date (YYYY-MM-DD) | `"2024-06-05"` |
| `conference_location` | `str` | City, country, or venue | `"Chicago"` |
| `series_name` | `str` | Conference series name | `"ASCO"` |
| `series_organization` | `str` | Organizing body | `"American Society of Clinical Oncology"` |
| `series_area` | `List[str]` | Therapeutic area(s) | `["oncology", "cardiology"]` |
| `from_n` | `int` | Pagination offset | `0` |
| `size` | `int` | Results per page | `10` |

### Examples

```bash
# Find all ASCO conferences
python scripts/search_conferences.py --params '{"series_name": "ASCO"}'

# Oncology conferences in Chicago in 2024
python scripts/search_conferences.py --params '{"series_area": ["oncology"], "conference_location": "Chicago", "conference_start_date": "2024-01-01", "conference_end_date": "2024-12-31"}'

# Cardiology conferences, raw JSON
python scripts/search_conferences.py --params '{"series_area": ["cardiology"]}' --raw
```

### Result fields
`conference_name`, `conference_abbreviation`, `conference_website`, `conference_description`, `conference_start_date`, `conference_end_date`, `conference_location`, `series_id`, `series_name`, `series_abbreviation`, `series_website`, `series_organization`, `series_area`

---

## Script 2 — Search Presentations Only

Use when the user asks about **specific drugs, diseases, targets, authors, or institutions** and already knows (or doesn't need to filter by) an exact conference. The `series_name` field is sufficient for most queries.

```bash
python scripts/search_presentations.py --params '<JSON>'
python scripts/search_presentations.py --params-file query.json
python scripts/search_presentations.py --params '<JSON>' --raw
python scripts/search_presentations.py --params '<JSON>' --output results.json
```

### Query fields

| Field | Type | Description | Example |
|---|---|---|---|
| `authors` | `List[str]` | Presenter / author name(s) | `["John Smith"]` |
| `institutions` | `List[str]` | Author institution(s) | `["MD Anderson"]` |
| `drugs` | `List[str]` | Drug name(s) | `["pembrolizumab"]` |
| `diseases` | `List[str]` | Disease / indication(s) | `["lung cancer", "NSCLC"]` |
| `targets` | `List[str]` | Biological target(s) | `["PD-1", "VEGF"]` |
| `conference_name` | `str` | Exact conference name | `"2024 ASCO Annual Meeting"` |
| `series_name` | `str` | Conference series name | `"ESMO"` |
| `from_n` | `int` | Pagination offset | `0` |
| `size` | `int` | Results per page | `10` |

### Examples

```bash
# Pembrolizumab lung cancer abstracts at ESMO
python scripts/search_presentations.py --params '{"drugs": ["pembrolizumab"], "diseases": ["lung cancer"], "series_name": "ESMO"}'

# KRAS G12C data from MD Anderson researchers
python scripts/search_presentations.py --params '{"targets": ["KRAS G12C"], "institutions": ["MD Anderson"]}'

# All presentations at a specific conference (exact name required)
python scripts/search_presentations.py --params '{"conference_name": "2024 ASCO Annual Meeting", "diseases": ["NSCLC"]}'

# Save to file
python scripts/search_presentations.py --params '{"drugs": ["nivolumab"]}' --output results.json
```

### Result fields
`session_title`, `presentation_title`, `presentation_website`, `main_author`, `main_author_institution`, `authors`, `institutions`, `abstract`, `design`, `efficacy`, `safety`, `summary`, `drugs`, `diseases`, `targets`, `series_name`, `conference_name`

---

## Script 3 — Chained Search (Conference → Presentation)

Use when you need to **find the exact conference first**, then search its presentations. The top-matching `conference_name` from Step 1 is **automatically injected** into Step 2 — no manual copy-paste needed.

```bash
python scripts/search_chained.py \
    --conference-params '<JSON>' \
    --presentation-params '<JSON>'

# With file inputs
python scripts/search_chained.py \
    --conference-params-file conf.json \
    --presentation-params-file pres.json

# Raw JSON + save to file
python scripts/search_chained.py \
    --conference-params '<JSON>' \
    --presentation-params '<JSON>' \
    --raw --output results.json
```

### Examples

```bash
# PD-1 data at ASCO 2024 (auto-resolves conference name)
python scripts/search_chained.py \
    --conference-params '{"series_name": "ASCO", "conference_start_date": "2024-01-01", "conference_end_date": "2024-12-31"}' \
    --presentation-params '{"targets": ["PD-1"]}'

# Roche bispecific antibodies at hematology conferences
python scripts/search_chained.py \
    --conference-params '{"series_area": ["hematology"]}' \
    --presentation-params '{"drugs": ["bispecific antibody"], "institutions": ["Roche"]}'
```

---

## Choosing the Right Script

```
User asks about conferences / dates / locations
  → search_conferences.py

User asks about drug / disease / abstract data, knows the series (e.g. "at ESMO")
  → search_presentations.py  (use series_name field)

User asks about drug / disease / abstract data, knows the year but not exact conference name
  → search_chained.py  (resolves conference_name automatically)
```

---

## Conversion Examples

**User:** "What PD-1 drug data was presented at ASCO 2024?"

```bash
python scripts/search_chained.py \
  --conference-params '{"series_name": "ASCO", "conference_start_date": "2024-01-01", "conference_end_date": "2024-12-31"}' \
  --presentation-params '{"targets": ["PD-1"]}'
```

---

**User:** "Pembrolizumab lung cancer abstracts from ESMO"

```bash
python scripts/search_presentations.py \
  --params '{"drugs": ["pembrolizumab"], "diseases": ["lung cancer"], "series_name": "ESMO"}'
```

---

**User:** "Oncology conferences in Chicago 2024"

```bash
python scripts/search_conferences.py \
  --params '{"series_area": ["oncology"], "conference_location": "Chicago", "conference_start_date": "2024-01-01", "conference_end_date": "2024-12-31"}'
```

---

**User:** "KRAS G12C presentations by MD Anderson researchers"

```bash
python scripts/search_presentations.py \
  --params '{"targets": ["KRAS G12C"], "institutions": ["MD Anderson"]}'
```

---

**User:** "Roche bispecific antibody data at hematology conferences"

```bash
python scripts/search_chained.py \
  --conference-params '{"series_area": ["hematology"]}' \
  --presentation-params '{"drugs": ["bispecific antibody"], "institutions": ["Roche"]}'
```

---

## Tips

- If no results are returned, try relaxing filters (e.g. use `series_name` instead of `conference_name`, or broaden the disease term).
- `conference_name` in `search_presentations.py` must be an **exact match** — use `search_chained.py` or run `search_conferences.py` first to get the correct name.
- All `List[str]` fields accept multiple values: `["NSCLC", "lung cancer"]`.

---

## Dependencies

- Python 3.8+
- `requests` library (`pip install requests`)
- Environment variable `NOAH_API_TOKEN` — register at [noah.bio](https://noah.bio) to obtain your API key.