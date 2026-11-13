# skim-then-verify

A Claude skill: **Haiku reads a long document in full; a stronger model only cross-checks the figures and decisive claims.**

## Why
- Cheaper and faster: the raw reading happens in a subagent and does not fill the main context.
- Still trustworthy: the stronger model recomputes and spot-checks the decisive points instead of blindly trusting the smaller model's report.

## Structure
```
skim-then-verify/
├── README.md
├── evals/
│   └── evals.json
├── test-files/
│   ├── flawed-report.txt
│   ├── figures.csv
│   └── short-note.txt
└── skills/
    ├── skim-then-verify/
    │   └── SKILL.md
    └── plan-write-review/
        └── SKILL.md
```

## Skills
- **skim-then-verify** (reading): Haiku reads a long document in full; the stronger model cross-checks figures and decisive claims.
- **plan-write-review** (writing): the stronger model plans and classifies; Haiku drafts ROUTINE chunks in parallel while the stronger model writes the CORE chunks itself; routine drafts are skimmed and cross-checked against a numbered fact sheet (Haiku returns a CLAIMS list to make this cheap); then merge and publish.

## Requirements
Needs a tool for spawning subagents (e.g. the Agent/Task tool). Without it, `skim-then-verify` falls back to keyword search and `plan-write-review` writes everything itself, and both report that no agents ran.

## Installation
Copy the skill folders you want from `skills/` into your skills directory (for example `~/.claude/skills/` for Claude Code, or `.claude/skills/` inside a project):
```
cp -r skills/* ~/.claude/skills/
```

## Making Claude actually follow the skills
Skills are only loaded when Claude judges them relevant, and Claude may treat them as optional context. To make them stick:
- Each skill opens with a **copyable checklist** that Claude ticks off, and uses MUST for critical rules.
- Descriptions are third person and list trigger phrases ("review", "fact-check", "long report", "save tokens").
- Name the skill in your request: "use skim-then-verify on this report".
- Add a standing rule to your project's `CLAUDE.md` or custom instructions, for example: "For long documents, you MUST use the skim-then-verify skill. For long multi-section outputs, you MUST use plan-write-review."
- Run the cases in `evals/evals.json` against Haiku, Sonnet, and Opus, and tighten the wording wherever a model skips a step.

## Cost note (measured)
Each Haiku subagent carries a fixed input overhead (about 60k tokens in our tests, even for a one-word reply). Delegation therefore pays off only for large inputs or chunks: `plan-write-review` delegates chunks of about 300+ words and only when the total ROUTINE text is about 600+ words; otherwise it writes everything itself. Ten subagents ran in parallel in one message without errors; higher counts were not tested.

## Usage
Ask Claude to read and assess a long document, for example: "use skim-then-verify on this report". The skill will:
1. Have Haiku read everything and extract key figures and suspicious points.
2. Have the stronger model cross-check the decisive items and recompute the calculations.
3. Return a conclusion that states what was verified.

## Tested
Evals 1-5 and 7 were run manually (7 only at the mechanism level: a fabricated figure, name and causal claim were caught by the scan step). Eval 6 ("no subagents") was checked by reading the skill, not run.

## Do not use for
Short documents, or work that needs deep reasoning and a polished final write-up.
