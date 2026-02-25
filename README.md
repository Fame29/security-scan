# ZeriFlow Security Scan

AI-powered security scanning for your pull requests. Catches secrets, vulnerabilities, auth issues, and more.

## Quick Setup (3 minutes)

### 1. Get your API key
Go to [zeriflow.com/dashboard/ci](https://zeriflow.com/dashboard/ci) and connect your repository.

### 2. Add your API key to GitHub Secrets
Go to **Settings > Secrets and variables > Actions > New repository secret**

- Name: `ZERIFLOW_API_KEY`
- Value: your API key from step 1

### 3. Create the workflow file

Create `.github/workflows/zeriflow.yml`:

```yaml
name: ZeriFlow Security
on:
  pull_request:
    branches: [main, master]

permissions:
  contents: read
  pull-requests: write

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: Fame29/security-scan@v1
        with:
          api-key: ${{ secrets.ZERIFLOW_API_KEY }}
```

Every PR now gets a security check.

## What it checks

| Category | Examples |
|----------|---------|
| Secrets | Hardcoded API keys, tokens, passwords, .env files |
| Dependencies | CVEs, abandoned packages, known malware |
| Injection | SQL injection, XSS, eval(), command injection |
| Auth | Missing auth middleware, IDOR, session issues |
| Config | CORS *, debug mode, exposed stack traces |
| Rate Limiting | Missing rate limits on auth/API endpoints |
| Data Exposure | Sensitive data in responses, PII leaks |
| Error Handling | Stack traces in production, unhandled errors |
| Performance | N+1 queries, bundle size, unnecessary re-renders |
| Accessibility | Missing alt text, labels, ARIA attributes |
| Best Practices | Code quality, security patterns |

## Configuration

```yaml
- uses: Fame29/security-scan@v1
  with:
    api-key: ${{ secrets.ZERIFLOW_API_KEY }}
    threshold: 70           # Minimum score to pass (default: project setting)
    fail-on-critical: true  # Block merge on critical issues (default: true)
    comment: true           # Post results as PR comment (default: true)
```

## How it works

1. **Static analysis** — Semgrep + Gitleaks + npm audit scan your changed files
2. **AI review** — Claude Sonnet 4 analyzes findings in context, removes false positives, catches missed issues
3. **Score & report** — You get a /100 score with actionable fix suggestions posted as a PR comment
4. **Gate** — If score is below threshold or critical issues found, the PR check fails

## Pricing

CI/CD scans use your ZeriFlow plan or tokens:

| Plan | Included CI scans/month |
|------|------------------------|
| Free | 0 (tokens only) |
| Pro ($4.99/mo) | 5 |
| Business ($19.99/mo) | 20 |

Extra scans: buy token packs starting at $4.99/10 tokens.

[View pricing](https://zeriflow.com/pricing)

## Example PR comment

> ## ZeriFlow Security Scan — PASSED
>
> **Score: 87/100** | Threshold: 60 | 0 critical · 2 warnings · 1 info
>
> Good security posture. 2 minor issues found with suggested fixes.
>
> ### Findings
> - WARNING: Missing rate limiting on /api/auth/login (`src/routes/auth.ts:45`)
> - WARNING: CORS allows all origins in development config (`src/config.ts:12`)
> - INFO: Consider adding Content-Security-Policy header (`next.config.js:8`)

## Requirements

- `fetch-depth: 0` in checkout step (required for git diff)
- `pull-requests: write` permission (for PR comments)
- Node.js available in runner (default on ubuntu-latest)

## Support

- [Documentation](https://zeriflow.com/docs/ci-cd)
- [Dashboard](https://zeriflow.com/dashboard/ci)
- [Report an issue](https://github.com/Fame29/security-scan/issues)
