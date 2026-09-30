## Scientific Paper Reproduction Tasks

InternAgent can take a published scientific paper, the data it used, and attempt to reproduce its key findings autonomously — writing the analysis code, running it, iterating on errors, and producing a written report. This is what a *sci_task* is.

You do not need to know anything about AI or machine learning to run one. You provide the scientific materials; InternAgent does the coding and analysis.

---

### What the agent does

Given a task, InternAgent will:

1. Read the task description and understand what findings need to be reproduced
2. Write analysis code from scratch, informed by the provided data and any related literature
3. Run the code, debug failures, and revise iteratively
4. Produce a written research report summarising the methods and results
5. Score the report automatically against the original paper's findings

This process typically takes 30–90 minutes depending on the complexity of the analysis and the number of iterations allowed.

---

### Task structure

Each task is a folder under `sci_tasks/tasks/` with the following layout:

```
sci_tasks/tasks/Astronomy_000/
├── task_info.json        # Task description and list of input data files
├── data/                 # Input data files (measurements, samples, tables, etc.)
├── target_study/
│   ├── paper.pdf         # The paper being reproduced
│   ├── checklist.json    # Evaluation criteria derived from the paper
│   └── images/           # Reference figures from the paper
└── related_work/         # Background papers for context (PDFs)
```

`task_info.json` describes the scientific goal and each data file. The agent reads this at the start and uses it to plan its approach.

`checklist.json` is used for automated scoring after the run. It contains a list of specific findings from the paper (quantitative results, figures, analytical conclusions) that the agent's output is judged against.

---

### Running a task

Initialize the submodule if you have not already (needed after a fresh clone):

```bash
git submodule update --init
```

Then run with the default task (`Astronomy_000`):

```bash
bash scripts/run_sci.sh
```

Or specify a different task by name:

```bash
bash scripts/run_sci.sh Chemistry_001
```

The full list of available tasks is under `sci_tasks/tasks/`. Task names follow the pattern `<Domain>_<index>`, covering Astronomy, Chemistry, Earth, Energy, Information, and more.

---

### What to expect during the run

The agent runs as a loop: it generates hypotheses about how to approach the reproduction, writes code, executes it, and refines based on the output. You will see log output in the terminal and a live log file under `logs/`.

Results are saved under `results/` in a timestamped folder. Inside you will find:

```
results/<timestamp>_<run_name>/
├── run_0/               # Baseline (empty — the agent starts from scratch)
├── run_1/               # First attempt
│   ├── code/            # Code written by the agent
│   ├── outputs/         # Output files and figures produced
│   ├── report/
│   │   └── report.md    # Written research report
│   └── final_info.json  # Scores for this run
└── ...
```

Each `run_N/` folder represents one iteration. The agent builds on the best result from the previous run.

**Expected runtime:** a full run with default settings can take several hours, since `default_config.yaml` runs 10 discovery rounds with up to 5 ideas per round. For a quick first test, reduce these in `config/default_config.yaml`:

```yaml
workflow:
  loop_rounds: 1       # default: 10 — number of full idea→experiment cycles
  top_ideas_count: 2   # default: 5  — ideas generated and run per round

experiment:
  max_runs: 2          # default: 2  — experiment iterations per idea (fine as-is)
```

With `loop_rounds: 1` and `top_ideas_count: 2`, a sci_task typically completes in 30–60 minutes.

---

### How scoring works

After each run, an LLM judge reads the agent's `report.md` alongside the paper's `checklist.json` and scores each criterion on a 0–100 scale:

- **50** means the result is roughly comparable to what the published paper reports
- **Above 50** means the agent's result is stronger than the paper on that criterion
- **Below 50** means the result is weaker or incomplete

Criteria are weighted (as defined in `checklist.json`) and combined into a single `total_score`. Both quantitative results (specific numbers, metrics, plots) and qualitative analysis (interpretation, mechanistic reasoning) are evaluated.

The scores appear in `final_info.json` inside each run folder and are summarised in the terminal at the end of the run.

---

### Adding your own task

To reproduce a paper of your own, create a folder under `sci_tasks/tasks/` with:

- `task_info.json` — describe the scientific goal and list each data file with its name, path, and a one-sentence description
- `data/` — place the actual data files here
- `target_study/paper.pdf` — the paper to reproduce
- `target_study/checklist.json` — a list of findings to evaluate against (see any existing task for the format)
- `target_study/images/` — reference figures from the paper (for image-based checklist items)
- `related_work/` — optional background PDFs

Once the folder is in place, run it with:

```bash
bash scripts/run_sci.sh <your_task_name>
```

## 📝 Citation

```bibtex
@article{team2025internagent,
  title={InternAgent: When Agent Becomes the Scientist--Building Closed-Loop System from Hypothesis to Verification},
  author={Team, InternAgent and Zhang, Bo and Feng, Shiyang and Yan, Xiangchao and Yuan, Jiakang and Ma, Runmin and Hu, Yusong and Yu, Zhiyin and He, Xiaohan and Huang, Songtao and others},
  journal={arXiv e-prints},
  pages={arXiv--2505},
  year={2025}
}
```

```bibtex
@article{du2025automlgen,
  title={AutoMLGen: Navigating Fine-Grained Optimization for Coding Agents},
  author={Shangheng Du and Xiangchao Yan and Dengyang Jiang and Jiakang Yuan and Yusong Hu and Xin Li and Liang He and Bo Zhang and Lei Bai},
  journal={arXiv preprint arXiv:2510.08521},
  year={2025}
}
```
