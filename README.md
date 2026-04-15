# aio-test-skill

Generate, adapt, and validate CSV/Excel files for importing test cases into AIO Tests (Jira). This skill is aligned with the official AIO sample files and import documentation so users can generate files that stay close to the real formats AIO publishes.

## Install

Install globally with the skills CLI:

```bash
npx skills add https://github.com/alambertt/aio-test-skill -g -y
```

## Alignment with Official AIO Documentation

This skill now treats the official AIO Tests sample files and Excel/CSV import documentation as the source of truth.

It is designed to stay aligned with:

- the official classic single-line sample format
- the official classic multi-line sample format
- the official blank import template format
- AIO's Field Mapping and Data Mapping workflow
- official conventions such as folder hierarchy with `->`, dataset values with `|`, and multi-line continuation rows with blank metadata cells

This means the skill prefers official source headers such as:

- `Test Id`, `Summary`, `TestSteps`, `ExpectedResults`, `Story`
- `Test Key`, `Title`, `Preconditions`, `Test Steps`, `Data for Steps`, `Jira Story ID`

## What This Skill Covers

- Classic format: official single-line and multi-line test case patterns
- Blank template format aligned to AIO's sample import header
- BDD guidance without forcing a made-up CSV schema when the user really needs AIO's official flow
- Datasets: data-driven test cases with parameters
- Folder hierarchies using `->` separators
- Field Mapping and Data Mapping guidance
- Common import errors and how to avoid them based on AIO documentation

## Usage

After installation, ask your AI agent to:

- "Generate a CSV for AIO Tests with these test cases..."
- "Create test cases for login and registration in BDD format..."
- "Build a multi-line CSV for the checkout flow..."
- "Convert this spreadsheet to match the official AIO import sample..."
- "Review this CSV and tell me if it aligns with AIO's official format..."

## Official Documentation

Users can verify the alignment directly against the official AIO documentation:

- Excel/CSV import guide: https://aiosupport.atlassian.net/wiki/spaces/AioTests/pages/1999044652/Excel+CSV
- Knowledge base home: https://aiosupport.atlassian.net/wiki/spaces/AioTests

## Repository

This is a single-skill repository. The skill definition lives in `SKILL.md` at the root.

- Skill name: `aio-tests-csv`
- Language: Spanish (can be adapted to English upon request)

## License

MIT
