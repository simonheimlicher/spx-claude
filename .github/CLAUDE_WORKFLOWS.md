# Claude Code GitHub Workflows

This repository uses reusable workflows from [spx-gh-actions](https://github.com/simonheimlicher/spx-gh-actions) for Claude Code integration:

1. **`claude.yml`** - Interactive Claude assistant triggered by `@claude` mentions
2. **`claude-code-review.yml`** - Automatic code review on pull requests

## Configuration

### Secrets

Add `CLAUDE_CODE_OAUTH_TOKEN` to your repository secrets (Settings → Secrets and variables → Actions → Secrets).

### Customization

To customize behavior, add `with:` parameters to the workflow files:

```yaml
jobs:
  claude:
    uses: simonheimlicher/spx-gh-actions/.github/workflows/claude.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    with:
      authorized_roles: '["OWNER", "MEMBER"]'
      mention_trigger: "@bot"
      allowed_tools: '--allowed-tools "Read,Grep,Glob,Bash(gh pr:*)"'
```

See [spx-gh-actions README](https://github.com/simonheimlicher/spx-gh-actions#configuration) for all available options.

## Security

Both workflows include authorization checks. Only users with matching `author_association` can trigger Claude workflows.

**Default:** `["OWNER", "MEMBER", "COLLABORATOR"]`
