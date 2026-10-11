# uav-template

Start here to design an RC aircraft with an agent. Create a repository from this template (GitHub: "Use this
template"), then:

```
git clone --recursive git@github.com:<you>/<project>.git
cd <project>/uav
curl -fsSL https://pixi.sh/install.sh | bash     # once per machine, no sudo
pixi install && pixi run build                   # the toolchain (~1 GB) and the solver (~3 min)
cd ..
export PATH="$PWD/uav/bin:$PATH"                 # f5
claude                                           # then say what you want to fly
```

The agent interviews you, writes `brief.md`, iterates `plane.toml` against the solver, and leaves a `.fl5`
project in `build/` for the flow5 GUI. How and why: `uav/VISION.md`.

| path | what |
|---|---|
| `brief.md`, `plane.toml` | the project: written by the agents during the conversation |
| `foils/` | airfoil files the design uses, fetched by the tools |
| `build/` | generated, git-ignored: solver inputs, runs, the `.fl5` |
| `uav/` | the tools, pinned; `.claude/` symlinks into `uav/agent/` so the agent material matches them |

Update the tools: `git -C uav fetch && git -C uav checkout <tag> && git submodule update --init --recursive`,
then rebuild.
