# gastown-devcontainer

A ready-to-use, non-root Linux DevContainer for Gas Town's **full stack**:
`gt`, Beads (`bd`), Dolt, Go, Git, SQLite, ICU development headers, tmux,
Node.js, and Claude Code (the default agent). It supports amd64 and arm64.
See the upstream [installation guide](https://github.com/gastownhall/gastown/blob/main/docs/INSTALLING.md).

## Open the container

Use Docker and VS Code with the **Dev Containers** extension, open this repository,
and select **Dev Containers: Reopen in Container**. Alternatively, create a GitHub
Codespace from this repository. The first build downloads and compiles the tools.
Versions are pinned in `.devcontainer/Dockerfile`; update its build arguments
and rebuild to upgrade. Keep Beads aligned with the version in Gas Town's
`go.mod` to avoid incompatible database migrations.

## First-time setup

Run these commands in the container terminal, as `vscode` (not root).
Dev Containers normally forwards your host Git configuration and SSH agent;
Codespaces provides its own Git authentication. Configure your identity if needed:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
dolt config --global --add user.name "$(git config user.name)"
dolt config --global --add user.email "$(git config user.email)"

# Authenticate with your own Claude account; follow the interactive login.
claude
```

Claude Code requires an eligible account/subscription or API access. Do not put
credentials in the Dockerfile or commit them to this repository. If using an API
key, supply it as a runtime secret/environment variable, not a build argument.

Create the town **outside this repository**, in the persistent home volume:

```bash
gt install ~/gt --shell --git
cd ~/gt
gt rig add myproject https://github.com/you/your-project.git
gt enable
gt up
gt doctor --fix
gt status
gt mayor attach
```

Replace the example rig name and URL with your project. Authenticate Git access
before adding a private repository. `gt up` starts the full-stack services,
including the daemon and tmux-backed agents. Detach from tmux with **Ctrl-b d**.
Use `gt down` to stop services; after a container restart, run `cd ~/gt && gt up`
again. Services and agents are deliberately not started during a build or attach.

## Verify and monitor

```bash
go version
git --version
sqlite3 --version
pkg-config --modversion icu-uc
dolt version
bd version
gt version
tmux -V
claude --version

cd ~/gt
gt convoy list
gt dashboard
```

Open the dashboard using the forwarded **8080** port. Keep the port private in
Codespaces; neither the dashboard nor Dolt should be exposed publicly. Dolt runs
inside the container and does not need a host port or a separate database service.

The per-DevContainer named volume at `/home/vscode` preserves the town, rigs,
Dolt data, agent login, and shell configuration across rebuilds. The repository
remains in the normal Dev Containers workspace mount. Removing the home volume
deletes that state, including credentials; back up valuable work first. Keeping
Dolt data in a Docker volume also avoids macOS bind-mount journaling issues.
The container does not mount the Docker socket or require privileged mode.
