# Contributing to Awesome Backtesting Python

The goal of this list is to be the most honest, up-to-date reference for Python backtesting in 2026 and beyond. Accuracy matters more than comprehensiveness.

## What We Accept

**Most valuable**:
- Maintenance status updates for listed frameworks (with evidence: last commit date, open issues count, Python 3.11+ compatibility)
- New frameworks that are actively maintained and offer something genuinely different from existing entries
- Additional pitfall documentation with real-world examples
- Code examples that correct or improve existing snippets
- Benchmark comparisons (performance, accuracy) with reproducible methodology

**Also welcome**:
- New sections for uncovered areas (e.g., crypto-specific backtesting, options backtesting)
- Corrections to the Decision Guide based on real-world experience

## What We Do Not Accept

- Frameworks that haven't had a meaningful commit in 24+ months (open an issue to discuss "legacy" status)
- Code examples that require proprietary data or paid API keys to run (unless clearly labeled)
- Promotional content from framework authors without disclosure
- "Also useful" additions that don't meaningfully differ from existing entries
- Framework comparisons that don't include the framework's actual weaknesses

## The Honesty Standard

This list exists because most backtesting framework comparisons are either outdated or written by people with an interest in promoting one option. Contributions must:

1. **Acknowledge weaknesses** — every framework has them; a contribution that only praises is incomplete.
2. **Include maintenance status** — a framework that was great in 2020 but is now unmaintained should be labeled as such.
3. **Be reproducible** — code examples must run with the stated dependencies; include version pins if the example is sensitive to library version.

## Verifying Maintenance Status

Use this checklist when submitting a framework status update:
- [ ] Checked GitHub repository's last commit date
- [ ] Checked open issues count and whether maintainers are responding
- [ ] Tested with Python 3.11 or 3.12
- [ ] Checked for known incompatibilities with current pandas/numpy versions
- [ ] Verified the listed installation command still works

## Format for New Framework Entries

```markdown
### Framework Name

**GitHub**: [org/repo](https://github.com/org/repo) | **Stars**: ~Xk | **Status note**

One paragraph: What is it, who is it for, what makes it different?

**Strengths**:
- Specific strength 1
- Specific strength 2

**Weaknesses**:
- Specific weakness 1 (this section is non-negotiable)

**Installation**:
```bash
pip install framework-name
```

**Example — [Descriptive name of what the example shows]**:
```python
# Minimal working code example
# Must run without errors with pip install framework-name
```
```

## Pull Request Process

1. Fork and create a descriptive branch
2. Run all code examples yourself before submitting
3. Update the comparison table at the top if you change a framework's status
4. Reference your evidence (GitHub commit links, issue links, test output) in the PR description

## Disclosures

If you are a maintainer or contributor to a framework you're adding or updating, disclose this in the PR. We appreciate framework maintainers keeping their entries accurate — just be transparent about the relationship.
