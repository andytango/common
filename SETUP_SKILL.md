---
name: setup-guidelines
description: Setup agent development guidelines from andytango/common repo based on project language
argument-hint: "[project-path]"
allowed-tools: Read, Write, Edit, Bash, Glob, WebFetch, AskUserQuestion
---

# Setup Agent Development Guidelines

You are tasked with setting up agent development guidelines for a project (or multiple nested projects).

## Project Path

Project path to setup: $ARGUMENTS (defaults to current directory if not provided)

## Step 1: Discover Projects

1. If $ARGUMENTS is provided, use that as the base path. Otherwise, use the current working directory.
2. Check if the base path contains a package.json, Cargo.toml, pyproject.toml, or other language-specific project files
3. Also scan for nested projects (subdirectories that contain their own project files)
4. For each project found, note its primary language(s):
   - TypeScript/JavaScript: package.json with typescript dependency or .ts files
   - Rust: Cargo.toml
   - Python: pyproject.toml, setup.py, or requirements.txt

## Step 2: Fetch Guidelines

For each language detected, fetch the latest guidelines from https://github.com/andytango/common:

**Base guidelines (always include):**
- `https://raw.githubusercontent.com/andytango/common/main/AGENT_GUIDELINES_BASE.md`

**Language-specific guidelines:**
- TypeScript: `https://raw.githubusercontent.com/andytango/common/main/AGENT_GUIDELINES_TS.md`
- Python: `https://raw.githubusercontent.com/andytango/common/main/AGENT_GUIDELINES_PYTHON.md`
- Rust: `https://raw.githubusercontent.com/andytango/common/main/AGENT_GUIDELINES_RUST.md`

**Setup prompts (optional, based on language):**
- TypeScript: `https://raw.githubusercontent.com/andytango/common/main/SETUP_PROMPT_TS.md`
- Python: `https://raw.githubusercontent.com/andytango/common/main/SETUP_PROMPT_PYTHON.md`
- Rust: `https://raw.githubusercontent.com/andytango/common/main/SETUP_PROMPT_RUST.md`

Use the WebFetch tool to retrieve the raw content from these URLs.

## Step 3: Create/Update AGENTS.md

For each project directory:

1. **Check if AGENTS.md exists**:
   - If it exists, read it first to understand any custom guidelines
   - Ask the user if they want to preserve custom sections or completely replace the file

2. **Create/Update AGENTS.md** with the following structure:
   ```markdown
   # Agent Development Guidelines

   This project follows the agent development guidelines from [andytango/common](https://github.com/andytango/common).

   **Last updated**: [current date]
   **Languages**: [detected languages]

   ---

   [Insert AGENT_GUIDELINES_BASE.md content]

   ---

   [Insert language-specific guidelines for each detected language]

   ---

   ## Project-Specific Guidelines

   [Any custom guidelines - preserve if user requests, or leave blank for new files]
   ```

3. **Create/Update CLAUDE.md symlink**:
   - Check if CLAUDE.md exists
   - If it's a regular file (not a symlink), ask the user if they want to back it up before converting to a symlink
   - Create a symlink: `ln -sf AGENTS.md CLAUDE.md`
   - Verify the symlink is correct: `ls -la CLAUDE.md`

## Step 4: Summary

After processing all projects, provide a summary:

```markdown
## Setup Summary

### Projects Updated:
- [Project path 1]: [Languages] - AGENTS.md [created/updated], CLAUDE.md symlink [created/verified]
- [Project path 2]: ...

### Guidelines Applied:
- Base guidelines
- TypeScript guidelines (if applicable)
- Python guidelines (if applicable)
- Rust guidelines (if applicable)

### Next Steps:
1. Review the generated AGENTS.md files
2. Add any project-specific guidelines to the "Project-Specific Guidelines" section
3. Commit the changes when ready
```

## Step 5: Wait for User Confirmation

**IMPORTANT**: Do NOT commit any changes automatically. Instead:

1. Show the user the summary of changes
2. Ask: "Would you like to commit these changes? I can create a commit with the message 'docs: setup agent development guidelines from andytango/common'"
3. Wait for explicit user confirmation
4. If confirmed, create the commit using conventional commits format

## Installation

To use this skill, install it to your personal Claude skills directory:

```bash
# Download and install the skill
curl -o ~/.claude/skills/setup-guidelines/SKILL.md \
  https://raw.githubusercontent.com/andytango/common/main/SETUP_SKILL.md

# Or if you've cloned the repo:
mkdir -p ~/.claude/skills/setup-guidelines
cp /path/to/common/SETUP_SKILL.md ~/.claude/skills/setup-guidelines/SKILL.md
```

## Example Usage

```bash
# Setup guidelines for current project
/setup-guidelines

# Setup guidelines for a specific project
/setup-guidelines ~/projects/my-app

# Setup guidelines for a monorepo (will detect nested projects)
/setup-guidelines ~/projects/monorepo
```

## Error Handling

- If unable to fetch guidelines from GitHub, inform the user and ask if they have a local copy
- If a project's language cannot be determined, ask the user which guidelines to use
- If symlink creation fails (e.g., on Windows), create a copy of AGENTS.md as CLAUDE.md instead
