## Deep Research

FlowSearch: The Deep Research (DR) module is InternAgent's built-in literature and web research pipeline. Given a question or research topic, it decomposes the task into subtasks, gathers information in parallel from academic databases and the web, and synthesizes the findings into a structured answer or report. It is the backbone of QA mode and is also used during idea generation in discovery runs to ground hypotheses in existing literature.

**When is it useful?**

Use it whenever a task requires going beyond what is already in the agent's context — surveying a research area, verifying a claim against recent papers, or producing a standalone research answer. In discovery mode it runs in the background to inform idea generation; in QA mode it runs directly in response to a user question.

---

### Modes

The DR module has two output modes:

| Mode | Output | When to use |
|---|---|---|
| `qa` | Concise direct answer | Single factual or research questions |
| `report` | Structured markdown report with sections and references | Surveys, in-depth backgrounds, multi-part topics |

QA mode skips outline generation and section polishing, making it significantly faster. Report mode runs a full multi-stage synthesis pipeline and optionally polishes each section.

---

### Configuration in Discovery Mode

When running a discovery experiment, DR is controlled by the `agents.dr` block in `config/default_config.yaml`:

```yaml
agents:
  dr:
    model_provider: "default"  # Model to use (inherits from models.default_provider)
    enabled: true              # Set to false to disable DR background research entirely
    mode: "simple"             # Which DR config to use: "simple" or "complex"
                               #   simple  → config_simple.yaml (faster, no coordinator)
                               #   complex → config_complex.yaml (thorough, with coordinator)
```

Disabling DR (`enabled: false`) skips background literature grounding during idea generation — useful when running offline or when API rate limits are a concern.

---

### Configuration

Three pre-built configs are provided under `internagent/mas/agents/dr_agents/`:

| Config file | Mode | Coordinator | Polish | Best for |
|---|---|---|---|---|
| `config_qa.yaml` | `qa` | off | off | Direct question answering (~5–10 min) |
| `config_simple.yaml` | `report` | off | off | Quick background research (~10–20 min) |
| `config_complex.yaml` | `report` | on | on | Comprehensive reports (~20–40 min) |

The coordinator is an optional validation stage that checks research completeness and adds follow-up subtasks if gaps are found. Enabling it improves quality at the cost of additional time and API calls.

Key parameters to tune:

```yaml
main:
  max_iter: 5              # Maximum execution cycles; increase for more thorough research
  enable_coordinator: false # Set true for quality validation loops (complex config only)

global_planner:
  max_nodes: 7             # Number of research subtasks generated; higher = broader coverage

global_execution:
  max_workers: 10          # Parallel threads; reduce if hitting API rate limits
  execution:
    max_tool_calls: 5      # Tool calls per subtask

synthesizer:
  mode: "qa"               # "qa" for direct answers, omit for report mode
  polish: false            # Set true to refine each section (report mode only)
```

To use a custom config, pass it via `DRAgent`:

```python
agent = DRAgent(model='o4-mini', config_path='path/to/my_config.yaml')
```

---

### API Keys and Environment Setup

The DR module uses five search tools. Required keys depend on which tools are enabled.

#### `arxiv_search`
Searches preprints on arXiv. No API key required.

#### `openalex_search`
Searches the OpenAlex academic database (250M+ works across all disciplines). No key required, but set an email for polite access:

```
OPENALEX_EMAIL=you@example.com
```

#### `crossref_search`
Searches CrossRef for DOI metadata and citations (150M+ records). No key required, but set an email:

```
CROSSREF_EMAIL=you@example.com
```

#### `volc_search`
Web search via Volcengine, with strong Chinese-language support and automatic full-text extraction. Requires an enterprise API key:

```
VOLC_SEARCH_API_KEY=your-key
```

#### `url_processor`
Fetches and extracts content from a URL, then answers a specific query against it. Uses Tavily as the primary extractor (fast and clean), falling back to Volcengine if unavailable:

```
TAVILY_API_KEY=tvly-your-key
```

Add these to your `.env` file alongside the existing OpenAI keys.

---

### Standalone Usage

QA mode is the primary standalone entry point. Report mode is used internally by the discovery pipeline — if you need it outside of a discovery run, instantiate `DRAgent` directly with a report config (see [Configuration](#configuration) above).

**QA mode** (direct question answering):

```bash
python launch_qa.py --question "What are recent advances in memory-augmented LLMs?" \
                    --output answer.md
```

Pass a local file with `--file` to ground the answer in a specific document (e.g. a paper PDF or notes):

```bash
python launch_qa.py --question "What methodology does this paper use?" \
                    --file path/to/paper.pdf \
                    --output answer.md
```

To invoke `DRAgent` directly (e.g. with a custom config):

```python
from internagent.mas.agents.dr_agent import DRAgent
import asyncio

agent = DRAgent(model='o4-mini', config_path='path/to/my_config.yaml')
answer = asyncio.run(agent.execute({'task': 'Your research question'}, {}))
print(answer)
```


## 📝 Citation

```bibtex
@article{hu2025flowsearch,
  title={FlowSearch: Advancing deep research with dynamic structured knowledge flow},
  author={Yusong Hu and Runmin Ma and Yue Fan and Jinxin Shi and Zongsheng Cao and Yuhao Zhou and Jiakang Yuan and Xiangchao Yan and Wenlong Zhang and Lei Bai and Bo Zhang},
  journal={arXiv preprint arXiv:2510.08521},
  year={2025}
}
```