# Changelog

## v2.0.0 (2026-03-24)

### Breaking Changes
- Streamlined from 488 lines to 181 lines (63% reduction)
- Removed redundant PowerShell code examples (kept concise workflow descriptions)
- Absorbed capabilities from deleted skills: agent-browser, playwright-mcp

### New Features
- **Screenshot as core exclusive capability**: Explicitly declared as the only screenshot entry point
- **Fallback chain**: agent-browser CLI and Playwright MCP documented as Gateway fallback options
- **Traditional search engines**: Added Bing/Google workflow
- **JavaScript examples**: Scroll, get links, get title — 3 common use cases
- **Limitations section**: 5 structured constraints documented
- **Rate limiting troubleshooting**: Added to fault diagnosis table
- **Full manual startup script**: Including SYSTEM user symlink creation for Windows services

### Changes
- `openclaw.json` configuration example added to skill
- Trigger words optimized in YAML frontmatter
- Coordination note with opencli-natural-commands added to description

## v1.3.0 (2026-03-23)

- Added OpenCLI smart startup mode (`opencli doctor --live`)

## v1.2.0 (2026-03-18)

- Improved trigger words, troubleshooting, use cases

## v1.1.0 (2026-03-18)

- AI search priority: Kimi/Gemini Web support

## v1.0.0 (2026-03-18)

- Initial release: basic browser control via OpenClaw Chrome Attach mode
