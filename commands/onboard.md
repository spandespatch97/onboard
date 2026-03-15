---
description: Generate a quick codebase summary for onboarding - project purpose, tech stack, architecture, entry points, build/test steps
codex-description: 'Use when user asks to "onboard to project", "what does this project do", "summarize codebase", "get oriented", "new to this repo", "quick overview". Generates a structured project summary for developers new to a codebase.'
argument-hint: "[path] [--depth=quick|normal|deep] [--output=display|file]"
allowed-tools: Bash(git:*), Read, Glob, Grep, Task, Write
---

# /onboard - Quick Codebase Summary

Generate a structured overview of any codebase to help developers get oriented quickly.

## Arguments

Parse from `$ARGUMENTS`:

- **Path**: Directory to analyze (default: current directory)
- `--depth`: Analysis depth
  - `quick`: Package.json/Cargo.toml + README + directory structure only
  - `normal` (default): + key source files, entry points, patterns
  - `deep`: + dependency analysis, architecture diagram, convention detection
- `--output`: Where to put the summary
  - `display` (default): Output to conversation
  - `file`: Write to `.onboard.md` in the repo root

## Execution

### Step 1: Pre-fetch Repo-Intel (Optional)

```javascript
const fs = require('fs');
const path = require('path');
const cwd = process.cwd();
const stateDir = ['.claude', '.opencode', '.codex']
  .find(d => fs.existsSync(path.join(cwd, d))) || '.claude';
const mapFile = path.join(cwd, stateDir, 'repo-intel.json');

let repoIntelContext = '';
if (fs.existsSync(mapFile)) {
  try {
    const { binary } = require('@agentsys/lib');
    const onboardData = JSON.parse(binary.runAnalyzer(['repo-intel', 'query', 'onboard', '--map-file', mapFile, cwd]));
    const canHelp = JSON.parse(binary.runAnalyzer(['repo-intel', 'query', 'can-i-help', '--map-file', mapFile, cwd]));

    repoIntelContext = '\n\nRepo-intel onboard data (structured summary from binary - use this as the primary source):\n' + JSON.stringify(onboardData, null, 2);
    repoIntelContext += '\n\nContributor guidance:\n' + JSON.stringify(canHelp, null, 2);
  } catch (e) { /* unavailable */ }
}
```

### Step 2: Spawn Onboard Agent

```javascript
await Task({
  subagent_type: "onboard:onboard-agent",
  prompt: `Generate a codebase onboarding summary.
Path: ${targetPath}
Depth: ${depth}
${repoIntelContext}

Analyze the project and produce a structured summary.`
});
```

### Step 3: Output

If `--output=file`, write the summary to `.onboard.md` in the repo root.
Otherwise, display directly in the conversation.

## Output Format

```markdown
# Project: {name}

## What This Project Does
{1-2 sentence purpose statement}

## Tech Stack
- **Language**: {language} {version}
- **Framework**: {framework}
- **Build**: {build tool}
- **Tests**: {test framework}

## Project Structure
{directory tree with annotations}

## Key Entry Points
- **Main**: {entry file} - {what it does}
- **Config**: {config files}
- **Tests**: {test location and how to run}

## Getting Started
```bash
{install command}
{build command}
{test command}
{run command}
```

## Architecture Overview
{2-3 paragraphs on how the code is organized}

## Active Development Areas
{hotspots and recent activity from repo-intel if available}

## Conventions
- {commit style}
- {code patterns observed}
```
