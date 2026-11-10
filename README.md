# my_teaching_tools

Tools I build for my teaching and research work. Each tool here is a **Claude skill**: a folder with a `SKILL.md` that tells Claude how to run a structured workflow.

| Skill | What it does |
|---|---|
| [`skills/research-os`](skills/research-os/SKILL.md) | **ResearchOS v2.** An evidence-backed research-discovery pipeline. It scopes an idea (PICO), mines the literature, finds gaps, checks novelty against the closest existing work, runs an expert-council review, and gives a PARK / PRIORITISE / REJECT verdict. A citation ledger and audit gates block hallucinated or unverified citations. Works on one idea or several in parallel, and has a benchmark mode for measuring the skill itself. |
| [`skills/code-team`](skills/code-team/SKILL.md) | **Code Team.** A project-manager-led coding team for research code. The PM writes a brief, then a coder, an independent tester, a sample-run expert and a fresh-context code reviewer each do their part, with at most two fix loops. The PM's final check confirms that the code matches what the paper or project claims, and every "it works" must map to a test that actually ran. |
| [`skills/dsu-qp-scrutiny`](skills/dsu-qp-scrutiny/SKILL.md) | **Question-paper scrutiny.** Audits a question paper (.docx) against Dayananda Sagar University's approved Bloom's-taxonomy action-verb sheet (after Anderson & Krathwohl, 2001) and the review panel's recorded decisions, makes minimal verb or level fixes, then fills the School of Engineering Scrutiny Form PDF from the fixed paper (Bloom's % per level, CO mapping, marks checks). The two Python scripts it uses are embedded in `SKILL.md`. Built for DSU's format; other institutions can swap in their own verb sheet. |

## Using a skill

- **Claude Code:** copy the skill folder into `~/.claude/skills/` (for example `~/.claude/skills/research-os/`). Claude picks it up from the `description` in `SKILL.md`, or you can call it by name.
- **Claude apps:** zip the skill folder (the zip should contain the folder with `SKILL.md` at its top level) and add it as a custom skill.

ResearchOS needs web search; Code Team and the scrutiny skill need a code workspace (Python). They use subagents when the environment offers them and fall back to clearly separated passes when it does not.

## Design principles

- **No claim without evidence.** Citations must be fetched or marked `UNVERIFIED`; test results, timings and sample numbers must come from real runs.
- **Nobody grades their own work.** Testers write tests from the spec, not the code; reviewers start from a fresh context; an audit step checks the writer.
- **Honest output over tidy output.** Open issues, thin evidence and failed checks are reported, not smoothed over.

## Author

Souvik Ghosh, Assistant Professor, Department of CSE (AI & ML), Dayananda Sagar University, Bengaluru. GitHub: [@souvikg23](https://github.com/souvikg23)

## License

[MIT](LICENSE)
