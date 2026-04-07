Perform a full dependency audit of this codebase. Check all of the following and print a summary to the terminal:

## 1. Security Vulnerabilities
Run `npm audit` and report any known vulnerabilities grouped by severity (critical, high, moderate, low).

## 2. Outdated Packages
Run `npm outdated` and list all production and dev dependencies that have newer versions available. Show current version, wanted version, and latest version.

## 3. Unused Dependencies
Scan the entire `src/` directory and cross-reference against all dependencies and devDependencies in `package.json`. Report any packages that are installed but never imported or referenced anywhere in the codebase. Check for imports using all common patterns: `import`, `require`, and references in config files (next.config.ts, postcss.config.mjs, vitest.config.mts, tailwind config, .eslintrc.json, components.json).

## 4. Missing Dependencies
Scan all imports across `src/` and config files. Report any packages that are imported in code but not listed in `package.json` (excluding Node.js built-ins and path aliases like `@/`).

## 5. Duplicate/Redundant Packages
Check for packages that serve the same purpose or overlap significantly (e.g., multiple CSS-in-JS solutions, multiple state managers, multiple test runners).

## Output Format
Print a single terminal summary grouped by section. For each section show a count of findings, then list each finding on its own line. Include both production and dev dependencies in all checks. This is purely informational — no fixes should be applied, but do include actionable suggestions (e.g., "run `npm update X`" or "remove X from package.json") next to each finding.

At the end, print a totals line: `Found: X vulnerabilities, X outdated, X unused, X missing, X duplicates`.
