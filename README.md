## Noah Skills

A collection of domain-specific skills for querying Noah medical data APIs from AI agents or other automation. Each skill lives in its own folder under `skills/` with a `SKILL.md` description and a small CLI wrapper.

### Repository structure

- `AGENTS.md` — overview of all available skills and how agents should select and call them.
- `skills/clinical_trail` — **clinical-trial-search** skill for querying clinical trial databases.
- `skills/drug_pipeline` — **drug-search** skill for querying drug pipeline / development data.
- `skills/medical_conference` — **medical-conference-search** skill for querying conferences and presentations.

Query skill folder typically contains:

- `SKILL.md` — formal skill specification (when to trigger, parameters, examples, dependencies).
- `scripts/search.py` — CLI entrypoint that takes JSON parameters and calls the Noah backend API.

### Requirements

- Python 3.8+
- `requests` (`pip install requests`)
- Environment variable `NOAH_API_TOKEN` set to a valid API token from `noah.bio`

#### Setting `NOAH_API_TOKEN`

macOS / Linux (bash/zsh):

```bash
export NOAH_API_TOKEN="your_token_here"
```

### Running a skill from the CLI

All skills use a similar pattern: construct a JSON object with the query parameters described in that skill’s `SKILL.md`, then call the corresponding `scripts/search.py`.

Example (clinical trial search with inline JSON):

```bash
python skills/clinical_trail/scripts/search.py --params '{
  "indication": ["lung cancer"],
  "phase": ["Phase 3"],
  "page_num": 0,
  "page_size": 5
}'
```

Or with a params file:

```bash
echo '{
  "indication": ["lung cancer"],
  "phase": ["Phase 3"],
  "page_num": 0,
  "page_size": 5
}' > /tmp/query.json

python skills/clinical_trail/scripts/search.py --params-file /tmp/query.json
```

See each `skills/<name>/SKILL.md` for the full schema, examples, and any additional flags (e.g. conference-specific parameters).