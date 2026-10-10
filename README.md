# devenv

Local Docker environments for coding agents. Each environment is its own container and can see only one workspace directory. The container has no extra privileges, no Docker socket, and no host SSH agent.

The image includes:

- zsh, Oh My Zsh, and Powerlevel10k
- plugins for git, docker, fzf, suggestions, syntax highlighting, and a few more
- [RESH](https://github.com/curusarn/resh) shell history
- [nvm](https://github.com/nvm-sh/nvm) with Node.js 24.18.0
- [SDKMAN](https://sdkman.io) with Java 21 (Temurin)
- GCC, Clang 18, GDB, CMake, Ninja, Autotools, and headers for building native code
- Python 3, pip, and venv, including pyelftools, requests, and capstone
- [Blutter](https://github.com/worawit/blutter) for Flutter AOT snapshots (`blutter`)
- [Cursor agent](https://cursor.com/cli)
- [OpenCode](https://opencode.ai)
- [Pi](https://pi.dev), with goal, cursor, questions, MCP, subagents, web access, plan, and lens packages
- [Claude Code](https://code.claude.com)
- [Ghidra](https://ghidra-sre.org) 12.1.4
- [REA](https://github.com/morluto/rea)

## Requirements

- Linux with Docker Engine and Docker Compose v2
- Your user in the `docker` group

## Install

```bash
git clone git@github.com:jehadbaeth/devenv.git
cd devenv
./devenv build
```

The build creates the container user with your uid and gid, so files written in a workspace stay owned by you.

The container can use the public internet. `./devenv new` and `./devenv shell` drop connections to this machine, private networks, and link-local addresses. DNS is allowed only to the container's nameservers. Memory is capped at 16 GB, CPUs at 8, and the process count at 2048. The command refuses to mount `/`, your home directory, this repo, or credential directories such as `.ssh`.

The agent is not root and `sudo` is not installed. `pip install`, `npm install -g`, and `sdk install` write into the home directory, so those do not need root. Packages that have to be installed with `apt` belong in the image.

## Use

```bash
./devenv new                 # name is a timestamp, workspace is environments/<name>
./devenv new api ~/src/api   # create it, or replace that container and keep ~/src/api
./devenv new api             # workspace is environments/api
./devenv new web ~/src/web   # use a directory you already have
./devenv shell api           # zsh in that environment
./devenv ls
./devenv stop api
./devenv rm api              # removes the container and keeps the workspace
```

`./devenv ls` prints the name to pass to the other commands.

Open as many terminals as you want on one environment. Each `./devenv shell api` starts another zsh in the same container. They share that workspace. `stop` and `rm` apply to the whole environment.

## Cursor agent

Copy the example env file and set a key:

```bash
cp .env.example .env
```

```bash
CURSOR_API_KEY=your_key_here
```

Compose reads `.env` when an environment starts and passes the key into the container. The key is not written into the image. `.env` is gitignored.

You can also sign in inside the environment with `cursor-agent login`. That login lives in the container and is gone after `./devenv rm`.

OpenCode, Pi, and REA have their own sign-in:

```bash
opencode providers
pi auth
rea setup
```

## Update

After pulling a newer revision, rebuild and recreate environments. A container keeps the image it was created with until you remove it.

```bash
git pull
./devenv build
./devenv rm api
./devenv new api ~/src/api
```
