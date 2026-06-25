# Contributing to Co-Architect

Thanks for your interest in improving Co-Architect.

## What to Contribute

The most valuable contributions right now:

- **Bug reports** — if the skill behaves unexpectedly during a session
- **Session examples** — real (or realistic) session files that demonstrate
  different use cases
- **Documentation improvements** — clearer installation steps, better
  explanations, FAQ entries
- **Interaction contract refinements** — improvements to the per-decision loop
  or audit checklist based on real usage

## How to Contribute

1. Fork the repo
2. Create a branch (`git checkout -b improve-audit-checklist`)
3. Make your changes
4. Test by installing the modified skill and running a session
5. Submit a pull request with a clear description of what changed and why

## Guidelines

- **Keep the interaction contract tight.** It's intentionally concise. Changes
  should be battle-tested in real sessions before being proposed.
- **Session files are the test.** The best way to verify a change is to run a
  Co-Architect session with the modified skill and see if the output quality
  improves.
- **Don't add implementation features.** Co-Architect is a design tool. It
  intentionally does not generate code, scaffold projects, or manage
  dependencies. The hard launch gate is a feature, not a limitation.

## Reporting Issues

Open an issue with:
- What you expected to happen
- What actually happened
- The session context (what you were designing, how far in)
- Your Claude Code version and setup (project-level vs. global install)

## Code of Conduct

Be constructive, be kind, assume good intent. This is a small project and
we're all here to make software design better.
