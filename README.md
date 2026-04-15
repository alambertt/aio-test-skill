# aio-test-skill

Generate CSV files for importing test cases into AIO Tests (Jira). Supports Classic and BDD formats, multi-line test cases, datasets with parameters, folder hierarchies, and best practices for clear, complete, traceable test cases.

## Install

Install globally with the skills CLI:

```bash
npx skills add https://github.com/alambertt/aio-test-skill -g -y
```

## What This Skill Covers

- Classic format: single-line and multi-line test cases
- BDD format: Gherkin-style (Feature/Scenario/Given/When/Then)
- Datasets: data-driven test cases with parameters
- Folder hierarchies using `->` separators
- Best practices for titles, steps, expected results, priorities, labels
- Common import errors and how to avoid them

## Usage

After installation, ask your AI agent to:

- "Generate a CSV for AIO Tests with these test cases..."
- "Create test cases for login and registration in BDD format..."
- "Build a multi-line CSV for the checkout flow..."

## Repository

This is a single-skill repository. The skill definition lives in `SKILL.md` at the root.

- Skill name: `aio-tests-csv`
- Language: Spanish (can be adapted to English upon request)

## License

MIT
