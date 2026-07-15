# Publishing & Updating This Plugin

This repository is packaged for both Codex and Claude Code.

## Maintained Package Files

- Codex manifest: `.codex-plugin/plugin.json`
- Claude manifest: `.claude-plugin/plugin.json`
- Claude marketplace: `.claude-plugin/marketplace.json`
- Shared skills: `skills/`

The `skills/` tree is shared by both hosts. Keep host-specific packaging in the
manifest directories. Do not fork the skill tree unless behavior must diverge.

This repo does not require a repo-local `.agents/` marketplace. A Codex
marketplace file can be added later for team distribution, but the Codex plugin
manifest is enough for local/plugin-browser development.

---

## Quick Update Flow

1. Edit skills/config as needed.
2. If releasing, bump `version` in all relevant manifests:
   - `.codex-plugin/plugin.json`
   - `.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json` (`plugins[].version`)
3. Validate JSON:

   ```bash
   python -m json.tool .codex-plugin/plugin.json
   python -m json.tool .claude-plugin/plugin.json
   python -m json.tool .claude-plugin/marketplace.json
   ```

4. Commit and push.
5. Reinstall or refresh in the target host.

---

## Codex Notes

Codex uses `.codex-plugin/plugin.json` and loads the shared skills directory:

```json
"skills": "./skills/"
```

Superpowers must be installed separately in Codex. For full subagent workflow
behavior, enable Codex multi-agent support in `~/.codex/config.toml`:

```toml
[features]
multi_agent = true
```

If `[features]` already exists, add `multi_agent = true` inside that block.

Start a new Codex task after installing or refreshing the plugin so the skill
list reloads.

---

## Claude Code Notes

Claude Code uses `.claude-plugin/plugin.json` and
`.claude-plugin/marketplace.json`.

Refresh and reinstall:

```text
/plugin marketplace update edward-dev-workflow
/plugin install edward-dev-workflow@edward-dev-workflow
```

Claude Code clones the marketplace repo fresh or updates its cache during
reinstall. Local uncommitted changes are not picked up by a remote marketplace
install; push before testing a remote install.

---

## Claude Marketplace Source Type

Do not write:

```json
"source": "git"
```

That is not a supported Claude Code source type.

Also avoid the bare GitHub shorthand unless SSH is configured on the installing
machine. Use HTTPS instead:

```json
"source": {
  "source": "url",
  "url": "https://github.com/edwardkao6413/edward-dev-workflow.git"
}
```

Sanity checks:

```bash
git ls-remote https://github.com/edwardkao6413/edward-dev-workflow.git
python -m json.tool .claude-plugin/marketplace.json
```

If Claude Code still uses stale marketplace metadata:

```text
/plugin marketplace update edward-dev-workflow
```

If needed, remove and re-add:

```text
/plugin marketplace remove edward-dev-workflow
/plugin marketplace add https://github.com/edwardkao6413/edward-dev-workflow.git
```
