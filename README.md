# MC CyberSec Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**Minecraft security and engineering tools for AI coding agents.** MC CyberSec Skill combines a Rust CLI, a stdio MCP server, reusable agent skills, optional online research, and a local web dashboard to help agents inspect Minecraft projects and make better-informed changes.

> **Status:** Early-stage. The current runtime focuses on local inspection and opt-in research. Remote SSH/SFTP operations, live profiling, deployment workflows, game bots, and automated lab execution are not implemented.

## What it does

- **Discover projects** — collect selected Minecraft, mod-loader, plugin, and dependency metadata.
- **Inspect plugins and mods** — inventory JARs, review manifests and class-file versions, and surface selected static indicators without extracting or running plugin code.
- **Scan source and logs** — inspect selected source files and analyze supported server-lag and Java GC log formats.
- **Research compatibility** — look up supported official documentation and search Modrinth when online research is explicitly enabled.
- **Compare artifacts** — compare a local JAR's SHA-512 and size against a specific Modrinth project version.
- **Connect coding agents** — expose tools through stdio MCP and provide reusable skills under `.agents/skills`.
- **Support UI/UX decisions** — guide agents toward player-first design, with clear menus and feedback, accessible interaction, understandable permissions and costs, and attention to fairness and performance.
- **Review results locally** — use the browser dashboard to run supported operations and inspect or export backend evidence.

Audit results are evidence for review, not a verdict that a plugin is malicious or safe.

## Requirements

- Rust 1.89 or newer.
- A native linker and the C build tools required by your platform (SQLite is bundled).
- Python 3.11+ only if you want to generate coding-agent adapter examples.

Windows is the currently tested environment. Linux has not yet been validated. The core CLI does not require Java, Docker, Node.js, or a local LLM.

## Build

Clone the repository and build the workspace:

```powershell
git clone https://github.com/bachvnju12/mc-cyber-sec-skill.git
cd mc-cyber-sec-skill
cargo build --locked --workspace
```

Run the Rust test suite with:

```powershell
cargo test --locked --workspace
```

For an optimized Windows build:

```powershell
cargo build --locked --release --workspace -j 1
```

The current executable is named `sentinel` (`sentinel.exe` on Windows).

## Run a local audit

Choose a state directory **outside** the project being inspected. State is bound to the canonical project path, so use a separate state directory for each project. Example commands for PowerShell:

```powershell
# Discover project metadata
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state discover

# Inventory plugins and mods
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state inventory

# Inspect a plugin JAR without executing it
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state scan-jar plugins/example.jar --java-version 21

# Scan a source file
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state scan-source src/main/java/Example.java

# Analyze a saved server log
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state analyze-log logs/latest.log
```

Discovery reads selected declarations from files such as `fabric.mod.json`, `velocity-plugin.json`, `gradle.lockfile`, and `gradle.properties`. These declarations may be incomplete or stale; they do not prove which versions are running on a live server. JAR inspection and log analysis also cover only the supported formats and checks.

## MCP and agent skills

Start the MCP server with the project and state paths you want to expose to the connected agent:

```powershell
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state mcp
```

The available MCP tools cover project discovery, file reading, plugin inventory, JAR and source inspection, documentation lookup, plugin search, knowledge queries, log analysis, and artifact verification.

Canonical skills live in `.agents/skills`. The shared `AGENTS.md` also gives coding agents a player-first design mindset: understand the player's journey, make goals and feedback clear, account for accessibility and fairness, and consider how security or performance changes affect gameplay. This is practical UI/UX guidance, not a fixed visual component library or theme.

### Generate host adapter examples

Adapter examples are available for Claude Code, Codex, Antigravity CLI, and OpenCode. Generate configuration fragments into a **new output directory**:

```powershell
python -I scripts/generate_adapters.py --binary .\target\debug\sentinel.exe --project D:\servers\staging --state D:\mc-cybersec-state --output D:\adapter-examples
```

See [Host adapters](docs/host-adapters.md) for where each fragment belongs. Review and merge the generated configuration yourself; existing client settings are not changed automatically.

## Optional online research

Network research is off by default. Add `--allow-research` to explicitly enable supported lookups against catalogued official HTTPS sources. Redirects are disabled and plugin artifacts are not downloaded.

```powershell
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state --allow-research docs paper 1.21.4
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state --allow-research search permissions 1.21.4 paper
```

To compare a local JAR against one specific Modrinth version:

```powershell
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state --allow-research verify-artifact plugins/example.jar <Modrinth-project-ID> <Modrinth-version-ID>
```

A matching hash and file size means the local artifact matches the file declared for that version. It does not prove the publisher's identity, establish that the code is safe, or confirm server compatibility.

## Local dashboard

Set a unique token of at least 32 non-whitespace characters, then start the dashboard:

```powershell
$env:SENTINEL_DASHBOARD_TOKEN = "<your-random-token>"
.\target\debug\sentinel.exe --root D:\servers\staging --state-dir D:\mc-cybersec-state serve --port 8741
```

Open [http://127.0.0.1:8741](http://127.0.0.1:8741) and enter the token. The dashboard is local-only, and its project root is selected when the backend starts. Browser-folder import, ZIP upload, and remote connection management are not currently supported.

## Project docs

- [Host adapter setup](docs/host-adapters.md)
- [Implementation plan](docs/implementation-plan.md)
- [Current status](docs/status.md)

## License

MIT. Third-party dependencies retain their own licenses.
