# zg-skills

coral's skills library, packaged as independently installable plugins for Codex.

## Plugins

| Plugin | Skill | Purpose |
| --- | --- | --- |
| [Draft PR](plugins/draft-pr/plugin.json) | [`draft-pr`](plugins/draft-pr/skills/draft-pr/SKILL.md) | Draft evidence-backed pull request titles and descriptions. |
| [Merge to Branch](plugins/merge-to-branch/plugin.json) | [`merge-to-branch`](plugins/merge-to-branch/skills/merge-to-branch/SKILL.md) | Squash every committed change from the current branch into an explicit target branch. |
| [Optimize Prompt](plugins/optimize-prompt/plugin.json) | [`optimize-prompt`](plugins/optimize-prompt/skills/optimize-prompt/SKILL.md) | Optimize GPT-5.6 prompts and surface instruction or information conflicts. |

Each plugin contains one skill and its Codex UI metadata. No bundled MCP server or connector is required.

## Install in Codex

From a local checkout, register the repository as a marketplace:

```bash
codex plugin marketplace add /absolute/path/to/zg-skills
```

Once these changes are available on GitHub, you can register the published repository instead:

```bash
codex plugin marketplace add CoralLeiCN/zg-skills
```

Restart the desktop app, open the Plugins Directory, select **coral's Skills**, and install the plugins you want. The repository marketplace is defined in [`.agents/plugins/marketplace.json`](.agents/plugins/marketplace.json).

Invoke an installed skill by its qualified name, for example:

```text
Use $draft-pr:draft-pr to draft a PR title and body for these changes.
Use $merge-to-branch:merge-to-branch to squash my committed changes into main.
Use $optimize-prompt:optimize-prompt to improve this prompt for GPT-5.6: …
```

For Merge to Branch, replace `main` with your intended local target branch; the skill requires that target to be checked out in a separate clean worktree.

## Repository layout

```text
.agents/plugins/marketplace.json
plugins/
  draft-pr/
    plugin.json
    LICENSE
    skills/draft-pr/
      SKILL.md
      agents/openai.yaml
  merge-to-branch/
    plugin.json
    LICENSE
    skills/merge-to-branch/
      SKILL.md
      agents/openai.yaml
  optimize-prompt/
    plugin.json
    LICENSE
    skills/optimize-prompt/
      SKILL.md
      agents/openai.yaml
```

The original top-level skill directories now live under their plugin's `skills/` directory. Update any direct skill-install paths to the locations linked above.

## Maintain the plugins

Edit each skill in its plugin directory and bump that plugin's `version` in `plugin.json` when releasing changes. Keep the manifest's display information and default prompt aligned with `agents/openai.yaml`. Each plugin includes the repository's Apache 2.0 license so its folder can be distributed independently.

The packages use the portable Agent Plugins manifest with OpenAI presentation metadata under `extensions.com.openai`. See the [official plugin packaging documentation](https://developers.openai.com/plugins/build/plugins) for marketplace setup and distribution details.
