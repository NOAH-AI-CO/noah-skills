# Noah Skills Agents Guide

This file helps AI agents discover and use the medical data search skills provided in this repository.

All available skills live under `skills/<name>/SKILL.md`:

| Skill name | Folder | Primary use (when to trigger) |
|-----------|--------|--------------------------------|
| `clinical-trial-search` | `clinical_trail` | **Clinical trial search**: use when the user asks about clinical trials, NCT IDs, trial phases, enrollment/recruitment, route of administration, trial locations, sponsors, or efficacy/safety readouts. |
| `drug-search` | `drug_pipeline` | **Drug pipeline / development search**: use when the user asks about specific drugs/molecules, targets, indications, companies, modalities, routes of administration, or development stages (e.g. Phase 1/2/3, approved). |
| `medical-conference-search` | `medical_conference` | **Medical conference and abstract search**: use when the user asks about medical conferences (e.g. ASCO, ESMO), sessions, posters, oral presentations, or data presented at conferences (efficacy/safety summaries, trial updates, etc.). |

### Environment and Dependencies

- Python 3.8+
- `requests` library (`pip install requests`)
- Environment variable `NOAH_API_TOKEN` (register at `noah.bio` to obtain an API token)

#### How to set `NOAH_API_TOKEN`

On **macOS / Linux (bash/zsh)**, add this to your shell (or run it in the current terminal):

```bash
export NOAH_API_TOKEN="your_token_here"
```

To make it persistent, append that line to your `~/.bashrc`, `~/.zshrc`, or other shell startup file.  

On **Windows (PowerShell)**:

```powershell
$Env:NOAH_API_TOKEN = "your_token_here"
```

On **Windows (Command Prompt)**:

```cmd
set NOAH_API_TOKEN=your_token_here
```

### Common Invocation Pattern

All skills ultimately parse a natural language question into structured JSON parameters and then call a common search entrypoint:

```bash
python scripts/search.py --params '<JSON string>'
```

Or via a parameter file:

```bash
python scripts/search.py --params-file /tmp/query.json
```

Some skills support additional flags (for example, the conference skill uses `--conference-params` and `--presentation-params`). Follow each skill's own `SKILL.md` for the exact CLI options.

### Usage Guidelines

- **Pick the right skill**: decide based on whether the question is primarily about *clinical trials*, *drugs/pipeline*, or *conferences and abstracts*, and call the corresponding skill.  
- **Prefer English keywords**: the underlying indices are English-first; if the user asks in another language, translate disease names, drugs, and targets into English before constructing query parameters.  
- **Pagination and filters**: skills default to `page_num/from_n = 0` and `page_size/size = 5`. If there are too many results, prompt the user to narrow filters; if there are no results, suggest relaxing some constraints.

For full field definitions, example parameter JSON, and response structures, see the corresponding `skills/<name>/SKILL.md` file for each skill.
