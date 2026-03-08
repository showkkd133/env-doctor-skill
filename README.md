# env-doctor

Environment variable health checker skill for Claude Code. Scans your codebase for env var references, compares `.env.example` with `.env`, detects hardcoded secrets, validates variable formats, and auto-generates `.env.example` templates.

## Features

- **Multi-language scanning**: Supports JS/TS (`process.env`, `import.meta.env`), Python (`os.environ`, `os.getenv`), and Go (`os.Getenv`, `os.LookupEnv`)
- **Missing variable detection**: Compares `.env.example` against `.env` to find missing or undocumented variables
- **Hardcoded secret detection**: Regex-based scanning for API keys, tokens, passwords, AWS keys, JWTs, connection strings, and more
- **Format validation**: Validates URLs, ports, booleans, paths, emails, and detects placeholder values
- **Auto-generate `.env.example`**: Creates a categorized template when none exists
- **Docker support**: Scans `docker-compose.yml` and `Dockerfile` for env var references

## Installation

### As a Claude Code Skill

Copy the skill to your Claude skills directory:

```bash
mkdir -p ~/.claude/skills/env-doctor
cp skills/env-doctor/SKILL.md ~/.claude/skills/env-doctor/SKILL.md
```

### As a Claude Code Plugin

Install directly from the repository:

```bash
claude plugin add showkkd133/env-doctor-skill
```

## Usage

Trigger the skill by saying any of the following in Claude Code:

- "env doctor"
- "env check"
- "Check environment variables" / "检查环境变量"
- "检查 .env"

## What It Checks

### 1. Variable Reference Scan

Finds all environment variable references across your codebase:

| Language | Patterns |
|----------|----------|
| JS/TS | `process.env.XXX`, `import.meta.env.XXX` |
| Python | `os.environ["XXX"]`, `os.environ.get("XXX")`, `os.getenv("XXX")` |
| Go | `os.Getenv("XXX")`, `os.LookupEnv("XXX")` |

### 2. `.env` Comparison

- Variables in `.env.example` but missing from `.env`
- Variables used in code but not documented in `.env.example`
- Variables defined in `.env` but unused in code

### 3. Hardcoded Secret Detection

Detects patterns including:

- AWS access keys (`AKIA...`)
- Private keys (`-----BEGIN PRIVATE KEY-----`)
- JWTs (`eyJ...`)
- Database connection strings (`postgres://...`)
- GitHub tokens (`ghp_...`)
- Slack tokens (`xoxb-...`)
- OpenAI keys (`sk-...`)
- Generic API key/secret/password/token assignments

### 4. Format Validation

| Variable Pattern | Expected Format |
|-----------------|----------------|
| `*_URL`, `*_URI` | Valid URL (`https://...`) |
| `*_PORT` | Integer 1-65535 |
| `*_HOST` | Domain or IP address |
| `*_EMAIL` | Email format |
| `IS_*`, `*_ENABLED` | Boolean (`true`/`false`/`0`/`1`) |
| `*_TIMEOUT`, `*_TTL` | Positive integer |
| `*_KEY`, `*_SECRET` | Not a placeholder value |

## Example Output

```
========================================
  env-doctor Health Check Report
========================================

Project type: JS/TS
Scan time: 2026-03-08 14:30:00

-- 1. Variable References ---------------
  Found 12 environment variables in code

-- 2. .env Comparison -------------------
  [MISSING]  REDIS_URL       Defined in .env.example but missing from .env
  [UNDOC]    FEATURE_FLAG_X  Used in code but not in .env.example

-- 3. Hardcoded Secrets -----------------
  [CRITICAL] src/config.ts:42  Suspected hardcoded API key

-- 4. Format Validation -----------------
  [FORMAT]      DATABASE_URL: Expected URL format
  [PLACEHOLDER] API_SECRET: Placeholder value detected

-- 5. Summary ---------------------------
  Pass: 9 | Warning: 2 | Critical: 1
========================================
```

## License

MIT License - see [LICENSE](LICENSE) for details.
