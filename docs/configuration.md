# Configuration Guide

## Session Storage

By default, Co-Architect stores session files in a `sessions/` directory at
your project root. This works out of the box with no configuration.

### Custom Session Homes

If you want session files to live somewhere other than `sessions/`, or if you
work across multiple projects and devices, configure `homes.json` in the skill's
references directory.

Copy the example file and edit it:

```bash
cp skill/references/homes.json.example skill/references/homes.json
```

### homes.json Structure

```json
{
  "default": "sessions/",
  "registry": {
    "project-name": {
      "path": "/absolute/path/to/sessions",
      "cross_device": true
    }
  }
}
```

| Field | Purpose |
|-------|---------|
| `default` | Fallback directory when no project match is found |
| `registry` | Map of project names to session directories |
| `path` | Where session files for this project are stored |
| `cross_device` | If `true`, this path is accessible from multiple machines (e.g., synced via git or Dropbox) |

### Resolution Order

When you start a session and name a destination project:

1. Co-Architect checks `homes.json` for a matching registry entry
2. If found with `cross_device: true` → uses that path
3. If found with `cross_device: false` → uses that path (but won't expect
   cross-device resume to work)
4. If not found → falls back to `default`

### Cross-Device Setup

For session files to be available across devices, they need to live in a
location that syncs. Two common approaches:

**Git repo (recommended):** Keep sessions in a git repo that you push/pull
across devices. Co-Architect commits after each locked decision, so the
latest state is always available.

**Synced folder:** Use Dropbox, iCloud, or similar. Works but doesn't give
you the version history that git provides.

## Session Index

The session index (`sessions/_index.md` or wherever your default points) is
the discovery file that Co-Architect reads to find and list open sessions.
It's created automatically when you start your first session.

If you use multiple project homes across devices, keep one canonical index
in a location that all devices can reach. This is your cross-device
discovery source.

## Skill Placement

The skill can live in two locations:

| Location | Scope | Use when |
|----------|-------|----------|
| `your-project/.claude/skills/co-architect/` | This project only | You only use Co-Architect in one project |
| `~/.claude/skills/co-architect/` | All projects | You want it available everywhere |

For global installation, symlink rather than copy so updates propagate:

```bash
ln -s /path/to/co-architect/skill ~/.claude/skills/co-architect
```
