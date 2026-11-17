# skills

Claude Code skills for running bioinformatics hackathon projects, from the first idea to a clean, reviewed repository.

## Skills

- **hackathon-brainstorm** - Turns a rough hackathon problem statement into a scoped, peer-reviewed pipeline plan (README.md, requirements.txt and PROMPTS.md) before any code is written.
- **hackathon-firstprot** - Builds the first minimal, working end-to-end prototype for a scoped hackathon idea, with a small synthetic example dataset and run/test instructions.
- **hackathon-cleanup** - Turns a messy hackathon repo into an organized, documented, minimally-tested software package, logging every change to a prompt_history file.

## Installation

1. Clone the repository:

   ```bash
   git clone https://github.com/agenticbioinformatics/skills.git
   cd skills
   ```

2. Copy the skills into `.claude/skills/`. Each skill contains a `SKILL.md` file.

   For a single project (run from the cloned repo, replace `/path/to/your/project`):

   ```bash
   mkdir -p /path/to/your/project/.claude/skills
   cp -r hackathon-brainstorm hackathon-firstprot hackathon-cleanup /path/to/your/project/.claude/skills/
   ```

   Or for all your projects:

   ```bash
   mkdir -p ~/.claude/skills
   cp -r hackathon-brainstorm hackathon-firstprot hackathon-cleanup ~/.claude/skills/
   ```

3. Restart Claude Code

## Example usage

```bash
claude
```

```text
/hackathon-brainstorm We have 3 days to predict antibiotic resistance from bacterial genomes. What should we build?
/hackathon-firstprot Build a first prototype of the plan in README.md
/hackathon-cleanup Clean up this repo now that the hackathon is over
```