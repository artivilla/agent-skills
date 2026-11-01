# Agent Skills

Shared skills for Codex, Claude Code, AMP, and other agent tools.

## Install

Clone this repository, then make Codex's expected skills path point at it:

```bash
mkdir -p ~/dev/github.com/artivilla
git clone https://github.com/artivilla/agent-skills.git ~/dev/github.com/artivilla/agent-skills
mkdir -p ~/.agents
mv ~/.agents/skills ~/.agents/skills.backup 2>/dev/null || true
ln -s ~/dev/github.com/artivilla/agent-skills ~/.agents/skills
```

To install a single skill directly from GitHub, clone or download the
repository, then copy that skill directory into `~/.agents/skills`. For this
repository, the skill directory is the top-level directory named after the
skill.

If the repository is already cloned, update it with:

```bash
git -C ~/dev/github.com/artivilla/agent-skills pull --ff-only
```

Skills are installed directly from this checkout. Other agent tools can use
the same files by linking their skills directory to `~/.agents/skills` or to
this repository.

## Contributing

Each standalone skill lives in its own directory and must contain a `SKILL.md`
file. The `entire/` directory is a bundled multi-skill plugin and keeps its
skills under `entire/skills/`. Commit changes to the relevant skill directory,
then push them to GitHub.
